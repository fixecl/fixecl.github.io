---  
layout: post  
title:  "Ubuntu18.04安装注意事项"  
date:  2021-06-30 21:10:12  
categories: linux  
tags: linux ubuntu  
---  

* content  
{:toc}  

### Ubuntu-gnome-插件安装

安装插件程序

```bash  
sudo apt-get install gnome-tweak-tool
sudo apt-get install gnome-shell-extensions
sudo apt-get install chrome-gnome-shell  #（浏览器插件）
```  

安装结束后进设置-详细信息看一下gnome-shell-extensions版本

打开FireFox或者Chrome，添加gnome插件

打开https://extensions.gnome.org/，点击想要的插件选择版本，然后打开右上角的开关就可以

推荐三个插件：

Dash to dock（把dock移到底部）

Hide top bar（顶部状态栏可以自动隐藏）

User themes（安装自定义主题必须）



### Ubuntu-gnome-主题图标等美化

进入网站https://www.gnome-look.org/ 下载喜欢的主题和图标

主题复制到/usr/share/themes文件夹下

图标复制到/usr/share/icons文件夹下

shell主题点击左侧按钮，选择下下来的zip即可

这两个文件夹都需要系统权限，可以sudo cp或sudo nautilus 打开文件管理器复制

个人喜欢：

主题：McOS-HS-(transparent)

图标：Flat-Remix

Shell：Flat-Remix

复制时注意不要复制多重目录



### Ubuntu中文搜狗输入法

18.04默认框架是iBUS，搜狗输入法不支持

先安装fcitx框架：
```bash  
sudo apt-get install fcitx-bin
```
再安装fcitx基础输入法：

```bash  
sudo apt-get install fcitx-table
```

重启，这时候就可以用拼音输入法了

下载搜狗输入法，安装，重启，切换即可



### Ubuntu减少开机grub的等待时间
```bash
sudo gedit /etc/default/grub
```

修改GRUB_TIMEOUT=1 或者自己想要的时间



### Ubuntu开机自动打开小键盘

安装一个小软件
```bash  
sudo apt install numlockx
```  
加入开机启动
```bash  
sudo gedit /usr/share/lightdm/lightdm.conf.d/50-ubuntu.conf
```  
最后一行加入
```bash  
greeter-setup-script=/usr/bin/numlockx on
```  


### Ubuntu修改时间为本地时间

默认情况下如果安装双系统，Ubuntu的时间会与Windows相差8小时，用以下三行命令修改Ubuntu的时间为本地时间，与Windows一致
```bash  
sudo apt-get install ntpdate
sudo ntpdate time.windows.com
sudo hwclock --localtime --systohc
```  

### Ubuntu其他软件安装

WPS：下deb包，单独缺少的几个字体

XDM：Linux下可以媲美迅雷的软件，下载./install/sh安装即可


### 修改主文件夹中的语言

Ubuntu安装时选择成中文版时，会自动把主文件夹下的各个文件夹设置为中文，这对很多软件的默认目录不友好。修改成英文如下：
```bash  
export LANG=en_US
xdg-user-dirs-gtk-update
```  
同意将目录转为英文路径
```bash  
export LANG=zh_CN
```  
下次重启时选择不转为中文即可



### pip安装

python2.7和python3分别各自使用pip和pip3

pip（3）下载包的方式较慢，可以修改源
```bash  
cd ~
mkdir .pip
cd .pip
gedit pip.conf
```  
在文件中加入：
```bash  
[global]
index-url = http://mirrors.aliyun.com/pypi/simple/
[install]
trusted-host = mirrors.aliyun.com
```  


### Java安装

推荐在Oracle官网下载手动安装的方式

下载解压，复制到/opt或者/usr/share

修改本用户的环境变量

gedit ~/.bashrc

在最后加入：
```bash  
#Java
export JAVA_HOME=/opt/jdk1.8.0_201 （根据自己的版本和安装路径修改）
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```  
启用环境变量
```bash  
source ~/.bashrc
```  


### NV显卡驱动更新

在某些情况下，需要更新NV显卡驱动。默认附加驱动里面的已经测试过的驱动（tested）比较旧，可选择较为新的驱动（如果要安装CUDA和cuDNN注意特定驱动版本的选择，这三者之间有相互适配个关系）

获取更新的显卡驱动
```bash  
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update
```  
这时候在附加驱动菜单或者使用ubuntu-drivers devices就能看到新的可用驱动了
