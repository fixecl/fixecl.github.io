---  
layout: post  
title:  "搭建WSL的inav固件开发环境"  
date:  2019-10-24 22:02:12  
categories: 硬件  
tags: UAV ubuntu inav  
---  

* content  
{:toc}  

inav固件常用于F3 F4飞控上，其有出色的导航能力，适用于多旋翼和固定翼的飞行  
inav固件与CleanFilght BetaFlight关系紧密，所以下文对另外两种固件也基本适用  
本文主要介绍inav的开发环境搭建，基于Windows10 + WSL + VSCode开发  

### 系统环境搭建  
在搜索栏中搜索并打开“启用或关闭Windows功能”，勾选“适用于Linux的Windows子系统”项  
打开win10应用商店，搜索ubuntu，下载18.04版本，安装  
安装完成后主要需要修改源，修改```/etc/apt/sources.list```文件内容为：  

``` bash
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse  
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic main restricted universe multiverse  
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse  
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-updates main restricted universe multiverse  
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse  
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-backports main restricted universe multiverse  
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse  
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-security main restricted universe multiverse  
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse  
deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ bionic-proposed main restricted universe multiverse  
```  

### 安装交叉编译工具  
安装make工具  
``` bash 
sudo apt install make  
```  
#### 安装gcc方法1 直接更新  
首先更新源  
``` bash 
sudo apt update  
```  
安装交叉编译工具  
``` bash 
sudo apt install gcc-arm-none-eabi  
```  
#### 安装gcc方法2 下载deb包安装  
本人使用的版本下载地址：  
http://ppa.launchpad.net/team-gcc-arm-embedded/ppa/ubuntu/pool/main/g/gcc-arm-none-eabi/gcc-arm-embedded_7-2018q2-1~bionic1_amd64.deb  
其他版本未确认是否可用  
然后使用dpkg安装该包  

### 尝试对inav进行编译  
下载inav源代码  
github地址： https://github.com/iNavFlight/inav  
在win10中解压到本地，路径上最好不要有中文名（只是为了减少不必要的麻烦）  
进入inav目录内，按住shift，右击，在快捷菜单中选择“在此处打开Linux Shell”  
输入make，在我的笔记本上约1分钟后编译完成（WSL下编译比较慢，相比之下，直接在Linux下直接编译会快几倍哦）  
![Build]({{site.baseurl}}/images/inavenvsetup/build.png)  
至此编译成功  
不过会有消息```fatal: 不是一个 git 仓库（或者直至挂载点 /mnt 的任何父目录）```，其实可以不用管，如果想要去掉可以了解下git的知识就好了  

### 配置编辑环境  
#### windows安装vscode  
去官网下载vscode，安装以下插件  
Chinese中文插件  
C/C++  
Cortex-Debug  (用于代码调试)  
Remote-WSL   (可打开WSL终端)  
ARM  
C++ Intellisense  

安装完成后，使用VSCode打开inav的文件夹，接下来，在VSCode下面的终端中点击右侧的下拉箭头，选择“选择默认Shell” -> “WSL Bash”，再点击右侧的“+”号，即可新建WSL的终端，在终端中输入```make```，输出结果应该跟之前一致。  

inav默认生成的固件是REVO版本，不是对应常见的飞控，下面是inav支持的飞控种类  
```  
ALIENFLIGHTF3 ALIENFLIGHTF4 AIRHEROF3 AIRHEROF3_QUAD COLIBRI_RACE  
LUX_RACE SPARKY REVO SPARKY2 COLIBRI FALCORE FF_F35_LIGHTNING FF_FORTINIF4  
FF_PIKOF4 FF_PIKOF4OSD SPRACINGF3 SPRACINGF3EVO SPRACINGF3EVO_1SS  
SPRACINGF3MINI SPRACINGF3NEO SPRACINGF4EVO CLRACINGF4AIR CLRACINGF4AIRV2  
BEEROTORF4 BETAFLIGHTF3 BETAFLIGHTF4 PIKOBLX OMNIBUS AIRBOTF4 BLUEJAYF4  
OMNIBUSF4 OMNIBUSF4PRO OMNIBUSF4V3 FIREWORKSV2 SPARKY2 MATEKF405  
OMNIBUSF7 DYSF4PRO OMNIBUSF4PRO_LEDSTRIPM5 OMNIBUSF7NXT OMNIBUSF7V2  
ASGARD32F4 ANYFC ANYFCF7 ANYFCF7_EXTERNAL_BARO ANYFCM7 ALIENFLIGHTNGF7  
PIXRACER YUPIF4 YUPIF4MINI YUPIF4R2 YUPIF7 MATEKF405SE MATEKF411  
MATEKF722 MATEKF405OSD MATEKF405_SERVOS6  
```  
如果需要编译对应飞控的固件，只需要使用```make + 飞控种类```即可  
比如常见的F3飞控：```make SPRACINGF3```  
如果需要修改默认目标飞控  
打开"/Makefile"  
修改TARGET即可  
``` makefile
TARGET    ?= SPRACINGF3  
```  

至此，编辑环境创建成功，改好代码后，直接```make + 飞控种类```即可  

### 配置调试环境  
要创建调试环境，则需要有调试器，常见的调试器有stlink,Jlink等，我是用的Jlink,因此以Jlink举例，stlink大同小异  
#### Jlink配置  
首先下载安装Win10 Jlink驱动，将Jlink上对应的端口连接到飞控的SWD调试端口上。飞控调试端口一般有SWCLK，SWDIO，NRST，GND等，把这4个端口连接到Jlink对应端口。上电。  
打开SEGGER J-Flash程序，选择“new project”，Target Device选择自己的飞控型号，比如我的是F3飞控，则选择“STM32F303CC”，其他参数默认，确定  
然后“File”->“Open Date File”，选择现有的固件，然后“Target”->“Connect”，连接成功后按F7即写入  
如果写入成功，证明Jlink及Jlink与飞控的连接没有问题  

#### 配置VSCode调试环境  
在左侧打开VSCode调试页，“添加配置”->"Cortex Debug"，修改右侧配置文件  
``` json 
"name": "Cortex Debug",  
"cwd": "${workspaceRoot}",  
"executable": "${workspaceRoot}/obj/main/inav_SPRACINGF3.elf",  
"request": "launch",  
"type": "cortex-debug",  
"servertype": "jlink",  
"device": "STM32F303CC",  
"interface": "swd"  
```  
按F5即可进入调试  
