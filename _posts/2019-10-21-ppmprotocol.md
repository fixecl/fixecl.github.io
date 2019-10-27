---  
layout: post  
title:  "无人机接收机PPM协议"  
date:   2019-10-21 23:00:53  
categories: 硬件  
tags: UAV  
---  

* content  
{:toc}  

### PPM协议  
无人机遥控器与飞控之间传输最常用的就是PWM，PPM，S.BUS等协议。其中PPM协议只需要一根线就可以传输，避免了PWM需要多路才能传输多个通道的问题。  
下面是PPM协议的波形图  
![PPM]({{site.baseurl}}/images/ppmprotocol/ppm.png)  
PPM协议最多传输20个通道，使用一个定时器就可以轻松解决了  

### PPM协议用于模拟器  
如果是自己制作遥控器，想要添加遥控器的模拟器功能，像商品遥控器那样连上加密狗就可以畅玩模拟器，就需要给加密狗传输PPM信号啦  
加密狗的传输线一般是音频线，外面的圆柱连接地线GND，内芯传输PPM信号，信号如上图所示  
