---  
layout: post  
title:  "inav代码解读2 - inav的启动和任务调度"  
date:  2019-11-06 15:37:56  
categories: 无人机  
tags: UAV inav  
---  

* content  
{:toc} 

这是inav代码解读的第二部分。inav固件与px4固件的一大区别是，inav固件的思想更偏近于小型的RTOS的开发，在程序中有清楚的逻辑，具有一个小型的实时调度器。而PX4固件有一个更加复杂的Nuttx实时系统，同时引入uORB进行任务间通讯，使用nsh实时解析类shell脚本用于飞控的启动流程控制，使用ROMFS文件系统，PX4与inav的设计思想完全不一样。  
相比之下，由于系统资源的限制，inav(包括CF/BF固件)是一个传统小型工程，易于把控整个飞控从上到下的逻辑，更易于理解，不用去了解操作系统层面的知识。虽然inav代码层次感和模块划分并没有这么强，但是这却不阻碍其成为一个成功的飞控固件(当然PX4飞控如果只是单独开发某个模块也很方便，但是如果说对整个系统的了解和把握却比inav要难)

## 从飞控启动入口说起
飞控加电，stm32主控启动，stm32的启动默认是从0x8000000地址启动，初始运行的汇编位于```./src/main/startup/startup_stm32f30x_md_gcc.S```，在这里将对stm32进行堆栈初始化，确定中断向量地址等等一系列操作，同一般的stm32开发相同，之后将转入```main()```函数  

```c
int main(void)
{
    init();
    loopbackInit();

    while (true) {
        scheduler();
        processLoopback();
    }
}
```

很常规的一个main函数，首先```init()```调用初始化函数，然后进入循环，开始任务调度  

## 飞控初始化init()
飞控初始化函数位于```./src/main/fc/fc_init.c```，在该函数内进行  
遵循的顺序是```时钟-IO-EEPROM-串口-舵机-蜂鸣器-SPI-IIC-ADC-sensor-pid-imu-gps-导航-其他辅助设备```

## 任务调度器
任务调度器位于```./src/main/scheduler/scheduler.c```中，这里简单介绍一下调度器，该调度器是非抢占式的多优先级的调度器，任务被列在```./src/main/fc/fctasks.c```的```cfTasks[TASK_SYSTEM]```结构体中，调度器依据各个任务的排序和优先级进行调度，任务具有两种方式：一种是事件驱动，一种是周期时间驱动。  
因此，我们重点关注一下```cfTasks[TASK_SYSTEM]```,下面截取一小段  

```c
//任务定义
cfTask_t cfTasks[TASK_COUNT] = {
    [TASK_SYSTEM] = { 
        .taskName = "SYSTEM",  //系统任务
        .taskFunc = taskSystem,  //任务函数
        .desiredPeriod = TASK_PERIOD_HZ(10),              // 每100 ms运行一次, 10Hz
        .staticPriority = TASK_PRIORITY_HIGH,
    },


    //对于Flash大于128k的飞控，将采用异步执行的方式分别处理PID，陀螺仪，加速度计，高度计的数据
    //占用代码空间多，但是性能更加优异
    //该宏定义位于./src/main/target/common.h
    #ifdef USE_ASYNC_GYRO_PROCESSING
        [TASK_PID] = {
            .taskName = "PID",
            .taskFunc = taskMainPidLoop,    //该任务函数是飞行控制的核心处理函数！！！
            .desiredPeriod = TASK_PERIOD_HZ(500), // Run at 500Hz
            .staticPriority = TASK_PRIORITY_HIGH,
        },

        [TASK_GYRO] = {
            .taskName = "GYRO",
            .taskFunc = taskGyro,   //更新陀螺仪任务函数
            .desiredPeriod = TASK_PERIOD_HZ(1000), //Run at 1000Hz
            .staticPriority = TASK_PRIORITY_REALTIME,
        },

        [TASK_ACC] = {
            .taskName = "ACC",
            .taskFunc = taskAcc,   //更新加速度计数据的任务函数
            .desiredPeriod = TASK_PERIOD_HZ(520), //520Hz is ACC bandwidth (260Hz) * 2
            .staticPriority = TASK_PRIORITY_HIGH,
        },

        [TASK_ATTI] = {
            .taskName = "ATTITUDE",
            .taskFunc = taskAttitude,  //更新高度计数据的任务函数
            .desiredPeriod = TASK_PERIOD_HZ(60), 
            .staticPriority = TASK_PRIORITY_HIGH,
        },

    #else
        /*
         * 传统的同步处理方式，节省Flash空间
         */

        [TASK_GYROPID] = {
            .taskName = "GYRO/PID",
            .taskFunc = taskMainPidLoop,
            .desiredPeriod = TASK_PERIOD_US(1000),
            .staticPriority = TASK_PRIORITY_REALTIME,
        },
    #endif

    [TASK_SERIAL] = {    //串口接收 处理msp消息
        .taskName = "SERIAL",
        .taskFunc = taskHandleSerial,
        .desiredPeriod = TASK_PERIOD_HZ(100),    
        .staticPriority = TASK_PRIORITY_LOW,
    },

#ifdef BEEPER
    [TASK_BEEPER] = {
        .taskName = "BEEPER",
        .taskFunc = beeperUpdate,   //更新蜂鸣器任务处理
        .desiredPeriod = TASK_PERIOD_HZ(100),    
        .staticPriority = TASK_PRIORITY_MEDIUM,
    },
#endif

    [TASK_RX] = {     //接收器
        .taskName = "RX",
        .checkFunc = taskUpdateRxCheck,    //基于事件驱动 检查是否帧数据完整
        .taskFunc = taskUpdateRxMain,
        .desiredPeriod = TASK_PERIOD_HZ(50),      // 如果基于事件的调度不起作用，则回退到定期调度
        .staticPriority = TASK_PRIORITY_HIGH,
    },

#ifdef USE_GPS
    [TASK_GPS] = {
        .taskName = "GPS",
        .taskFunc = taskProcessGPS,
        .desiredPeriod = TASK_PERIOD_HZ(25),    
        .staticPriority = TASK_PRIORITY_MEDIUM,
    },
#endif
    //省略...
};
```

在该结构体中能看到各个任务的名称，处理函数，优先级，运行周期等各个数据，任务主要分为以下：  
- 主处理任务：```taskMainPidLoop()```，在该任务中将进行PID解算，飞行模式控制，导航处理（导航不单独占一个任务）
- 接收机处理任务：```TASK_RX```，该任务会进行失控保护判断，飞行器解锁和锁定等动作也由该任务执行
- 处理主控输入输出：传感器数据、接收机输入、蜂鸣器等，如，```TASK_GPS```，```TASK_BEEP```等
- 其他辅助任务
- 处理MSP数传任务：```TASK_SERIAL```

核心任务是```taskMainPidLoop()```，其他任务都是为其提供数据输入和电机控制输出，在下一节我们主要从```taskMainPidLoop()```入手，了解inav的实际运行流程
