---
layout: post
title:  "搭建PX4开发环境"
date:  2019-10-22 13:30:53
categories: 硬件
tags: UAV ubuntu
---

* content
{:toc}

本文基于PX4官方教程：
[PX4-Developer](https://dev.px4.io/master/zh/index.html)
[PX4-Env](http://dev.px4.io/v1.9.0/zh/setup/dev_env_linux.html)
基于Ubuntu18.04

### 安装及配置

##### 权限
为防止串口连接问题，而tty设备属于dialout用户组，因此需要将用户加入dialout用户组
```
sudo usermod -a -G dialout $USER
```
注销，重新登陆
既然已经把用户加入了该组，那么之后就避免使用root用户来进行开发和操作

##### 安装依赖
下面安装所需依赖包(后面加参数-y，安装时就不再确认了)
```
sudo add-apt-repository ppa:george-edison55/cmake-3.x -y
sudo apt update
# 必备软件
sudo apt install python-argparse git-core wget zip \
    python-empy qtcreator cmake build-essential genromfs -y
```

##### 安装仿真包
安装仿真工具包，如果事先安装有java了就不用安装openjdk了
```
# 仿真工具
sudo add-apt-repository ppa:openjdk-r/ppa
sudo apt update
sudo apt install openjdk-8-jre
sudo apt install ant protobuf-compiler libeigen3-dev libopencv-dev openjdk-8-jdk openjdk-8-jre clang-3.5 lldb-3.5 -y
```

##### 安装arm-none-eabi-工具链
如果系统有高版本的工具链，卸载之
下载该版本：https://launchpad.net/gcc-arm-embedded/5.0/5-2016-q2-update/+download/gcc-arm-none-eabi-5_4-2016q2-20160622-linux.tar.bz2
解压,添加环境变量
```
tar -jxf gcc-arm-none-eabi-5_4-2016q2-20160622-linux.tar.bz2
gedit /etc/profile
```
最后一行加入
```
export PATH=$PATH:/usr/..../gcc-arm-none-eabi-.../bin
```
环境变量生效
```
source /etc/profile
```

##### 安装pip，安装python包
```
sudo apt install python-pip
pip install jinja2 toml numpy pyyaml
```


### 下载PX4代码
直接git clone,不要使用zip压缩包，找一个网络环境好的地方
```
git clone https://github.com/PX4/Firmware.git
cd Firmware
git submodule update --init --recursive
```

### 测试模拟器
```
make px4_sitl jmavsim
```
等待一段时间后将会出现一个3D渲染的无人机飞行窗口
