# 在Docker容器中运行Elasticsearch, Kibana和Cerebro
## 课程Demo
### 在 docker 中运行 Elasticsearch
进入 7.x-docker-2-es-instance目录

```
#启动
docker-compose up

#停止容器
docker-compose down

#停止容器并且移除数据
docker-compose down -v

#一些docker 命令
docker ps
docker stop Name/ContainerId
docker start Name/ContainerId

#删除单个容器
$docker rm Name/ID
-f, –force=false; -l, –link=false Remove the specified link and not the underlying container; -v, –volumes=false Remove the volumes associated to the container

#删除所有容器
$docker rm `docker ps -a -q`  
停止、启动、杀死、重启一个容器
$docker stop Name/ID  
$docker start Name/ID  
$docker kill Name/ID  
$docker restart name/ID

```
## 相关阅读
- 安装docker  https://www.docker.com/products/docker-desktop
- 安装 docker-compose https://docs.docker.com/compose/install/
- 如何创建自己的Docker Image - https://www.elastic.co/cn/blog/how-to-make-a-dockerfile-for-elasticsearch
- 如何在为docker image安装 Elasticsearch 插件 - https://www.elastic.co/cn/blog/elasticsearch-docker-plugin-management
- 如何设置 Docker 网络 - https://www.elastic.co/cn/blog/docker-networking
- Cerebro 源码 https://github.com/lmenezes/cerebro
- 一个开源的 ELK（Elasticsearch + Logstash + Kibana） docker-compose 配置 https://github.com/deviantony/docker-elk
- Install Elasticsearch with Docker https://www.elastic.co/guide/en/elasticsearch/reference/7.2/docker.html
- 
- 

## 笔记
 [Install Elasticsearch with Docker | Elasticsearch Guide [7.12] https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#docker-prod-prerequisites

  安装插件

1.安装中文分词插件，

登录容器

docker exec -it es7_12_01  /bin/bash

执行

 elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.12.1/elasticsearch-analysis-ik-7.12.1.zip

重启容器



2.安装拼音插件，登录容器，执行

 elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-pinyin/releases/download/v7.12.1/elasticsearch-analysis-pinyin-7.12.1.zip



3.简繁体转换插件

https://github.com/medcl/elasticsearch-analysis-stconvert





