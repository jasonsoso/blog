---
title: 玩转Mysql8
date: 2022/9/27 19:46:25
tags:
    - CentOS
    - 软件
    - Mysql
categories: 备忘录
---


## mysql5.7和mysql8.0区别


### 背景
1. 以往的职业生涯当中，大多数都是使用mysql5.7版本，听闻mysql8.0性能不错，特意来玩玩！

2. 总体来说，各个业务表存储引擎为InnoDB的mysql 5.7在使用语法上和mysql 8.0差别不大，官方表示 MySQL 8 要比 MySQL 5.7 快 2 倍，还带来了大量的改进和更快的性能。

### 新增功能

1. 性能： 
MySQL 8.0 在以下方面带来了更好的性能：读/写工作负载、IO 密集型工作负载、以及高竞争(”hot spot”热点竞争问题)工作负载。

2. NoSQL：
MySQL 从 5.7 版本开始提供 NoSQL 存储功能，目前在 8.0 版本中这部分功能也得到了更大的改进。该项功能消除了对独立的 NoSQL 文档数据库的需求，而 MySQL 文档存储也为 schema-less 模式的 JSON 文档提供了多文档事务支持和完整的 ACID 合规性。

3. 窗口函数(Window Functions)：
从 MySQL 8.0 开始，新增了一个叫窗口函数的概念，它可以用来实现若干新的查询方式。窗口函数与 SUM()、COUNT() 这种集合函数类似，但它不会将多行查询结果合并为一行，而是将结果放回多行当中。即窗口函数不需要 GROUP BY。
   
4. 隐藏索引：
在 MySQL 8.0 中，索引可以被“隐藏”和“显示”。当对索引进行隐藏时，它不会被查询优化器所使用。我们可以使用这个特性用于性能调试，例如我们先隐藏一个索引，然后观察其对数据库的影响。如果数据库性能有所下降，说明这个索引是有用的，然后将其“恢复显示”即可；如果数据库性能看不出变化，说明这个索引是多余的，可以考虑删掉。

5. 降序索引：
MySQL 8.0 为索引提供按降序方式进行排序的支持，在这种索引中的值也会按降序的方式进行排序。

6. 通用表表达式(Common Table Expressions CTE)：
在复杂的查询中使用嵌入式表时，使用 CTE 使得查询语句更清晰。

7. UTF-8 编码：
从 MySQL 8 开始，使用 utf8mb4 作为 MySQL 的默认字符集。

8. JSON：
MySQL 8 大幅改进了对 JSON 的支持，添加了基于路径查询参数从 JSON 字段中抽取数据的 JSON_EXTRACT() 函数，以及用于将数据分别组合到 JSON 数组和对象中的 JSON_ARRAYAGG() 和 JSON_OBJECTAGG() 聚合函数。

9. 可靠性：
InnoDB 现在支持表 DDL 的原子性，也就是 InnoDB 表上的 DDL 也可以实现事务完整性，要么失败回滚，要么成功提交，不至于出现 DDL 时部分成功的问题，此外还支持 crash-safe 特性，元数据存储在单个事务数据字典中。

10. 高可用性(High Availability)：
InnoDB 集群为您的数据库提供集成的原生 HA 解决方案。

### 使用区别

1. 身份验证：
caching_sha2_password是MySQL 8.0中的默认身份验证插件，替换了mysql 5.7的mysql_native_password，身份验证安全性能提升。

2. 授权：
与帐户管理相关的授权语法略有差异： MySQL5.7创建用户和用户授权命令可以同时执行
MySQL8.0创建用户和用户授权的命令需要分开执行
   ``` bash
   grant all privileges on . to 'canal'@'%' identified by '123456' 
   ```

4. 创建用户
   ``` bash
   create user 'canal'@'%' identified by '123456';
   ```

5. 用户授权【给予所有权限】
   ``` bash
   grant all privileges on *.* to 'canal'@'%';
   ```

6. 语法层面
语法使用区别总体变化很小，但是部分因为索引排序的变化，可能mysql5.7和mysql8.0查询结果的顺序不一致。
json相关函数在mysql8.0有部分方法名有差异。如JSON_MERGE替换为JSON_MERGE_PRESERVE

