# 在Docker容器中运行Elasticsearch, Kibana和Cerebro
## 课程Demo
### 在 docker 中运行 Elasticsearch
进入 7.x-docker-2-es-instance目录

```
#启动
docker-compose up

#停止容器
docke-compose stop

#停止容器：命令将停止您的容器，但也会删除已停止的容器以及所有已创建的网络
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



## 笔记
docker 文档资源 https://support.websoft9.com/docs/docker/zh/

 [Install Elasticsearch with Docker | Elasticsearch Guide [7.12] https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#docker-prod-prerequisites

https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-getting-started-initialization.html

Java High Level REST Client

https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high.html

中文文档

https://learnku.com/docs/elasticsearch73/7.3

https://www.kancloud.cn/yiyanan/elasticsearch_7_6

https://zq99299.github.io/note-book/elasticsearch-senior/



  安装插件

1.安装中文分词插件

https://github.com/medcl/elasticsearch-analysis-ik

登录容器

docker exec -it es7_12_01  /bin/bash

执行

 elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.12.1/elasticsearch-analysis-ik-7.12.1.zip

重启容器



2.安装拼音插件

https://github.com/medcl/elasticsearch-analysis-pinyin/tree/master

登录容器

执行

 elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-pinyin/releases/download/v7.12.1/elasticsearch-analysis-pinyin-7.12.1.zip



3.简繁体转换插件

https://github.com/medcl/elasticsearch-analysis-stconvert





```

#查看插件列表
GET _cat/plugins
#默认英文分词，会将中文拆分为单个
GET /_analyze
{
    "text": "我爱祖国"
}
#中文分词
GET /_analyze
{
    "analyzer": "ik_smart",
    "text": "北京印千山拍卖有限公司"
}
GET /_analyze
{
    "analyzer": "ik_max_word",
    "text": "北京印千山拍卖有限公司"
}

#pinyin
POST /_analyze
{
  "analyzer": "pinyin",
  "text":"新型冠状病毒"
}
PUT /medcl/ 
{
    "settings" : {
        "analysis" : {
            "analyzer" : {
                "pinyin_analyzer" : {
                    "tokenizer" : "my_pinyin"
                    }
            },
            "tokenizer" : {
                "my_pinyin" : {
                    "type" : "pinyin",
                    "keep_separate_first_letter" : false,
                    "keep_full_pinyin" : true,
                    "keep_original" : true,
                    "limit_first_letter_length" : 16,
                    "lowercase" : true,
                    "remove_duplicated_term" : true
                }
            }
        }
    }
}
GET /medcl/_analyze
{
    "analyzer": "pinyin_analyzer",
    "text": "北京印千山拍卖有限公司"
}
GET /medcl/_analyze
{
    "analyzer": "pinyin_analyzer",
    "text": "北京印千山拍卖有限公司"
}
POST /medcl/_mapping 
{
        "properties": {
            "name": {
                "type": "keyword",
                "fields": {
                    "pinyin": {
                        "type": "text",
                        "store": false,
                        "term_vector": "with_offsets",
                        "analyzer": "pinyin_analyzer"
                    }
                }
            }
        }
    
}

POST /medcl/_create/andy
{"name":"刘德华"}

GET /medcl/_search
{   
  "query": {
    "query_string": {
      "query": "ldh",
      "default_field": "name.pinyin"
    }
  }
}


```



# org 索引测试

```
PUT /org
{
  "settings": {
     "number_of_replicas": 1,
     "number_of_shards": 1,
     "analysis": {
       "analyzer": {
         "pinyin_analyzer" :{
           "tokenizer":"my_pinyin"
         },
         "user_name_analyzer" : {
                    "tokenizer" : "whitespace",
                    "filter" : "pinyin_first_letter_and_full_pinyin_filter"
         }
       },
       "filter": {
         "pinyin_first_letter_and_full_pinyin_filter" : {
                    "type" : "pinyin",
                    "keep_first_letter" : true,
                    "keep_full_pinyin" : false,
                    "keep_none_chinese" : true,
                    "keep_original" : false,
                    "limit_first_letter_length" : 16,
                    "lowercase" : true,
                    "trim_whitespace" : true,
                    "keep_none_chinese_in_first_letter" : true
         }
       }, 
       "tokenizer": {
         "my_pinyin":{
            "type": "pinyin",
            "keep_separate_first_letter" : false,
            "keep_full_pinyin" : true,
            "keep_original" : true,
            "limit_first_letter_length" : 16,
            "lowercase" : true,
            "remove_duplicated_term" : true
         }
       }
     }
  },
  "mappings": {
    "properties": {
        "name": {
          "type": "keyword",
          "fields": {
            "pinyin":{
               "type": "text",
                "store": false,
                "term_vector": "with_offsets",
                "analyzer": "pinyin_analyzer"
              
            },
            "ik":{
              "type": "text",
              "analyzer": "ik_max_word",
              "search_analyzer": "ik_smart"
            }
          }
        },
        "full_name": {
          "type": "keyword",
          "fields": {
            "pinyin":{
               "type": "text",
                "store": false,
                "term_vector": "with_offsets",
                "analyzer": "pinyin_analyzer"
             
            },
            "ik":{
              "type": "text",
              "analyzer": "ik_max_word",
              "search_analyzer": "ik_smart"
            }
          }
        }
    }
  },
  "aliases": {
    "dev_org": {}
  }
}
```







