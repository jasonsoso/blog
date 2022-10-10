---
title: Canal：mysql数据同步到ElasticSearch
date: 2022/09/30 19:46:25
tags:
    - canal
    - ElasticSearch
    - ES
categories: 备忘录
---


## mysql配置


- 对于自建 MySQL , 需要先开启 Binlog 写入功能，配置 binlog-format 为 ROW 模式，my.cnf 中配置如下

```bash
[mysqld]
log-bin=mysql-bin   # 开启 binlog
binlog-format=ROW   # 选择 ROW 模式
server_id=1         # 配置 MySQL replaction 需要定义，不要和 canal 的 slaveId 重复
```

```bash
show variables like 'log_bin';        #判断MySQL是否已经开启binlog
show global variables like "binlog%"; #查看MySQL的binlog模式
```

我安装的mysql是 [mysql8.0.30版本](http://tech.jasonsoso.com/2022/09/centos-install-mysql/#centos7%E5%AE%89%E8%A3%85mysql8 "mysql8.0.30版本")，默认开启binlog，并且ROW模式。



- 授权 canal 链接 MySQL 账号具有作为 MySQL slave 的权限, 如果已有账户可直接 grant

```bash
    CREATE USER canal IDENTIFIED BY 'canal';  
    GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
    -- GRANT ALL PRIVILEGES ON *.* TO 'canal'@'%' ;
    FLUSH PRIVILEGES;
```

## 安装canal服务端


- 下载canal服务端，[release页面](https://github.com/alibaba/canal/releases "release页面")，下面以canal-1.1.5为例子

```bash
mkdir -p /usr/local/canal/canal-deployer
cd /usr/local/canal/canal-deployer
wget https://github.com/alibaba/canal/releases/download/canal-1.1.5/canal.deployer-1.1.5.tar.gz
```


- 解压

```bash
tar -zxvf canal.deployer-1.1.5.tar.gz
```


- 修改配置

```bash
vi conf/example/instance.properties
```
```bash
# position info 需要改成自己的数据库信息
canal.instance.master.address=10.0.12.4:3306

# username/password
canal.instance.dbUsername=canal
canal.instance.dbPassword=123456
canal.instance.connectionCharset = UTF-8
# enable druid Decrypt database password
canal.instance.enableDruid=false
```

- 启动并查看日志

```bash
# 启动
./bin/startup.sh
#查看日志
cat logs/canal/canal.log
```
![](http://tech.jasonsoso.com/images/202210/canal-1.png)
![](http://tech.jasonsoso.com/images/202210/canal-2.png)

- 关闭

```bash
 ./bin/stop.sh
```



## 安装ElasticSearch服务端
具体请移步到 [ElasticSearch部署实践](/2022/09/centos-install-es/ "ElasticSearch部署实践")




## 安装ClientAdapter客户端

- 下载ClientAdapter服务端，[release页面](https://github.com/alibaba/canal/releases "release页面")

```bash
mkdir -p /usr/local/canal/canal-adapter
cd /usr/local/canal/canal-adapter
wget https://github.com/alibaba/canal/releases/download/canal-1.1.5/canal.adapter-1.1.5.tar.gz
```


- 将canal.adapter进行解压

```bash
tar -zxvf canal.adapter-1.1.5.tar.gz
```

- 修改启动器配置application.yml

```bash
vim conf/application.yml
```
```yaml
  srcDataSources: #数据源
    defaultDS:
      url: jdbc:mysql://10.0.12.4:3306/test?useUnicode=true
      username: canal
      password: 123456
  canalAdapters: #适配器
  - instance: example # canal instance Name or mq topic name
    groups:
    - groupId: g1
      outerAdapters:
      - name: logger
      - 
        key: exampleKey
        name: es7  # or es6，adapter将会自动加载conf/es7下的所有.yml结尾的配置文件
        hosts: http://10.0.12.4:9200 # 127.0.0.1:9200 for rest mode
        properties:
          mode: rest # transport or rest
          cluster.name: my-application # es cluster name
```

- 适配器表映射文件
  修改es7下面的mytest_user.yml

```yaml
dataSourceKey: defaultDS   # 源数据源的key, 对应上面配置的srcDataSources中的值
outerAdapterKey: exampleKey # 对应application.yml中es配置的key
destination: example       # cannal的instance或者MQ的topic
groupId: g1                # 对应MQ模式下的groupId, 只会同步对应groupId的数据
esMapping:
  _index: mytest_user   # es 的索引名称
  _id: _id              # es 的_id, 如果不配置该项必须配置下面的pk项_id则会由es自动分配
#  upsert: true
#  pk: id               # 如果不需要_id, 则需要指定一个属性为主键属性
  sql: "select a.id as _id, a.id,a.name, a.role_id, a.created_time from user a"
#  objFields:           # 数组或者对象属性, array:; 代表以;字段里面是以;分隔的
#    _labels: array:;   # array或者json对象
  etlCondition: "where a.created_time>={}" # etl 的条件参数
  commitBatch: 3000
```


- 启动并查看日志

```bash
# 启动
./bin/startup.sh
#查看日志
tail -f logs/adapter/adapter.log
```

![](http://tech.jasonsoso.com/images/202210/canal-3.png)


- 问题一
```
canal com.alibaba.druid.pool.DruidDataSource cannot be cast to com.alibaba.druid.pool.DruidDataSource
```
解决方案：https://juejin.cn/post/6970249370688028679

常见报错解决方案：https://juejin.cn/post/7092319578465763365




- 单表映射索引示例sql:

```sql
CREATE TABLE `user` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(255) CHARACTER SET utf8mb4 DEFAULT '' COMMENT '名称',
  `role_id` bigint(20) unsigned NOT NULL COMMENT '角色id',
  `created_time` datetime DEFAULT NULL COMMENT '创建时间',
  `revised_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8 COMMENT='用户信息表';

INSERT INTO `user` (`name`, `role_id`, `created_time`) VALUES ('Jason', '1', '2021-06-23 10:04:27');
INSERT INTO `user` (`name`, `role_id`, `created_time`) VALUES ('Tom', '1', '2021-06-23 10:04:27');
```
![](http://zims.zhidianlife.com/attachment/MD/2021/06/23/head-4.png)

该sql对应的es mapping示例:

```json
{
  "mappings": {
    "properties": {
      "id": {
        "type": "long"
      },
      "name": {
        "type": "text"
      },
      "role_id": {
        "type": "long"
      },
      "created_time": {
        "type": "date"
      }
    }
  }
}
```

![](http://tech.jasonsoso.com/images/202210/canal-4.png)

![](http://tech.jasonsoso.com/images/202210/canal-5.png)

![](http://tech.jasonsoso.com/images/202210/canal-6.png)

![](http://tech.jasonsoso.com/images/202210/canal-7.png)

![](http://tech.jasonsoso.com/images/202210/canal-8.png)

以上展示，已经同步成功，无论新增，修改，删除，都同步到es





- 多表映射(一对一, 多对一)索引示例sql:

新增适配器表映射文件
es7下面的mytest_user_detail.yml

```yaml
dataSourceKey: defaultDS
outerAdapterKey: exampleKey     # 对应application.yml中es配置的key
destination: example
groupId: g1
esMapping:
  _index: mytest_user_detail
  _id: _id
#  upsert: true
#  pk: id
  sql: "select a.id as _id, a.id,a.name, a.role_id, r.role_name,a.created_time
	,d.gender,d.birthday,d.phone
        from user a
        LEFT JOIN user_role r on r.id=a.role_id
        LEFT JOIN user_detail d on a.id=d.user_id"
#  objFields:
#    _labels: array:;
  etlCondition: "where a.created_time>={}"
  commitBatch: 3000

```

```sql
CREATE TABLE `user` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(255) CHARACTER SET utf8mb4 DEFAULT '' COMMENT '名称',
  `role_id` bigint(20) unsigned NOT NULL COMMENT '角色id',
  `created_time` datetime DEFAULT NULL COMMENT '创建时间',
  `revised_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8 COMMENT='用户信息表';

CREATE TABLE `user_detail` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `user_id` bigint(20) unsigned NOT NULL COMMENT '用户id',
  `gender` int(4) unsigned DEFAULT '0' COMMENT '性别（0-未知，1-男，2-女）',
  `birthday` date DEFAULT NULL COMMENT '生日',
  `phone` varchar(64) NOT NULL DEFAULT '' COMMENT '手机号',
  `created_time` datetime DEFAULT NULL COMMENT '创建时间',
  `revised_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8 COMMENT='用户详细表';

CREATE TABLE `user_role` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `role_name` varchar(64) NOT NULL DEFAULT '' COMMENT '角色名称',
  `created_time` datetime DEFAULT NULL COMMENT '创建时间',
  `revised_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8 COMMENT='用户角色表';


```

该sql对应的es mapping示例:

```json
{
  "mappings": {
    "properties": {
      "id": {
        "type": "long"
      },
      "name": {
        "type": "text"
      },
      "role_id": {
        "type": "long"
      },
      "role_name": {
        "type": "text"
      },
      "created_time": {
        "type": "date"
      },
      "gender": {
        "type": "long"
      },
      "birthday": {
        "type": "date"
      },
      "phone": {
        "type": "text"
      }
    }
  }
}
```


![](http://tech.jasonsoso.com/images/202210/canal-9.png)
![](http://tech.jasonsoso.com/images/202210/canal-10.png)

以上展示，多表关联已经同步成功，无论新增，修改，删除，都同步到es





- adapter管理REST接口
  可以完成全量同步/部分数据同步

采用rest模式进行同步到es，则修改conf/application.yml的mode为rest

```
vim conf/application.yml
```

```yaml
  srcDataSources: #数据源
    defaultDS:
      url: jdbc:mysql://10.0.12.4:3306/test?useUnicode=true
      username: canal
      password: 123456
  canalAdapters: #适配器
  - instance: example # canal instance Name or mq topic name
    groups:
    - groupId: g1
      outerAdapters:
      - name: logger
      - 
        key: exampleKey
        name: es7  # or es6，adapter将会自动加载conf/es7下的所有.yml结尾的配置文件
        hosts: http://10.0.12.4:9200 # 127.0.0.1:9200 for rest mode
        properties:
          mode: rest #transport or rest
          # security.auth: test:123456 #  only used for rest mode
          cluster.name: my-application # es cluster name
```


```bash
#查看相关库总数据
curl http://127.0.0.1:8081/count/es7/exampleKey/mytest_user.yml
curl http://127.0.0.1:8081/count/es7/exampleKey/mytest_user_detail.yml

#手动ETL进行全量同步
curl http://127.0.0.1:8081/etl/es7/exampleKey/mytest_user.yml -X POST
curl http://127.0.0.1:8081/etl/es7/exampleKey/mytest_user_detail.yml -X POST

```
![](http://tech.jasonsoso.com/images/202210/canal-11.png)