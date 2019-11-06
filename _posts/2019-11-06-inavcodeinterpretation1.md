---  
layout: post  
title:  "inav代码解读1 - 初步了解软硬件"  
date:  2019-11-06 11:10:45  
categories: 无人机  
tags: UAV inav  
---  

* content  
{:toc}  

inav固件是由CleanFlight和BetaFlight发展而来，其更加注重于导航功能，同时也加强了固定翼的飞行能力。此外，inav飞控一般运行于单处理器的F3 F4飞控上，相比于Pixhawk，系统更加简单清晰明了，成本更低，也更适合低成易于开发和修改。  
这一系列文章基于飞控硬件SPRACING F3；软件版本inav2.0.0。其他比如F4等硬件大同小异。对于CleanFlight和BetaFight都有参考意义。  

## 飞控硬件图
飞控硬件图：  
![F3]({{site.baseurl}}/images/inavcode/SPRACINGF3.png) 

- 飞控主控为STM32F303CC
- IMU(惯性测量单元)包括MPU6050(三轴陀螺仪加速度计);HMC5883L(磁力计/罗盘)
- MS5611(气压计)
- W25Q64 8M的SPI Flash
- CP2012 USB转串口
- 直流稳压芯片和蜂鸣器放大电路

飞控的核心是主控和IMU，在F3飞控中，主控STM32与各传感器通讯都是采用IIC协议，通过IIC协议获取陀螺仪，加速度计，磁力计，气压计的数值，对飞行器进行姿态解算后，控制电机和舵机输出  

## 代码层次图
inav的代码可以从github上直接下载，详细安装参考前面的文章  
打开inav的源代码，内容主要为以下层次

```bash
.
├── AUTHORS
├── build_docs.sh
├── build.sh
├── CODE_OF_CONDUCT.md
├── CONTRIBUTING.md
├── Dockerfile
├── docs
├── fake_travis_build.sh
├── install-toolchain.sh
├── JLinkSettings.ini
├── lib    #stm32标准库，HAL库，USB库，MAVLink库等
│   ├── main
│   │   ├── CMSIS
│   │   │   ├── Core
│   │   │   └── DSP
│   │   ├── MAVLink
│   │   ├── STM32F3
│   │   │   ├── Drivers
│   │   │   │   ├── CMSIS
│   │   │   │   ├── STM32F30x_StdPeriph_Driver
│   │   │   │   └── STM32F3xx_HAL_Driver
│   │   │   └── Middlewares
│   │   │       └── ST
│   │   │           └── STM32_USB_Device_Library
│   │   ├── STM32F4
│   │   ├── STM32F7
│   │   ├── STM32_USB_Device_Library
│   │   ├── STM32_USB-FS-Device_Driver
│   │   └── STM32_USB_OTG_Driver
│   └── test
├── LICENSE
├── make   #分处理器的makefile文件
│   ├── mcu
│   │   ├── STM32F3.mk
│   │   ├── STM32F4.mk
│   │   └── STM32F7.mk
│   ├── release.mk
│   └── source.mk
├── Makefile   #总的Makefile文件
├── Notes.md
├── obj
├── README.md
├── src    #飞控源代码
│   ├── main  #几乎所有源代码都在这个文件夹内，我们主要看这个文件夹内的代码
│   │   ├── blackbox   #黑盒子，用于飞行时进行日志记录
│   │   ├── build      #飞控的版本信息，调试选项等
│   │   ├── cms
│   │   ├── common     #公用的函数，包括Math，CRC，字符串等
│   │   ├── config     #各个模块的配置
│   │   ├── drivers    #各个传感器，模块，以及IIC SPI等的驱动
│   │   ├── fc         #飞行控制核心代码
│   │   ├── flight     #飞行时的底层库，包括pid，混控，姿态解算，位置估计等
│   │   ├── io         #飞控的外设输入输出模块，包括GPS，显示屏，蜂鸣器，光流传感器等
│   │   ├── main.c     #main函数，入口
│   │   ├── msp        #msp协议
│   │   ├── navigation #导航模块，包括所有的定位，导航功能控制
│   │   ├── platform.h 
│   │   ├── rx         #接收机驱动，支持多种接收机包括sbus，msp，spektrum，nrf24l01等
│   │   ├── scheduler  #任务调度器
│   │   ├── sensors    #各类传感器模块
│   │   ├── startup    #启动代码，汇编
│   │   ├── target     #针对编译目标飞控的端口定义，功能裁剪
│   │   │   ├── SPRACINGF3
│   │   │   │   ├── target.c
│   │   │   │   ├── target.h
│   │   │   │   └── target.mk
│   │   │   ├── SPRACINGF3NEO
│   │   │   ├── SPRACINGF4EVO
│   │   │   ├── ........
│   │   │   ├── common.h
│   │   │   ├── common_hardware.c
│   │   │   ├── common_post.h
│   │   │   ├── stm32f7xx_hal_conf.h
│   │   │   ├── system_stm32f30x.c
│   │   │   ├── system_stm32f30x.h
│   │   │   ├── system_stm32f4xx.c
│   │   │   ├── system_stm32f4xx.h
│   │   │   ├── system_stm32f7xx.c
│   │   │   ├── system_stm32f7xx.h
│   │   │   ├── YUPIF4
│   │   │   └── YUPIF7
│   │   ├── telemetry          #数传模块
│   │   ├── uav_interconnect   #无人机交互连接功能
│   │   ├── vcp
│   │   ├── vcpf4
│   │   └── vcp_hal
│   ├── test
│   └── utils
├── support
│   ├── buildserver
│   ├── flash.bat
│   └── stmloader   #stm32 bootloader
└── Vagrantfile
```

对于初次接触该固件源代码或初次接触大型工程的同学可能会有点不知所措，但是只要我们依照上图查找功能所在对于目录就便于理解了  
对于我们梳理inav飞行控制逻辑，我们需要重点关注如下目录：  
```bash
./src/main/ #飞控源代码目录
./src/main/fc/ #飞行控制代码
./src/main/navigation/ #导航部分控制代码
```
