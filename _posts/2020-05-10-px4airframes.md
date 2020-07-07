---  
layout: post  
title:  "PX4机型及混控"  
date:  2020-05-10 14:10:12  
categories: 无人机  
tags: UAV  
---  

* content  
{:toc}  

PX4机型目录位于```\ROMFS\px4fmu_common\init.d\airframes\```    
PX4混控目录位于```\ROMFS\px4fmu_common\mixers\```  
PX4启动脚本位于```\ROMFS|px4fmu_common\init.d\```  

下面以13000标准VTOL垂直起降飞机为例介绍这三个：
### 机型文件  
```shell
#!/bin/sh
#
# @name Generic Quadplane VTOL
#
# @type Standard VTOL
# @class VTOL
#
# @maintainer
#
# @output MAIN1 motor 1
# @output MAIN2 motor 2
# @output MAIN3 motor 3
# @output MAIN4 motor 4
# @output AUX1 Aileron 1
# @output AUX2 Aileron 2
# @output AUX3 Elevator
# @output AUX4 Rudder
# @output AUX5 Throttle
#

sh /etc/init.d/rc.vtol_defaults

if [ $AUTOCNF = yes ]
then
	param set PWM_AUX_DIS5 950
	param set PWM_RATE 400

	param set VT_TYPE 2
	param set VT_MOT_ID 1234
	param set VT_FW_MOT_OFFID 1234
fi

set MAV_TYPE 22

set MIXER quad_x
set MIXER_AUX vtol_AAERT

set PWM_OUT 1234
```

这个文件是标准垂直起降飞机的机型文件，该文件指定了飞机的名称，类型，种类，作者，输出通道，脚本和参数设置  
- ```@name```指定机型的名称
- ```@type```指定机型的种类（在地面站里同一种类的飞机会放在一起供选择）  
- ```@class```指定机型的类型，可以是```VTOL```、```Copter```、```Plane```等  
- ```@output```指定机型的输出通道，该项没有实际用处，不参与实际输出通道的设置，只是供他人阅读了解   

接下来是配置的shell脚本，首先其执行```/etc/init.d/rc.vtol_defaults```该脚本文件，每种类型的机型对应的脚本文件不一样，然后是判断```AUTOCNF```参数如果设置为```yes```，那么就设置下面的参数项  
接下来是设置机型的特殊参数，包括
- ```MAV_TYPE```:[机型的种类](https://mavlink.io/en/messages/common.html#MAV_TYPE)
- ```MIXER```主通道混控文件
- ```MIXER_AUX```辅助通道混控文件
- ```PWM_OUT```要输出的主通道列表等

### 混控文件
请参考[PX4开发者文档-混控说明](http://dev.px4.io/master/en/concept/mixing.html)  
根据机型文件里面指定的两个参数：```MIXER```和```MIXER_AUX```可知具体的混控名称，还是以该机型为例，该机型指定了```quad_x```为主通道混控，```vtol_AAERT```为辅助通道混控。则该机型对应的主通道混控文件为：```quad_x.main.mix```，辅助通道混控文件为：```vtol_AAERT.aux.mix```  

在混控文件夹里面找到对应的两个混控文件：   
主通道混控：```\ROMFS\px4fmu_common\mixers\quad_x.main.mix```  
```
R: 4x 10000 10000 10000 0

AUX1 Passthrough
M: 1
S: 3 5  10000  10000      0 -10000  10000

AUX2 Passthrough
M: 1
S: 3 6  10000  10000      0 -10000  10000

Failsafe outputs
The following outputs are set to their disarmed value
during normal operation and to their failsafe falue in case
of flight termination.
Z:
Z:
```

辅助通道混控：```\ROMFS\px4fmu_common\mixers\vtol_AAERT.aux.mix```  
```
Mixer for an AAERT VTOL
=======================

Aileron 1 mixer
---------------
M: 1
S: 1 0  7500  7500    0 -10000  10000

Aileron 2 mixer
---------------
M: 1
S: 1 0  7500  7500    0 -10000  10000

Elevator mixer
--------------
M: 1
S: 1 1  10000  10000      0 -10000  10000

Rudder mixer
------------
M: 1
S: 1 2 -10000 -10000      0 -10000  10000

