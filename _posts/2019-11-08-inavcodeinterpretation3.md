---  
layout: post  
title:  "inav代码解读3 - inav的核心控制处理流程"  
date:  2019-11-08 16:31:54  
categories: 无人机  
tags: UAV inav  
---  

* content  
{:toc} 

inav代码的主处理循环函数为```taskMainPidLoop()```，位于```./src/main/fc/fc_core.c```。该函数承担着整个飞机的姿态控制，飞行状态，导航，混控和输出等功能，是inav的核心代码（看文件名：fc_core.c也可以知道），下面我们分析一下这个最核心的任务。

## 任务流程
该函数的主要流程如下

序号 | 功能(按顺序) | 函数/变量 | 备注
-|-|-|-
1 | 计算飞行时间 | 修改flightTime | 在解锁后记录飞行时间
2 | 更新imu | imuUpdate*() | 在异步/同步模式下处理陀螺仪和imu数据
3 | 更新通道数据和解锁状态 | annexCode() | 对接收的通道数据rcCommand[]进行缩放，并判断是否可以解锁
4 | 接收机数据滤波 | filterRc() | 可选
5 | 更新导航状态 | updateWaypointsAndNavigationMode() |  如果执行了```TASK_RX```就会更新导航状态，执行导航状态机
6 | 更新位置信息 | updatePositionEstimator() | 根据IMU及GPS的数据估计飞行器当前位置
7 | 导航控制飞行器 | applyWaypointNavigationAndAltitudeHold() | 前面导航所需的指令和位置都已获得，在这一步进行实际的通道控制
8 | 用户控制优化 | rcCommand[] | 最小油门时取消偏航，进行油门斜率补偿
9 | PID计算 | pidController() | 根据rcCommand[]计算各类姿态控制量
10 | 混控 | mixTable() | 根据姿态控制量控制电机及舵机
11 | 输出 | write*() | 控制输出电机和舵机的PWM

其中，当固件启用导航时，第5 6 7才会启用，默认开启，与是否安装GPS无关系。  
对于整个流程，从最简单的方式来看，其实就是：  
**读取传感器(2) -> 接收机输入(3) ->  更新状态(3 4 5 6 7)  ->  PID控制(9) -> 输出(10 11)**  
首先把遥控器摇杆数据存入rcCommand[]数组，然后根据当前飞行器的状态对rcCommand进行操作(比如在自动导航时导航模块会对rcCommand[]进行修改和微调)，接下来PID控制器根据IMU和rcCommand[]输出各个姿态控制量，混控器再对姿态控制量对电机进行映射。  
IMU和PID部分的实现基本上采用的是现在成熟的算法，因此把本系列把重心放在inav的逻辑部分，先不去讨论算法和数学。接下来，主要是分析状态更新(对应上表的3 4 5 6 7)

### 3更新通道数据和解锁状态 
本阶段都在```annexCode()```完成，主要分为更新通道数据和判断解锁状态  
#### 更新通道数据
该流程主要对横滚、俯仰、偏航通道进行缩放，计算各个通道的曲线  
然后计算推力，对推力范围进行限制，同时对推力通道进行单独滤波，放置推力发生突变  
#### 判断解锁状态updateArmingStatus()
飞机解锁时需要进行判断，比如传感器数据发生错误、飞机没有处于水平、油门杆没有在最低位等都会阻止飞机的解锁。所有解锁的判断都是由该函数完成，判断的结构都会反馈到```armingFlags```上。   
下面列出了该结构体和对其操作的宏定义：  

```c
typedef enum {
    ARMED                                           = (1 << 2),
    WAS_EVER_ARMED                                  = (1 << 3),

    ARMING_DISABLED_FAILSAFE_SYSTEM                 = (1 << 7),

    ARMING_DISABLED_NOT_LEVEL                       = (1 << 8),
    ARMING_DISABLED_SENSORS_CALIBRATING             = (1 << 9),
    ARMING_DISABLED_SYSTEM_OVERLOADED               = (1 << 10),
    ARMING_DISABLED_NAVIGATION_UNSAFE               = (1 << 11),
    ... //省略
} armingFlag_e;

extern uint32_t armingFlags;

extern const char *armingDisableFlagNames[];

#define isArmingDisabled()          (armingFlags & (ARMING_DISABLED_ALL_FLAGS))
#define DISABLE_ARMING_FLAG(mask)   (armingFlags &= ~(mask))
#define ENABLE_ARMING_FLAG(mask)    (armingFlags |= (mask))
#define ARMING_FLAG(mask)           (armingFlags & (mask))
```

在该函数中，将对以上各项进行检测，如果有不达标的项目，将会阻止飞行器解锁。  
其中有一个```navigationBlockArming()```函数(该函数在```./src/main/src/navigation/navigation.c```中)是是对导航状态的判断，如果导航模块阻止，则会禁止解锁  

下一节，将把重心调整到导航模块(navigation)，导航模块虽然作为一个可选的模块，没有与inav其他非导航相关的飞行模式集成在一起，但是这使得导航模块特别紧凑，其内部有单独的配置数据和导航状态向量机，是inav实现自动飞行的重要部分。

