---
title: CentOS7安装一些软件的备忘录
date: 2022/4/10 19:46:25
tags:
    - CentOS
    - 软件
categories: 备忘录
---


## CentOS7安装Nginx

### 安装所需环境
1. gcc 安装
   安装 nginx 需要先将官网下载的源码进行编译，编译依赖 gcc 环境，如果没有 gcc 环境，则需要安装：
   ``` bash
   yum install gcc-c++
   ```
2. PCRE pcre-devel 安装
   PCRE(Perl Compatible Regular Expressions) 是一个Perl库，包括 perl 兼容的正则表达式库。nginx 的 http 模块使用 pcre 来解析正则表达式，所以需要在 linux 上安装 pcre 库，pcre-devel 是使用 pcre 开发的一个二次开发库。nginx也需要此库。命令：
   ``` bash
   yum install -y pcre pcre-devel
   ``` 
3. zlib 安装
   zlib 库提供了很多种压缩和解压缩的方式， nginx 使用 zlib 对 http 包的内容进行 gzip ，所以需要在 Centos 上安装 zlib 库。
   ``` bash
   yum install -y zlib zlib-devel
   ``` 
4. OpenSSL 安装
   OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及 SSL 协议，并提供丰富的应用程序供测试或其它目的使用。
   nginx 不仅支持 http 协议，还支持 https（即在ssl协议上传输http），所以需要在 Centos 安装 OpenSSL 库。
   ``` bash
   yum install -y openssl openssl-devel
   ``` 

### 官方下载与解压

1. 直接下载.tar.gz安装包，地址：[https://nginx.org/en/download.html](https://nginx.org/en/download.html)
   ![nginx下载](http://tech.jasonsoso.com/images/202204/ng-1.png "nginx下载")

2. 使用wget命令下载（推荐）。确保系统已经安装了wget，如果没有安装，执行 yum install wget 安装。
   ``` bash
   wget -c https://nginx.org/download/nginx-1.20.2.tar.gz
   ``` 
   我下载的是nginx-1.20.2版本，这个是目前的稳定版。

3. 解压.tar.gz
   ``` bash
   tar -zxvf nginx-1.20.2.tar.gz
   cd nginx-1.20.2/
   ``` 
   ![nginx相关目录](http://tech.jasonsoso.com/images/202204/ng-2.png "nginx相关目录")

### 配置与安装

1. 使用默认配置
   ``` bash
   ./configure
   ``` 
   
2. 编译安装
   ``` bash
   make
   make install
   ``` 

3. 查找安装路径
   ``` bash
   whereis nginx
   ``` 

### Nginx相关常用命令

![nginx相关命令](http://tech.jasonsoso.com/images/202204/ng-3.png "nginx相关命令")

1. 启动停止Nginx
   ``` bash
   cd /usr/local/nginx/sbin/
   ./nginx 
   ./nginx -s stop
   ./nginx -s quit
   ./nginx -s reload
   
   ``` 

2. 修改文件不需要重启，需要检查语法和reload即可
   ``` bash
   ./nginx -t
   ./nginx -s reload
   ``` 




## CentOS7上安装Node.js和npm