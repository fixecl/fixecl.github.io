---  
layout: post  
title:  "inav代码解读4 - inav的导航状态"  
date:  2019-11-13 09:09:58  
categories: 无人机  
tags: UAV inav  
---  

* content  
{:toc} 

接上一节，inav的导航模块内有完整的导航状态机，可以在多种模式中进行转换，比如从定高模式切换到返航模式，从定点模式切换到紧急降落模式  

### 飞控导航状态机
有限状态机（英语：finite-state machine，缩写：FSM）又称有限状态自动机，简称状态机，是表示有限个状态以及在这些状态之间的转移和动作等行为的数学模型。在飞控导航里，每一种状态即一种模式下的飞行状态。比如，在返航RTH模式就分为：初始化状态；返航爬升状态；改变朝向状态；返航到起飞位置（家位置）状态；降落状态；正在结束返航状态；返航结束状态等。据此，可以画出状态转移图：  
![RTH]({{site.baseurl}}/images/inavcode/rthstates.png)   
当满足条件时转移到下一状态，而对于不同的模式，则添加更多的状态  
![RTH]({{site.baseurl}}/images/inavcode/rthposhold.png)
当加入多个模式时，不同模式之间的转换，都是转换到新模式的初始化步骤

### 代码实现
上面简单介绍了一下导航状态机，下面来看一下代码的实际实现  
```c
static const navigationFSMStateDescriptor_t navFSM[NAV_STATE_COUNT] = {
    /** Idle state ******************************************************/
    [NAV_STATE_IDLE] = {
        .persistentId = NAV_PERSISTENT_ID_IDLE,
        .onEntry = navOnEnteringState_NAV_STATE_IDLE,
        .timeoutMs = 0,
        .stateFlags = 0,
        .mapToFlightModes = 0,
        .mwState = MW_NAV_STATE_NONE,
        .mwError = MW_NAV_ERROR_NONE,
        .onEvent = {
            //当开关拨到定高时，将转换为定高模式的初始化状态
            [NAV_FSM_EVENT_SWITCH_TO_ALTHOLD]           = NAV_STATE_ALTHOLD_INITIALIZE, 
            [NAV_FSM_EVENT_SWITCH_TO_POSHOLD_3D]        = NAV_STATE_POSHOLD_3D_INITIALIZE,
            [NAV_FSM_EVENT_SWITCH_TO_RTH]               = NAV_STATE_RTH_INITIALIZE,
            [NAV_FSM_EVENT_SWITCH_TO_WAYPOINT]          = NAV_STATE_WAYPOINT_INITIALIZE,
            [NAV_FSM_EVENT_SWITCH_TO_EMERGENCY_LANDING] = NAV_STATE_EMERGENCY_LANDING_INITIALIZE,
            [NAV_FSM_EVENT_SWITCH_TO_LAUNCH]            = NAV_STATE_LAUNCH_INITIALIZE,
            [NAV_FSM_EVENT_SWITCH_TO_CRUISE_2D]         = NAV_STATE_CRUISE_2D_INITIALIZE,
            [NAV_FSM_EVENT_SWITCH_TO_CRUISE_3D]         = NAV_STATE_CRUISE_3D_INITIALIZE,
        }
    },
    //省略部分代码...
    [NAV_STATE_RTH_INITIALIZE] = {
        .persistentId = NAV_PERSISTENT_ID_RTH_INITIALIZE,
        .onEntry = navOnEnteringState_NAV_STATE_RTH_INITIALIZE,
        .timeoutMs = 10,
        .stateFlags = NAV_CTL_ALT | NAV_CTL_POS | NAV_CTL_YAW | NAV_REQUIRE_ANGLE | NAV_REQUIRE_MAGHOLD | NAV_REQUIRE_THRTILT | NAV_AUTO_RTH,
        .mapToFlightModes = NAV_RTH_MODE | NAV_ALTHOLD_MODE,
        .mwState = MW_NAV_STATE_RTH_START,
        .mwError = MW_NAV_ERROR_NONE,
        .onEvent = {
            //超时则表明还没有完成该状态的任务，重新进入该状态  
            [NAV_FSM_EVENT_TIMEOUT]                     = NAV_STATE_RTH_INITIALIZE,  
            //成功则进入下一状态
            [NAV_FSM_EVENT_SUCCESS]                     = NAV_STATE_RTH_CLIMB_TO_SAFE_ALT,
            [NAV_FSM_EVENT_SWITCH_TO_EMERGENCY_LANDING] = NAV_STATE_EMERGENCY_LANDING_INITIALIZE,
            [NAV_FSM_EVENT_SWITCH_TO_RTH_LANDING]       = NAV_STATE_RTH_HOVER_PRIOR_TO_LANDING,
            [NAV_FSM_EVENT_SWITCH_TO_IDLE]              = NAV_STATE_IDLE,
        }
    },
    //省略剩余代码
```  
而我们来看返航初始化的代码，与上面的定义一一对应，因此整个导航状态机都是在严格的状态转换中  
```c
static navigationFSMEvent_t navOnEnteringState_NAV_STATE_RTH_INITIALIZE(navigationFSMState_t previousState)
{
    navigationFSMStateFlags_t prevFlags = navGetStateFlags(previousState);

    if ((posControl.flags.estHeadingStatus == EST_NONE) || (posControl.flags.estAltStatus == EST_NONE) || (posControl.flags.estPosStatus != EST_TRUSTED) || !STATE(GPS_FIX_HOME)) {
        // 导航有问题切换到紧急降落模式
        return NAV_FSM_EVENT_SWITCH_TO_EMERGENCY_LANDING;
    }

    if (STATE(FIXED_WING) && (posControl.homeDistance < navConfig()->general.min_rth_distance) && !posControl.flags.forcedRTHActivated) {
        // 不满足最小返航要求进入空闲状态
        return NAV_FSM_EVENT_SWITCH_TO_IDLE;
    }

    // If we have valid position sensor or configured to ignore it's loss at initial stage - continue
    if ((posControl.flags.estPosStatus >= EST_USABLE) || navConfig()->general.flags.rth_climb_ignore_emerg) {
        // 省略部分代码

        // 离家很近则直接降落
        if (posControl.homeDistance < navConfig()->general.min_rth_distance) {
            //省略操作代码
            return NAV_FSM_EVENT_SWITCH_TO_RTH_LANDING;   // NAV_STATE_RTH_HOVER_PRIOR_TO_LANDING
        }
        else {
            //省略初始化操作代码

            //进入返航爬升状态
            return NAV_FSM_EVENT_SUCCESS;  
        }
    }
    /* Position sensor failure timeout - land */
    else if (checkForPositionSensorTimeout()) {
        // 传感器出问题，进入紧急降落模式
        return NAV_FSM_EVENT_SWITCH_TO_EMERGENCY_LANDING;
    }
    else {
        //其他情况
        return NAV_FSM_EVENT_NONE;
    }
}
```

