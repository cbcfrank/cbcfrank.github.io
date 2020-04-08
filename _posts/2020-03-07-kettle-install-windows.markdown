---
layout: post
title:  "在windows上安装kettle"
date:   2017-08-19 10:06:24 +0800
categories: jekyll update
---

[kettle下载地址](https://sourceforge.net/projects/pentaho)
[kettle9.0下载](https://sourceforge.net/projects/pentaho/files/Pentaho%209.0/client-tools/pdi-ce-9.0.0.0-423.zip/download)

windows上安装jdk
2.安装java环境

先新建两个目录例如：D:\02Java\Java\jdk1.8.0_161,D:\02Java\Java\jre1.8.0_161

运行jdk-8u161-windows-x64.exe，安装过程中会有两次选择目录，第一次选择是存放jdk，第二次选择是存放jre，分别放到刚刚新建的目录

安装完毕后配置环境java环境变量，这三个环境变量再用户级别环境变量添加就行

JAVA_HOME=D:\02Java\Java\jdk1.8.0_161

CLASSPATH=.;%JAVA_HOME%\lib;

PATH=%JAVA_HOME%\bin；

三个环境变量原本没有就新建，PATH原本有的话就再后面添加;%JAVAHOME%\bin；一定要注意多个变量一定要用分号隔开

打开cmd，输入java -version，如下显示

java version "1.8.0_161"
Java(TM) SE Runtime Environment (build 1.8.0_161-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.161-b12, mixed mode)

再输入javac，出现帮助信息则安装成功