Throttle mixer
--------------
M: 1
S: 1 3      0  20000 -10000 -10000  10000
```

混控文件除了可以内置到固件，也可以放到SD卡的```/etc/mixers/```目录下，飞控开机时会优先读取SD卡里的混控，如果SD卡里有混控，则直接使用SD卡内的混控  

#### 混控文件格式
首先是区分注释和实际有用的混控说明： 
- 以"```M```"、"```S```"、"```Z```"、"```R```"、"```O```"等字母加一个冒号```:```开头的视为混控的有效行，系统会读取其内容进行混控
- 除上述格式以外的其他任何格式都视为注释，只供大家方便阅读  

##### "M"和"S"
对通道进行一般的混控通常结合使用```M:```、```O:```和```S:```。  
我们先忽略```O:```这一行。```M:```用于指定混控来源的数量，即```M:```下面的```S:```的数量，```S:```用于指定混控的来源以及混控的正反缩放等。如下面所示M=2，底下有两行S用于指定混控来源  
```
M: 2
S: 1 2 -10000 -10000      0 -10000  10000
S: 0 1  10000  10000      0 -10000  10000
```  
 
对于```S:```，后面具有7个参数，其意义分别是  

参数序号 | 参数名称 | 备注
- | - | -
1 | 控制组Control Group序号(一般为0-6)| [参考网站](http://dev.px4.io/master/en/concept/mixing.html)
2 | 控制通道序号(范围0-7) | [参考网站](http://dev.px4.io/master/en/concept/mixing.html)
3 | 通道的负方向比例 | 10000代表1，4000代表0.4
4 | 通道的正方向比例 | 10000代表1，4000代表0.4
5 | 通道的偏差 | 10000代表1，0代表0
6 | 通道的最小值限定 | 10000代表1，4000代表0.4
6 | 通道的最大值限定 | 10000代表1，4000代表0.4

控制组和控制通道在[PX4开发者网站](http://dev.px4.io/master/en/concept/mixing.html#control-group-0-flight-control)具有详细的说明  
以控制组0和1为例，0-7的控制通道分别代表的含义如下：  
Control Group #0 (Flight Control)  
0: roll (-1..1)  
1: pitch (-1..1)  
2: yaw (-1..1)  
3: throttle (0..1 normal range, -1..1 for variable pitch / thrust reversers)  
4: flaps (-1..1)  
5: spoilers (-1..1)  
6: airbrakes (-1..1)  
7: landing gear (-1..1)   
Control Group #1 (Flight Control VTOL/Alternate)  
0: roll ALT (-1..1)  
1: pitch ALT (-1..1)  
2: yaw ALT (-1..1)  
3: throttle ALT (0..1 normal range, -1..1 for variable pitch / thrust reversers)  
4: reserved / aux0  
5: reserved / aux1  
6: reserved / aux2  
7: reserved / aux3  

对照上面的```S:```描述：```S: 0 1  10000  10000      0 -10000  10000```  
该句混控表示：混控的来源是控制组0的1通道：俯仰；负方向比例为1，正方向比例为1，所以不进行缩放；偏移为0，不进行偏移；上下限分别是-1到1。  

综上，一组输出通道的混控来源可以有很多（数量由M指定），既可以接受自稳控制的俯仰偏航等，也可以接受遥控器的俯仰偏航等信号，通过对每一组S进行变换之后，在对其进行求和，最终输出到一个输出通道即电机或舵机上。  

##### "O"
在某些情况下，```M：```下面加了一行```O:```  
```
M: 2
O:      10000  10000      0 -10000  10000
S: 1 2 -10000 -10000      0 -10000  10000
S: 0 1  10000  10000      0 -10000  10000
```
其中，```O:```用于对整个输出通道进行调整，它比```S:```少了前面两个参数，但是其他参数的含义是一样的。在对多个```S:```进行求和后，再通过```O:```对求和的结果进行缩放、偏移或限定大小。  

##### "Z"
对于一个混控文件，其指定的各个输出通道的来源。一般情况下，输出通道都顺序排列，即混控文件中定义的第一个混控即第一个输出通道，第二个混控即第二个输出通道。但是如果我们想要在中间插一个不用的输出通道，则可以用```Z:```，```Z:```后面什么都不用带，只表明顺序排下来的这个输出通道是空置的。  

##### "R"、"H"等
对于PX4，某些混控已经内置了，为了简单的调用，混控里面的```R:```和```H:```等代表的是内置的混控，这些可以直接参考各个机型的混控，这里不详细描述了  


### 启动脚本
对于该种垂直起降的飞机，在机型文件中调用了启动脚本```rc.vtol_defaults```  
```
#!/bin/sh
#
# VTOL default parameters.
#
# NOTE: Script variables are declared/initialized/unset in the rcS script.
#

set VEHICLE_TYPE vtol

if [ $AUTOCNF = yes ]
then

	# to minimize cpu usage on older boards limit inner loop to 400 Hz
	param set IMU_GYRO_RATEMAX 400

	param set MIS_TAKEOFF_ALT 20
	param set MIS_YAW_TMT 10

	param set MPC_ACC_HOR_MAX 2
	param set MPC_LAND_SPEED 0.7
	param set MPC_TKO_SPEED 1
	param set MPC_VEL_MANUAL 3
	param set MPC_XY_CRUISE 3
	param set MPC_XY_VEL_MAX 4
	param set MPC_Z_VEL_MAX_DN 1.5

	param set NAV_ACC_RAD 3

	param set PWM_AUX_RATE 50
	param set PWM_RATE 400

	param set RTL_LAND_DELAY 0
	param set RTL_TYPE 1

	param set WV_EN 1
fi
```
该文件较为简单，主要是为该种机型设置通用的参数



