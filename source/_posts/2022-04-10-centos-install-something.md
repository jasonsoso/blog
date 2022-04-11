---
title: CentOS7安装系统软件的备忘录
date: 2022/4/10 19:46:25
tags:
    - CentOS
    - 软件
    - Nginx
    - Git
    - Node.js
    - Ruby
categories: 备忘录
---


## CentOS7安装Nginx

### 安装所需环境

1. CentOS7更新升级
   ``` bash
   yum -y update
   ```
2. gcc 安装
   安装 nginx 需要先将官网下载的源码进行编译，编译依赖 gcc 环境，如果没有 gcc 环境，则需要安装：
   ``` bash
   yum install gcc-c++
   ```
3. PCRE pcre-devel 安装
   PCRE(Perl Compatible Regular Expressions) 是一个Perl库，包括 perl 兼容的正则表达式库。nginx 的 http 模块使用 pcre 来解析正则表达式，所以需要在 linux 上安装 pcre 库，pcre-devel 是使用 pcre 开发的一个二次开发库。nginx也需要此库。命令：
   ``` bash
   yum install -y pcre pcre-devel
   ``` 
4. zlib 安装
   zlib 库提供了很多种压缩和解压缩的方式， nginx 使用 zlib 对 http 包的内容进行 gzip ，所以需要在 Centos 上安装 zlib 库。
   ``` bash
   yum install -y zlib zlib-devel
   ``` 
5. OpenSSL 安装
   OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及 SSL 协议，并提供丰富的应用程序供测试或其它目的使用。
   nginx 不仅支持 http 协议，还支持 https（即在ssl协议上传输http），所以需要在 Centos 安装 OpenSSL 库。
   ``` bash
   yum install -y openssl openssl-devel
   ``` 

### 官方下载与解压

1. 直接下载.tar.gz安装包，地址：[https://nginx.org/en/download.html](https://nginx.org/en/download.html)
   ![nginx下载页面](http://tech.jasonsoso.com/images/202204/ng-1.png "nginx下载页面")

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


### 开机自动启动

1. 编辑/etc/rc.local文件，添加一行代码即可
   ``` bash
   vi /etc/rc.local
   # 增加一行 /usr/local/nginx/sbin/nginx
   ``` 

2. 设置执行权限：
   ``` bash
   chmod 755 rc.local
   ``` 



## CentOS7上安装Git客户端

### 安装Git客户端

1. 安装命令
   ``` bash
   yum install git
   ``` 

2. 查看版本等命令
   ``` bash
   git --version
   git --help
   ``` 
   ![Git相关命令](http://tech.jasonsoso.com/images/202204/git-1.png "Git相关命令")

3. 设置git的user name和email
   ``` bash
   git config --global user.name "jasonsoso"
   git config --global user.email  "648636045@qq.com"

   #查看配置
   git config --list
   ``` 
   ![Git相关配置](http://tech.jasonsoso.com/images/202204/git-2.png "Git相关配置")

### 通过SSH Key连接GitHub

1. 生成SSH Key
   ``` bash
   #查看是否有SSH Key
   cd ~/.ssh
   ll
   # 生成SSH Key
   ssh-keygen -t rsa -C "648636045@qq.com"
   ``` 
   ![生成的SSH Key](http://tech.jasonsoso.com/images/202204/git-3.png "生成的SSH Key")

2. GitHub添加SSH Key
   GitHub点击用户头像，选择setting
      ![github setting](http://tech.jasonsoso.com/images/202204/git-4.png "github setting")
   新增一个SSH Key
      ![新增一个SSH Key](http://tech.jasonsoso.com/images/202204/git-5.png "新增一个SSH Key")
   取个名字，把之前拷贝的秘钥id_rsa.pub复制进去

3. 测试验证

   ``` bash
      # 测试连接github是否成功
      ssh -T git@github.com
      # git clone 仓库
      git clone git@github.com:jasonsoso/blog.git
   ``` 
   ![验证是否能连接github](http://tech.jasonsoso.com/images/202204/git-6.png "验证是否能连接github")





## CentOS7上安装Ruby和Jekyll


### 安装Ruby

1. 安装命令
   ``` bash
   yum install ruby
   ``` 

2. 查看安装结果
   ``` bash
   ruby -v
   gem -v
   ``` 
   您可能注意到,这不是最新版本的Ruby。

### 升级为高版本的Ruby（RVM来管理）

1. 安装RVM来管理ruby
   ``` bash
   #导入key 
   gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
   #执行安装 (以下命令有时会提示报错误,我这里执行了几次才成功)
   curl -sSL https://get.rvm.io | bash -s stable
   source /etc/profile.d/rvm.sh
   rvm install ruby
   ``` 

   ``` bash
   #查看已有版本
   rvm list known
   #选择一个版本进行安装
   rvm install 2.4.1
   ``` 
   

2. 查看安装结果
   ``` bash
   ruby -v
   gem -v
   ``` 
   ![安装ruby](http://tech.jasonsoso.com/images/202204/ruby-1.png "安装ruby")



### 安装Jekyll

1. 安装命令
   ``` bash
   gem install jekyll
   
   gem install bundler
   ``` 

2. 查看安装结果
   ``` bash
   jekyll -v
   #jekyll 4.2.2
   ``` 
   后面就可以随便创建Jekyll静态网站和博客了: [https://www.jekyll.com.cn/docs/](https://www.jekyll.com.cn/docs/ "https://www.jekyll.com.cn/docs/")
   

## CentOS7上安装Node.js和npm

1. 安装命令
   ``` bash
   curl -fsSL https://rpm.nodesource.com/setup_16.x | sudo bash -
   yum install nodejs
   ``` 

2. 查看安装结果
   ``` bash
   node --version
   npm --version
   ``` 
   ![安装nodejs](http://tech.jasonsoso.com/images/202204/nodejs-1.png "安装nodejs")

3. hexo搞起
   ``` bash
   npm install -g hexo-cli
   npm install hexo
   hexo init blog
   ``` 
   后面就可以随便创建hexo快速、简洁且高效的博客框架了: [https://hexo.io/zh-cn/](https://hexo.io/zh-cn/ "https://hexo.io/zh-cn/")
   









