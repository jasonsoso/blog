---
title: CentOS7安装开发软件的备忘录
date: 2022/4/11 19:46:25
tags:
    - CentOS
    - 软件
    - JDK
categories: 备忘录
---


## CentOS7安装JDK8

### 下载与解压JDK

1. 下载
   前往[https://www.injdk.cn/](https://www.injdk.cn/)进行下载（懒得去Oracle注册了）
   ``` bash
   wget https://d6.injdk.cn/oraclejdk/8/jdk-8u301-linux-x64.tar.gz
   ```

2. 解压
   ``` bash
   mkdir -p /usr/local/java
   tar zxvf jdk-8u301-linux-x64.tar.gz -C /usr/local/java
   ``` 

### 配置环境变量

1. 复制以下到此文件（/etc/profile）
   ``` bash
   JAVA_HOME=/usr/local/java/jdk1.8.0_301
   export CLASSPATH=$:CLASSPATH:$JAVA_HOME/lib/
   export PATH=$PATH:$JAVA_HOME/bin
   ```
   
2. 编辑，并使之生效
   ``` bash
   vi /etc/profile
   source /etc/profile
   ``` 

### 检查是否安装成功

1. 检查命令
   ``` bash
   java -version
   javac -version
   ```
    ![检查JDK是否安装成功](http://tech.jasonsoso.com/images/202204/jdk-1.png "检查JDK是否安装成功")


