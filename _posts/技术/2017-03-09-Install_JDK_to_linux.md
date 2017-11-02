---
layout: blog
technology: true
title:  "linux 安装JDK"
background: red
background-image: https://sarasxu.github.io/winds/images/icon/linux.jpg
date:   2017-03-09 16:13:54
category: 技术
tags:
- java
- linux
- JDK
---

## 卸载JDK

>查看服务器是否安装过JDK

* `java -version` 

*java version "1.6.0_17"*

*OpenJDK Runtime Environment (IcedTea6 1.7.4) (rhel-1.21.b17.el6-i386)*

*OpenJDK Client VM (build 14.0-b16, mixed mode)*


>查看服务器安装的jdk软件包信息

* `rpm -qa |grep gcj` 

*libgcj-4.4.4-13.el6.i686*

*java-1.5.0-gcj-1.5.0.0-29.1.el6.i686*

>卸载软件包

* `yum -y remove java-1.5.0-gcj-1.5.0.0-29.1.el6.i686` 


## 安装JDK7.0

>创建安装目录

* `mkdir -p /usr/lib/jvm`

>解压安装包到安装目录

* `tar zxvf jdk-7u9-linux-i586.tar.gz  -C /usr/lib/jvm` 

>修改文件名

* `mv /usr/lib/jvm/jdk1.7.0_09    /usr/lib/jvm/java7` 

## 添加JDK7.0到系统环境变量

>以免出错先备份

* `cp /etc/profile /etc/profile.bak` 

>进入编辑模式

* `vi /etc/profile`  

>在最后添加下面的内容 

* `export JAVA_HOME=/usr/lib/jvm/java7`
* `export JRE_HOME=${JAVA_HOME}/jre`
* `export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib ` 
* `export PATH=${JAVA_HOME}/bin:$PATH `

使配置文件立即生效

* `source /etc/profile` 

>由于系统中可能会有默认的其他版本JDK，所以，为了将我们安装的JDK设置为默认JDK版本，还要进行如下工作。

* `update-alternatives --install /usr/bin/java java /usr/lib/jvm/java7/bin/java 300`  
* `update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/java7/bin/javac 300` 
* `update-alternatives --install /usr/bin/jar jar /usr/lib/jvm/java7/bin/jar 300`  
* `update-alternatives --install /usr/bin/javah javah /usr/lib/jvm/java7/bin/javah 300`  
* `update-alternatives --install /usr/bin/javap javap /usr/lib/jvm/java7/bin/javap 300` 

>执行下面命令，设置默认版本，此命令执行后，系统会列出当前存在的各种JDK版本，会提示你选择

* `update-alternatives --config java`

## 测试

* `java -version` 

*java version "1.7.0_09"*

*Java(TM) SE Runtime Environment (build 1.7.0_09-b05)*

*Java HotSpot(TM) Client VM (build 23.5-b02, mixed mode)*