7. UTF-8 编码：
从 MySQL 8 开始，使用 utf8mb4 作为 MySQL 的默认字符集。



   

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
   ![yum repo文件](http://tech.jasonsoso.com/images/202210/mysql-2.png "yum repo文件")

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
   ![mysql8的安装](http://tech.jasonsoso.com/images/202210/mysql-1.png "mysql8的安装")


5. 调整mysql相关配置

      ``` bash
    #(使用repo安装的mysql，生成的文件为my.cnfreoNew,修改为my.cnf即可)
    vi /etc/my.cnf
    ```
   
   在[mysqlId]下增加配置（数据库大小写敏感）
    ``` bash
    lower_case_table_names=1
    ```

   开启慢查询
    ``` bash
    slow_query_log = ON #慢查询开启状态
    slow_query_log_file = /usr/local/mysql/data/slow.log #慢查询日志存放的位置（这个目录需要MySQL的运行帐号的可写权限，一般设置为MySQL的数据存放目录）
    long_query_time = 1 #查询超过多少秒才记录
    ```

   然后ESC退出，:wq退出并保存，然后在启动服务
   ``` bash
    systemctl start mysqld.service
    ```
   
   或者
   ``` mysql
   service mysqld start #启动服务
   service mysqld stop #停止服务
   service mysqld restart #重启服务
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
   
    caching_sha2_password是MySQL 8.0中的默认身份验证插件，替换了mysql 5.7的mysql_native_password，身份验证安全性能提升。
     ![身份验证](http://tech.jasonsoso.com/images/202210/mysql-4.png "身份验证")
    修改以上问题可执行以下命令：
   
    ``` mysql
    select version(); #mysql版本
    show global variables like 'port'; #查看服务端口
    select host,user,authentication_string from mysql.user; #查看用户表
    ALTER USER 'root'@'%' IDENTIFIED BY '123456' PASSWORD EXPIRE NEVER; #修改加密规则
    ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456'; #修改密码
    FLUSH PRIVILEGES; #刷新权限数据
    ``` 
10. 重启mysql
    ``` mysql
    service mysqld start #启动服务
    service mysqld stop #停止服务
    service mysqld restart #重启服务
    ``` 

## 常用的一些sql集锦

1. 进入mysql常用管理命令
   ``` mysql
   mysql -u root -p
   SHOW DATABASES;
   USE <数据库名>;
   SHOW TABLES;
    ``` 

2. 创建用户并授权
   ``` mysql
   create user 'canal'@'%' identified by '123456'; #创建用户
   grant all privileges on *.* to 'canal'@'%'; #授权
   ALTER USER 'canal'@'%' IDENTIFIED BY '123456' PASSWORD EXPIRE NEVER; #修改加密规则
   ALTER USER 'canal'@'%' IDENTIFIED WITH mysql_native_password BY '123456'; #修改密码
    ``` 

   ``` mysql
   其中有报异常：ERROR 1227 (42000): Access denied; you need (at least one of) the SYSTEM_USER privilege(s) for this
   查阅了一下官方文档，原因是由于root用户没有SYSTEM_USER权限，把权限加入后即可解决：
   grant system_user on *.* to 'root';
   ```
   
3. 操作库
   ``` mysql
   -- 创建库
   create database db1;
   -- 创建库是否存在，不存在则创建
   create database if not exists db1;
   -- 查看所有数据库
   show databases;
   -- 查看某个数据库的定义信息
   show create database db1;
   -- 修改数据库字符信息
   alter database db1 character set utf8;
   -- 删除数据库
   drop database db1;
   ```
4. 操作表
   ``` mysql
   --创建表
   create table student(
   id int,
   name varchar(32),
   age int ,
   score double(4,1),
   birthday date,
   insert_time timestamp
   );
   
   -- 查看表结构
   desc 表名;
   -- 查看创建表的SQL语句
   show create table 表名;
   -- 修改表名
   alter table 表名 rename to 新的表名;
   -- 添加一列
   alter table 表名 add 列名 数据类型;
   -- 删除列
   alter table 表名 drop 列名;
   -- 删除表
   drop table 表名;
   drop table  if exists 表名 ;
   ```
5. 增删改
   ``` mysql
   -- 写全所有列名
   insert into 表名(列名1,列名2,...列名n) values(值1,值2,...值n);
   -- 不写列名（所有列全部添加）
   insert into 表名 values(值1,值2,...值n);
   -- 插入部分数据
   insert into 表名(列名1,列名2) values(值1,值2);
   ```
   
   ``` mysql
   -- 删除表中数据
   delete from 表名 where 列名  = 值;
   -- 删除表中所有数据
   delete from 表名;
   -- 删除表中所有数据（高效 先删除表，然后再创建一张一样的表。）
   truncate table 表名;
   ```

   ``` mysql
   -- 不带条件的修改(会修改所有行)
   update 表名 set 列名 = 值;
   -- 带条件的修改
   update 表名 set 列名 = 值 where 列名=值;
   ```


6. 多表关联update
   ``` mysql
   # 通过 INNER JOIN
   UPDATE test_a a
   INNER JOIN test_b b ON a.user_id = b.user_id
   SET b.dept_id = a.dept_id;
   
   #通过 LEFT JOIN
   UPDATE test_a a
   LEFT JOIN test_b b ON a.user_id = b.user_id
   SET b.dept_id = a.dept_id;
   
   #通过子查询
   UPDATE test_b b 
   SET dept_id = ( SELECT dept_id FROM test_a WHERE user_id = b.user_id );

    ``` 

7. 给某表的字段增加1s,方便触发表数据修改
   ``` mysql
   #加1秒
   UPDATE promotion_product set CREATE_TIME=date_add(CREATE_TIME, interval 1 second)
   WHERE  PROMOTION_ID in (
   '020512b9a0df49a48d05f92c097e2223',
   '4a8053ee194d4020ab56f904863fdd0f'
   ); 
   ```
