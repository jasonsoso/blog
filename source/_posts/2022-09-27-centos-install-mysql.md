---
title: CentOS7安装Mysql8
date: 2022/9/27 19:46:25
tags:
    - CentOS
    - 软件
    - Mysql
categories: 备忘录
---


## CentOS7安装mysql8

### 安装前清理工作

1. 清理原有的mysql数据库安装的mysql软件包和依赖包：
   ``` bash
   rpm -pa | grep mysql
   ```
   
   结果如下：
   ``` bash
    mysql80-community-release-el7-1.noarch
    mysql-community-server-8.0.11-1.el7.x86_64
    mysql-community-common-8.0.11-1.el7.x86_64
    mysql-community-libs-8.0.11-1.el7.x86_64
    mysql-community-client-8.0.11-1.el7.x86_64
    
    #使用以下命令依次删除上面的程序
    yum remove mysql-xxx-xxx-
   ```

2. 删除mysql的配置文件，卸载不会自动删除配置文件
    
    首先使用如下命令查找出所用的配置文件； 
    ``` bash
    find / -name mysql
    ```

    可能的显示结果如下：
    ``` bash
    /etc/logrotate.d/mysql
    /etc/selinux/targeted/active/modules/100/mysql
    /etc/selinux/targeted/tmp/modules/100/mysql
    /var/lib/mysql
    /var/lib/mysql/mysql
    /usr/bin/mysql
    /usr/lib64/mysql
    /usr/local/mysql
    ```
   
   根据需求使用以下命令 依次 对配置文件进行删除
    ``` bash
   rm -rf /var/lib/mysql
    ```


### Mysql8.0安装 (YUM方式)

1. 安装Mysql8.0 的yum资源库：
    ``` bash
   cd /usr/local/
    mkdir mysql
    cd mysql
    wget https://repo.mysql.com/mysql80-community-release-el7.rpm
    ```

2. yum repo文件并更新 yum 缓存
    ``` bash
    rpm -ivh mysql80-community-release-el7.rpm
    ```
    执行结果：
    会在/etc/yum.repos.d/目录下生成repo文件mysql-community.repo mysql-community-source.repo
   ![检查JDK是否安装成功](http://tech.jasonsoso.com/images/202210/mysql-2.png "检查JDK是否安装成功")

   更新 yum 命令
    ``` bash
    yum clean all
    yum makecache
    ```

3. 使用 yum安装mysql
   查看mysql yum仓库中mysql版本，使用如下命令
    ``` bash
    yum repolist all | grep mysql
    ```
   ![查看mysql yum仓库中mysql版本](http://tech.jasonsoso.com/images/202210/mysql-3.png "查看mysql yum仓库中mysql版本")

   或者可以编辑 mysql repo文件
    ``` bash
    vim /etc/yum.repos.d/mysql-community.repo
    cat /etc/yum.repos.d/mysql-community.repo 
    #将相应版本下的enabled改成 1 即可；
    ```


4. 安装mysql 命令如下
   ``` bash
    yum install mysql-community-server
    ```

5. 开启mysql 服务
   在开启前最最重要的一步，防止数据库运行后，产生数据库大小写敏感无法更改的问题！
   ``` bash
    #(使用repo安装的mysql，生成的文件为my.cnfreoNew,修改为my.cnf即可)
    vi /etc/my.cnf
    ```
   在[mysqlId]下增加配置
    ``` bash
    lower_case_table_names=1
    ```
   然后ESC退出，:wq退出并保存，然后在启动服务
   ``` bash
    systemctl start mysqld.service
    ```

6. 获取初始密码登录mysql
   mysql在安装后会创建一个root@locahost账户，并且把初始的密码放到了/var/log/mysqld.log文件中；
    ``` bash
    cat /var/log/mysqld.log | grep password
    ```

7. 使用初始密码登录mysql
    ``` bash
    mysql -u root -p  #会提示输入密码
    #修改初始密码：
    ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!'; #注意位数和种类至少大+写+小写+符号+数字
    ```

8. 将mysql 服务加入开机启动项，并启动mysql进程
   ``` bash
   systemctl enable mysqld.service
   systemctl start mysqld.service
   ```
9. 修改加密规则及密码
    ``` mysql
    select version(); #mysql版本
    show global variables like 'port'; #查看服务端口
    select host,user,authentication_string from mysql.user; #查看用户表
    ALTER USER 'root'@'%' IDENTIFIED BY '123456' PASSWORD EXPIRE NEVER; #修改加密规则
    ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456'; #修改密码
    FLUSH PRIVILEGES; #刷新数据
   ``` 
10. 重启mysql
    ``` mysql
    service mysqld start #启动服务
    service mysqld stop #停止服务
    service mysqld restart #重启服务
    ``` 






