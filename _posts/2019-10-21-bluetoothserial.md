---  
layout: post  
title:  "蓝牙串口HC-05使用及调试"  
date:   2019-10-21 16:21:23  
categories: 硬件  
tags: bluetooth serial  
---  

* content  
{:toc}  

## 默认设置  

模块工作角色：从模式  
串口参数：38400bits/s 停止位 1位 无校验位  
默认配对码：1234  

## 用USB转TTL模块设置  

### 蓝牙与USB转TTL模块连接方式  
RXD-TX  
TXD-RX  
注意电压  
EN引脚默认会连接到一个按键，拉高会进入AT模式  

### 进入AT调试模式  
首先让AT引脚置高，然后接上蓝牙模块（有按键则按住蓝牙上的按键，再接通电源，进入AT指令模式），这时候将转串口模块接入电脑，当蓝牙模块state灯变为慢闪，则表明已经进入AT模式。打开串口调试助手便可以开始设置AT模式。（具体AT指令参考HC05 AT指令集）  

### 配置蓝牙 （发送以下AT指令后返回OK表示设置成功）  
#### 恢复默认设置：  
将蓝牙恢复默认设置：AT+ORGL\r\n（\r\n即回车、换行，在串口调试助手上输入一个回车即可）  
#### 设置蓝牙名称：  
配置蓝牙的名称：AT+NAME=Bluetooth-Marster\r\n（主）或 Bluetooth-Slave\r\n（从） 蓝牙名称为Bluetooth-Marster 或 Bluetooth-Slave  
#### 设置配对码：  
配置蓝牙的配对码：AT+PSWD=1212\r\n（蓝牙A与蓝牙B的配对码相同，这样才能成功配对）  
#### 设置工作角色：  
将蓝牙A配置为主机模式：AT+ROLE=1\r\n，并将将蓝牙B配置为从机模式：AT+ROLE=0\r\n  
#### 配置串口参数：  
配置波特率、停止位和校验位：AT+UART=115200,0,0\r\n，设置蓝牙通信串口波特率为9600，停止位1位，无校验位  
#### 查询地址：  
查询蓝牙地址：AT+ADDR=？\r\n（如2015:2:120758）  
#### 清空配对列表：  
清空配对列表，方便配对新的蓝牙：AT+RMAAD\r\n  
#### 连接模式：  
配置蓝牙连接模式，若为任意地址连接模式则配置为0，无需进行地址绑定，否则配置为1，需要进行地址绑定：AT+CMODE=0\r\n（蓝牙连接模式为任意地址连接模式）  
#### 蓝牙A绑定蓝牙B：  
蓝牙A绑定蓝牙B地址：AT+BIND=2015,2,120758\r\n（注意把地址的冒号换成逗号）  

