---  
layout: post  
title:  "inav代码解读4 - inav的导航状态"  
date:  2019-11-19 13:13:52  
categories: 无人机  
tags: UAV inav  
---  

* content  
{:toc} 

上一节讲了导航的状态转换，这一节说说导航模块如何进行实际的流程控制。根据第三部分内容，飞控主处理流程中与导航模块相关的主要有三部分，分别是：  
序号 | 功能(按顺序) | 函数/变量 | 备注
-|-|-|-
5 | 更新导航状态 | updateWaypointsAndNavigationMode() |  如果执行了```TASK_RX```就会更新导航状态，执行导航状态机
6 | 更新位置信息 | updatePositionEstimator() | 根据IMU及GPS的数据估计飞行器当前位置
7 | 导航控制飞行器 | applyWaypointNavigationAndAltitudeHold() | 前面导航所需的指令和位置都已获得，在这一步进行实际的通道控制

我们来看看它们实际做了什么

### 更新导航状态
该部分主要由```updateWaypointsAndNavigationMode();```来进行调用，该函数的前提是```isRXDataNew```，即接收机任务处理完毕后就会执行，注意不是接收机收到新数据才执行。  
该函数注意做了以下工作，代码里写的清清楚楚  
```c
    //如果解锁了，更新家的位置
    updateHomePosition();

    //更新飞行距离等数据
    updateNavigationFlightStatistics();

    //更新导航状态是否可用，如果导航状态出错则报警
    updateReadyStatus();

    //地面辅助导航模式
    updateFlightBehaviorModifiers();

    //从遥控器模式来修改当前导航状态  执行状态机
    navProcessFSMEvents(selectNavEventFromBoxModeInput());

    //如果处于定高定点模式下，则根据遥控器来调整高度/水平位置，获得控制器所需的目标位置
    processNavigationRCAdjustments();

    //根据导航模式修改飞行模式（导航模式与飞行模式是分开的，注意）
    switchNavigationFlightModes();
```

### 更新位置信息
这一部分```updatePositionEstimator();```主要是根据GPS和IMU（如果有光流传感器的话还会加入光流传感器的数据）来估计当前的位置

### 导航控制飞行器
前面第一步获得了当前所需飞行到的具体位置，信息都记录在```posControl.desiredState```这个结构体中，第二步获得了当前的位置信息等数据，所以当前所需要的就是将第一步和第二步所获得的信息进行计算，将结果放入```rcCommand[]```数组中,所以导航模块的三部分分工很明确。  

导航模块的概览就介绍到这里，这里并不打算继续介绍inav的位置控制器和姿态控制器，深入里面需要先了解相关的控制算法，inav所采用的都是成熟的算法和控制。因此，根据前文的介绍，飞控的流程和功能分布就比较明了了，可以快速定位到自己想了解的东西，同时想要添加功能代码也更方便。下一节，将介绍除飞控主任务外的其他模块，重点说一下接收机及上锁解锁部分。


