---
title: 玩转Elasticsearch
date: 2022/9/28 19:46:25
tags:
    - CentOS
    - 软件
    - Elasticsearch
    - es
categories: 备忘录
---
## 玩转Elasticsearch的背景
早几年前玩过[Elasticsearch2.x](http://tech.jasonsoso.com/2015/12/elasticsearch/)，
针对新版本的ES了解甚少，程序员本应不断追求新技术，所以来玩玩ES7.x，当然ES8.x今年也新出了，但是生态还是7.x好。

## 安装ElasticSearch服务端


- 安装ElasticSearch服务端，[ElasticSearch下载页](https://www.elastic.co/cn/downloads/elasticsearch "ElasticSearch下载页")
    ```bash
    cd /usr/local/
    mkdir es
    cd es/
    wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.17.6-linux-x86_64.tar.gz
    ```

- 解压并启动搞起

    ```bash
    tar -zxvf elasticsearch-7.17.6-linux-x86_64.tar.gz
    cd elasticsearch-7.17.6/
    ./bin/elasticsearch                #启动成功
    ./bin/elasticsearch -d             #后台启动
    ```


- 不能用root身份进行启动
    ```bash
    useradd es                         #创建es用户，并设置密码
    passwd es  
    chown -R es:es /usr/local/es/*     #授权
    su es                              #切换成es用户
    ./bin/elasticsearch                #启动成功
    ./bin/elasticsearch -d             #后台启动
    ```
  
  ![不能用root身份进行启动](http://tech.jasonsoso.com/images/202210/es-2.png "不能用root身份进行启动")

  ![此截图代表启动成功](http://tech.jasonsoso.com/images/202210/es-4.png "此截图代表启动成功")



- 问题
  ![此截图代表启动时候遇到的错误](http://tech.jasonsoso.com/images/202210/es-5.png "此截图代表启动时候遇到的错误")

    ```bash
    #针对上面问题可参考以下配置
    vi config/elasticsearch.yml
    ```
    
   ``` bash
    cluster.name: my-application
    node.name: node-1
    network.host: 10.0.12.4
    
    http.port: 9200
    transport.profiles.default.port: 9300
    cluster.initial_master_nodes: ["node-1"]
   ```
  终于启动成功：
  ![内网查看启动结果](http://tech.jasonsoso.com/images/202210/es-6.png "内网查看启动结果")
  ![外网查看启动结果](http://tech.jasonsoso.com/images/202210/es-7.png "外网查看启动结果")



## 安装elasticsearch-head客户端


- 安装NodeJs环境

```bash
$ curl --silent --location https://rpm.nodesource.com/setup_10.x | sudo bash -
$ sudo yum -y install nodejs       #会将npm一起安装的
$ node -v                          #查看node版本信息
$ npm -v                           #查看npm版本信息
```

- 安装elasticsearch-head插件，

具体访问[github地址](https://github.com/mobz/elasticsearch-head "github地址")：https://github.com/mobz/elasticsearch-head

```bash
$ git clone git://github.com/mobz/elasticsearch-head.git
$ cd elasticsearch-head
$ npm install          #有时候不通过，自行百度解决nodejs环境问题
$ npm run start
open http://localhost:9100/
```

- 配置head插件
  修改Gruntfile.js配置，增加hostname: '*'配置
```bash
$ vi Gruntfile.js

                connect: {
                        server: {
                                options: {
                                        port: 9100,
                                        base: '.',
                                        keepalive: true,
                                        hostname: '*'
                                }
                        }
                }
```

修改head/_site/app.js文件
```bash
$ vi app.js
this.base_uri = this.config.base_uri || this.prefs.get("app-base_uri") || "http://192.168.199.93:9200";
```

ES 配置 修改elasticsearch.yml ,增加跨域的配置（需要重启es才能生效）
```bash
$ vi config/elasticsearch.yml
http.cors.enabled: true
http.cors.allow-origin: "*"
```

![](http://zims.zhidianlife.com/attachment/MD/2021/06/22/head-1.png)
以上代表head插件已经成功安装






## 安装kibana客户端


- 下载kibana客户端，[kibana下载页](https://www.elastic.co/cn/downloads/kibana "Download Kibana")，下面以最新版本kibana-7.13.2为例子

```bash
$ wget https://artifacts.elastic.co/downloads/kibana/kibana-7.13.2-linux-x86_64.tar.gz
```

- 解压并启动搞起

```bash
$ tar -zxvf kibana-7.13.2-linux-x86_64.tar.gz
$ cd kibana-7.13.2-linux-x86_64
$ vim kibana.yml                                               #配置elasticsearch.hosts: ["http://192.168.199.93:9200"] 和 server.host: "192.168.199.93"
$ ./bin/kibana                                                 #切换成zhidian用户则启动成功
$ nohup /usr/local/es/kibana-7.13.2-linux-x86_64/bin/kibana &  #后台运行
```
访问 http://192.168.199.93:5601/ 即可
![](http://zims.zhidianlife.com/attachment/MD/2021/06/28/kibana-1.png)







## 安装cerebro监控工具


- 下载cerebro监控工具，[cerebro下载页](https://github.com/lmenezes/cerebro/releases "Download Kibana")，下面以最新版本v0.9.4

```bash
$ wget https://github.com/lmenezes/cerebro/releases/download/v0.9.4/cerebro-0.9.4.tgz
```

- 解压并启动搞起

```bash
$ tar xf cerebro-0.9.4.tgz
$ cd cerebro-0.9.4
$ vim conf/application.conf     #配置host
$ ./bin/cerebro
```
访问 http://192.168.199.93:9000/ 即可
![](http://zims.zhidianlife.com/attachment/MD/2021/06/29/c-1.png)




## 安装其他常用插件

插件是用来增强 Elasticsearch 功能的方法，分为 核心插件（官方） & 社区插件。
参考文章[《Elasticsearch6.x和7.x版本常用插件汇总》](https://blog.csdn.net/weixin_30314813/article/details/101858621 "《Elasticsearch6.x和7.x版本常用插件汇总》")

安装 analysis-icu ICU 分析插件

```bash
$ ./bin/elasticsearch-plugin install analysis-icu
```

安装 elasticsearch-analysis-ik 分词插件,具体请参考https://github.com/medcl/elasticsearch-analysis-ik

```bash
$ ./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.13.2/elasticsearch-analysis-ik-7.13.2.zip
```

查看已安装的插件
```bash
$ ./bin/elasticsearch-plugin list
#或者浏览器访问：http://192.168.199.93:9200/_cat/plugins
```






