---  
layout: post  
title:  "搭建PX4开发环境"  
date:  2019-10-22 13:30:53  
categories: 无人机 
tags: UAV ubuntu  
---  

* content  
{:toc}  

本文基于PX4官方教程：  
[PX4-Developer](https://dev.px4.io/master/zh/index.html)  
[PX4-Env](http://dev.px4.io/v1.9.0/zh/setup/dev_env_linux.html)  
本文基于Ubuntu18.04，Pixhawk1.9.2(注意不同的版本所适用的gcc版本不一样)  

## 系统配置  

### 系统配置  
为防止串口连接问题，而tty设备属于dialout用户组，因此需要将用户加入dialout用户组  
``` bash 
sudo usermod -a -G dialout $USER  
```  
同时，卸载modemManager  
```bash
sudo apt remove modemmanager -y
```
注销，重新登陆  
既然已经把用户加入了该组，那么之后就避免使用root用户来进行开发和操作  

### 安装依赖  
下面安装所需依赖包(后面加参数-y，安装时就不再确认了)  
``` bash 
sudo apt update  
# 必备软件  
sudo apt install git-core wget zip qtcreator cmake build-essential genromfs -y  
#安装支持库  
sudo apt-get install openocd flex bison libncurses5-dev autoconf \  
    texinfo build-essential libftdi-dev libtool zlib1g-dev  -y  
```  

### 安装仿真包  
安装仿真工具包，官网推荐安装openjdk，我使用openjdk无法打开jMAVSim，还是选择安装Orcale的Jdk  
#### 安装openjdk  
```bash  
# 仿真工具 添加openjdk源可选  
# sudo add-apt-repository ppa:openjdk-r/ppa  
# sudo apt update  
sudo apt install openjdk-8-jre  
sudo apt install ant protobuf-compiler libeigen3-dev libopencv-dev openjdk-8-jdk openjdk-8-jre -y  
sudo apt install clang lldb -y  
```  
#### 安装Orcale JDK  
下载压缩包解压;  
在```/etc/profile```中添加环境变量;  

安装依赖包  
```  bash
sudo apt install ant protobuf-compiler libeigen3-dev libopencv-dev -y  
sudo apt install clang lldb -y  
```  

### 安装gcc工具链  
#### 直接安装  
现在高版本也可以直接编译了（实测2017q2可以）  
再Ubuntu下可以直接安装  
``` bash 
sudo apt install gcc-arm-none-eabi  
```  
#### 手动安装（高版本代码）
非Ubuntu系统可以自己下载安装(经测试gcc6以上才支持PX4 1.9.2)  
https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads
解压,添加环境变量  
``` bash 
tar -jxf gcc-arm-none-eabi-6-2017q2-update-linux.tar.bz2  
sudo mv gcc-arm-none-eabi-6-2017-q2-update /opt  
sudo gedit /etc/profile  
```  
最后一行加入  
``` bash 
export PATH=$PATH:/opt/gcc-arm-none-eabi-6-2017-q2-update/bin  
```  
环境变量生效  
``` bash 
source /etc/profile  
```  

#### 手动安装(低版本代码)  
下载官方推荐的5.4版本：https://launchpad.net/gcc-arm-embedded/5.0/5-2016-q2-update/+download/gcc-arm-none-eabi-5_4-2016q2-20160622-linux.tar.bz2  
解压
``` bash 
tar -jxf gcc-arm-none-eabi-5_4-2016q2-20160622-linux.tar.bz2  
sudo mv gcc-arm-none-eabi-5_4-2016q2 /opt  
sudo gedit /etc/profile  
```  
添加环境变量，最后一行加入  
``` bash 
export PATH=$PATH:/opt/gcc-arm-none-eabi-5_4-2016q2/bin  
```  
环境变量生效  
``` bash 
source /etc/profile  
```  
如果在不同的终端中编译，最好重启一下  

### 安装pip，安装python包  
``` bash 
sudo apt install python-pip  
sudo pip install jinja2 toml numpy==1.16 pyyaml pyserial empy argparse  
```  
注意！ 系统默认python版本为2.7，而有的包目前已经不支持python2.7了，所以需要限制版本。  
比如对于numpy，numpy1.17以上就不支持python2.7了，所以安装方式如下，其他所需包也一样  
``` bash 
pip install numpy==1.16  
```  

### 新版固件(1.10~)：安装pip3，安装python包  
``` bash 
sudo apt install python3-pip  
sudo pip3 install jinja2 toml numpy pyyaml pyserial empy argparse  
```  
注意！ 系统默认python版本为2.7，而有的包目前已经不支持python2.7了，所以需要限制版本。  
比如对于numpy，numpy1.17以上就不支持python2.7了，所以安装方式如下，其他所需包也一样  
``` bash 
pip install numpy==1.16  
```  

## 下载PX4代码  
直接git clone,不要使用zip压缩包，找一个网络环境好的地方，或者开VPN  
(我的方法是在windows下开ssr，然后用WSL环境git下来，打成tar包，再拷贝到虚拟机里面解压，不直接用WSL编译是因为WSL编译速度较慢，虚拟机的ubuntu更快)  
``` bash 
git clone https://github.com/PX4/Firmware.git  
cd Firmware  
git submodule init  
git submodule update --init --recursive  
# 初始化并clone子模块，recursive参数表明递归进行  
```  
PX4 1.9.2固件完整clone下来大概有836M，所以在国内网络条件一定要好  

## 测试模拟器  
``` bash 
make px4_sitl jmavsim  
```  
等待一段时间后将会出现一个3D渲染的无人机飞行窗口  

## 编译固件  
```  bash
make px4_fmu-v2_default  
```  
最后编译到100%时链接会报错，这是因为FMUv2的flash只有1M，不够用，需要对其进行裁剪  
如果到100%就表示成功，链接报错可以通过修改配置文件，裁剪一些功能来解决
