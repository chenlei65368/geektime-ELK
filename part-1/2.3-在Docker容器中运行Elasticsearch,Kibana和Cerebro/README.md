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
- windows vm.max_map_count设置
打开Window 10 的CMD
执行以下命令：
wsl -d docker-desktop
echo 262144 >> /proc/sys/vm/max_map_count
复制代码
通过这个方法，即使操作系统重启，参数仍然有效。


## 笔记
docker 文档资源 https://support.websoft9.com/docs/docker/zh/

 [Install Elasticsearch with Docker | Elasticsearch Guide [7.12] https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#docker-prod-prerequisites

https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-getting-started-initialization.html

Java High Level REST Client

https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high.html

## 中文文档

https://learnku.com/docs/elasticsearch73/7.3

https://www.kancloud.cn/yiyanan/elasticsearch_7_6

https://zq99299.github.io/note-book/elasticsearch-senior/

父子关系

https://esdoc.bbossgroups.com/#/elasticsearch6-parent-child



## 查询客户端参考

https://esdoc.bbossgroups.com/#/quickstart



logstach 中文文档

https://doc.yonyoucloud.com/doc/logstash-best-practice-cn/index.html

## 插件测试

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

# 添加错误索引
PUT test     
# 正确索引
PUT test_2   
#通过别名修改正确索引为test
POST /_aliases
{
  "actions" : [
    { "add":  { "index": "test_2", "alias": "test" } },
    { "remove_index": { "index": "test" } }  
  ]
}
#查看
GET test
GET /_cat/indices?v

#克隆索引==============================
#1.首先设置源索引为只读
PUT /movies/_settings
{
  "settings": {
    "index.blocks.write": true
  }
}
#2.克隆索引
POST /movies/_clone/my-movies
#3.恢复源索为可写
PUT /movies/_settings
{
  "settings": {
    "index.blocks.write": false
  }
}
#查看新索引
GET my-movies



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
GET /org
DELETE /org

PUT /dev_org_v1
{
  "settings": {
     "number_of_replicas": 1,
     "number_of_shards": 3,
     "analysis": {
       "analyzer": {
         "pinyin_analyzer" :{
           "tokenizer":"my_pinyin"
         },
         "ngramIndexAnalyzer": {
                    "type": "custom",
                    "tokenizer": "keyword",
                    "filter": ["edge_ngram_filter","lowercase"],
                    "char_filter" : ["html_strip","charconvert","tsconvert"]
         },
         "ngramSearchAnalyzer": {
                    "type": "custom",
                    "tokenizer": "keyword",
                    "filter":["lowercase"],
                    "char_filter" : ["html_strip","charconvert","tsconvert"]
         }, 
         "ikIndexAnalyzer": {
                    "type": "custom",
                    "tokenizer": "ik_max_word",                   
                    "char_filter" : ["html_strip","charconvert","tsconvert"]
         },
         "ikSearchAnalyzer": {
                    "type": "custom",
                    "tokenizer": "ik_smart",                       
                    "char_filter" : ["html_strip","charconvert","tsconvert"]
         },
         "pinyiSimpleIndexAnalyzer":{                   
                    "tokenizer" : "keyword",
                    "filter": ["pinyin_simple_filter","edge_ngram_filter","lowercase"],
                    "char_filter" : ["html_strip","charconvert"]
         },                
         "pinyiSimpleSearchAnalyzer":{
                    "tokenizer" : "keyword",     
                    "filter": ["my_pinyin_filter","lowercase"],
                    "char_filter" : ["html_strip","charconvert"]
         },
         "pinyiFullIndexAnalyzer":{                   
                    "tokenizer" : "keyword",
                    "filter": ["pinyin_full_filter","lowercase"],
                    "char_filter" : ["html_strip","charconvert"]
         },                
         "pinyiFullSearchAnalyzer":{
                    "tokenizer" : "keyword",     
                    "filter": ["pinyin_full_filter","lowercase"],
                    "char_filter" : ["html_strip","charconvert"]
         },
         "user_name_analyzer" : {
                    "tokenizer" : "whitespace",
                    "filter" : "pinyin_first_letter_and_full_pinyin_filter",
                    "char_filter" : ["html_strip","charconvert"]
         }
       },
       "filter": {
         "edge_ngram_filter": { 
                    "type":     "edge_ngram",
                    "min_gram": 1,
                    "max_gram": 50                    
         }, 
         "pinyin_simple_filter":{
           "type" : "pinyin",
                    "keep_first_letter":true,
                    "keep_separate_first_letter" : false,
                    "keep_full_pinyin" : false,
                    "keep_original" : false,
                    "limit_first_letter_length" : 50,
                    "lowercase" : true
         },
         "pinyin_full_filter":{
                    "type" : "pinyin",
                    "keep_first_letter":false,
                    "keep_separate_first_letter" : false,
                    "keep_full_pinyin" : true,                        
                    "none_chinese_pinyin_tokenize":true,
                    "keep_original" : false,
                    "limit_first_letter_length" : 50,
                    "lowercase" : true
         },
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
         },
         "my_pinyin_filter" : {
            "type": "pinyin",
            "keep_first_letter" : true,
            "keep_separate_first_letter" : true,
            "keep_full_pinyin" : true,
            "keep_original" : false,
            "limit_first_letter_length" : 16,
            "lowercase" : true,
            "remove_duplicated_term" : false
         },
         "tsconvert" : {
                     "type" : "stconvert",
                     "delimiter" : "#",
                     "keep_both" : false,
                     "convert_type" : "t2s"
                 }
         
       }, 
       "tokenizer": {
         "my_pinyin":{
            "type": "pinyin",
            "keep_first_letter" : true,
            "keep_separate_first_letter" : true,
            "keep_full_pinyin" : true,
            "keep_original" : false,
            "limit_first_letter_length" : 16,
            "lowercase" : true,
            "remove_duplicated_term" : false
         },
         "tsconvert" : {
                    "type" : "stconvert",
                    "delimiter" : "#",
                    "keep_both" : false,
                    "convert_type" : "t2s"
                }
         
       },
      
        "char_filter" : {
                "tsconvert" : {
                    "type" : "stconvert",
                    "convert_type" : "t2s"
                },
                "charconvert" : {
                  "type" : "mapping",
                   "mappings": [
                                "\\n=>",
                                "?=>",
                                "\\r=>",
                                "@ =>",
                                "/=> ",
                                "(=> ",
                                ")=> ",
                                "（=> ",
                                "）=> ",
                                "à=>a",
                                "á=>a",
                                "â=>a",
                                "ä=>a",
                                "À=>a",
                                "Â=>a",
                                "Ä=>a",
                                "è=>e",
                                "é=>e",
                                "ê=>e",
                                "ë=>e",
                                "È=>e",
                                "É=>e",
                                "Ê=>e",
                                "Ë=>e",
                                "î=>i",
                                "ï=>i",
                                "Î=>i",
                                "Ï=>i",
                                "ô=>o",
                                "ö=>o",
                                "Ô=>o",
                                "Ö=>o",
                                "ù=>u",
                                "û=>u",
                                "ü=>u",
                                "Ù=>u",
                                "Û=>u",
                                "Ü=>u",
                                "ç=>c",
                                "œ=>c",
                                "&=>",
                                "^=>",
                                ".=>",
                                "·=>",
                                "-=>",
                                "'=>",
                                "’=>",
                                "‘=>"
                              ]
                }
            }
     }
  },
  "mappings": {
    "dynamic": false, 
    "properties": {
        "name": {
          "type": "keyword",
          "fields": {
            "words":{
               "type": "text",
                 "store": false,
                 "term_vector": "with_offsets",
                 "analyzer": "ngramIndexAnalyzer",
                 "search_analyzer": "ngramSearchAnalyzer"
              
            },
            "pinyin":{
               "type": "text",
                "store": false,
                "term_vector": "with_offsets",
                "analyzer": "pinyin_analyzer"
              
            },
            "s_pinyin":{
               "type": "text",
                "store": false,
                "term_vector": "with_offsets",
                "analyzer": "pinyiSimpleIndexAnalyzer",
                "search_analyzer": "pinyiSimpleSearchAnalyzer"
              
            },
            "f_pinyin":{
               "type": "text",
                "store": false,
                "term_vector": "with_offsets",
                "analyzer": "pinyiFullIndexAnalyzer",
                "search_analyzer": "pinyiFullSearchAnalyzer"
              
            },
            "ik":{
              "type": "text",
              "store": false,
              "term_vector": "with_positions_offsets",
              "analyzer": "ikIndexAnalyzer",
              "search_analyzer": "ikSearchAnalyzer"
            }
          }
        },
        "full_name": {
          "type": "keyword",
          "fields": {
            "words":{
               "type": "text",
                 "store": false,
                 "term_vector": "with_offsets",
                 "analyzer": "ngramIndexAnalyzer",
                 "search_analyzer": "ngramSearchAnalyzer"
              
            },
            "pinyin":{
               "type": "text",
                "store": false,
                "term_vector": "with_offsets",
                "analyzer": "pinyin_analyzer"
              
            },
            "s_pinyin":{
               "type": "text",
                "store": false,
                "term_vector": "with_offsets",
                "analyzer": "pinyiSimpleIndexAnalyzer",
                "search_analyzer": "pinyiSimpleSearchAnalyzer"
              
            },
            "f_pinyin":{
               "type": "text",
                "store": false,
                "term_vector": "with_offsets",
                "analyzer": "pinyiFullIndexAnalyzer",
                "search_analyzer": "pinyiFullSearchAnalyzer"
              
            },
            "ik":{
              "type": "text",
              "store": false,
              "term_vector": "with_positions_offsets",
              "analyzer": "ikIndexAnalyzer",
              "search_analyzer": "ikSearchAnalyzer"
            }
          }
        },
        "area": {
          "type": "integer"
        },
        "code": {
          "type": "keyword"
        },
        "cooperateStatus": {
          "type": "long"
        },
        "address": {
          "type": "keyword",
          "fields": {
            "ik":{
              "type": "text",
              "store": false,
              "term_vector": "with_positions_offsets",
              "analyzer": "ikIndexAnalyzer",
              "search_analyzer": "ikSearchAnalyzer"
            }
          }
        }
         

    }
  }
}


PUT /org/_doc/1
{
  "name":"香港佳士得",
  "full_name":"佳士得香港有限公司"
}

PUT /org/_doc/2
{
  "name":"邦瀚斯",
  "full_name":"邦瀚斯拍卖行"
}

PUT /org/_doc/3
{
  "name":"东方艺都",
  "full_name":"东方艺都（北京）拍卖有限公司"
}

GET /org/_doc/3745

GET /org/_search?q=name.ik:（北京）
GET /org/_search?q=name.pinyin:dfyd
GET /org/_search?q=full_name.f_pinyin:dfyd
GET /org/_analyze
{
  "text": ["东方艺都（北京）拍卖有限公司"],
  "analyzer": "pinyiFullIndexAnalyzer"
}
GET /org/_analyze
{
  "text": ["df"],
  "analyzer": "pinyiSimpleSearchAnalyzer"
}
GET /org/_analyze
{
  "text": ["东方艺都（北京）拍卖有限公司"],
  "analyzer": "pinyiSimpleIndexAnalyzer"
}
GET /org/_analyze
{
  "text": ["东方艺都（北京）拍卖有限公司 佳士得香港有限公司"],
  "analyzer": "pinyin_analyzer"
}
GET /org/_search
{
 "query": {"term": {
   "name.ik": "东方"
 }}
}
GET /org/_search
{
 "query": {"match_phrase": {
   "full_name.ik": "东方"
 }}
}
GET /org/_search
{
 "query": {"match_phrase_prefix": {
   "full_name.ik": "东方"
 }}
}
GET /org/_search
{
 "query": {"match_phrase": {
   "full_name.pinyin": "dongfangyid"
 }}
}
GET /org/_search
{
 "query": {"match_phrase": {
   "full_name.f_pinyin": "dongfang"
 }}
}
GET /org/_search
{
 "query": {"match_phrase": {
   "full_name.s_pinyin": "dfyd"
 }}
}
GET /org/_search
{
 "query": {"match_phrase_prefix": {
   "full_name.f_pinyin": "dongfangyid"
 }}
}
GET /org/_search
{
 "query": {"term": {
   "full_name.s_pinyin": "dfyd"
 }}
}


GET /_analyze
{
  "tokenizer": "keyword",
  "char_filter": [
    "html_strip"
  ],
  "text": "<p>I&apos;m so <b>happy</b>!</p>"
}

GET /_analyze
{
  "tokenizer": "keyword",
  "char_filter": [
    {
      "type": "mapping",
      "mappings": [
        "٠ => 0",
        "١ => 1",
        "٢ => 2",
        "٣ => 3",
        "٤ => 4",
        "٥ => 5",
        "٦ => 6",
        "٧ => 7",
        "٨ => 8",
        "٩ => 9",
        "/=>",
         "\\n =>",
                                "? =>",
                                "\\r =>",
                                "（=> ",
                                "）=> "
      ]
    }
  ],
  "text": "东方艺都（北京）拍卖有限公司\r\n \n \r My /license plate is ٢٥٠١٥"
}

# 查看索引相关信息
GET kibana_sample_data_ecommerce

# 查看索引的文档总数
GET kibana_sample_data_ecommerce/_count

# 查看前10条文档，了解文档格式
POST kibana_sample_data_ecommerce/_search
{
}
# _cat indices API
# 查看indices
GET /_cat/indices/kibana*?v&s=index

# 查看状态为绿的索引
GET /_cat/indices?v&health=green

# 按照文档个数排序
GET /_cat/indices?v&s=docs.count:desc

# 查看具体的字段
GET /_cat/indices/kibana*?pri&v&h=health,index,pri,rep,docs.count,mt

# How much memory is used per index?
GET /_cat/indices?v&h=i,tm&s=tm:desc

# 别名操作 ===========================
# 增加别名，可以同时增加多个
POST /_aliases
{
  "actions" : [
    { "add" : { "index" : "kibana_sample_data_ecommerce", "alias" : "sample_data_ecommerce000" }},
    {"add": {"index": "kibana_sample_data_ecommerce","alias": "sample_data_ecommerce0001"}},
    {"remove": {"index": "kibana_sample_data_ecommerce","alias": "sample_data_ecommerce"}}
  ]
}
```



## logstash 导入mysql数据

全量导入

```
#全量导入
input {
         stdin {}
         jdbc {
               # mysql 数据库链接
               jdbc_connection_string => "jdbc:mysql:"
               # 用户名和密码
               jdbc_user => " "
               jdbc_password => " "
               # 驱动
               jdbc_driver_library => "E:/my_work_space/geektime/logstash-7.12.1/mysql_tongbu/mysql-connector-java-8.0.20.jar"
               # 驱动类名
               jdbc_driver_class => "com.mysql.cj.jdbc.Driver"
			   #分页
               jdbc_paging_enabled => "true"
               jdbc_page_size => "10"
			   
			   #直接执行sql语句
               statement => "select id,code,simple_name as name,name as full_name,address,area,cooperate_status as cooperateStatus,updated_at from auction_organization"
               # 执行的sql 文件路径+名称
               #statement_filepath => "E:/my_work_space/geektime/logstash-7.12.1/mysql_tongbu/org.sql"
               # 设置监听间隔  各字段含义（由左至右）分、时、天、月、年，全部为*默认含义为每分钟都更新
               schedule => "* * * * *"
			   # 每隔10分钟执行一次
			   #schedule => "*/10 * * * *"
			   # 是否记录上次执行结果, 如果为真,将会把上次执行到的 tracking_column 字段的值记录下来,保存到last_run_metadata_path
			   record_last_run => true
			   
			   # 指定最后运行时间文件存放的地址，记录最新的同步的offset信息
			   # 指定文件,来记录上次执行到的 tracking_column 字段的值
			   # 比如上次数据库有 10000 条记录,查询完后该文件中就会有数字 10000 这样的记录,下次执行 SQL 查询可以从 10001 条处开始.
               # 我们只需要在 SQL 语句中 WHERE MY_ID > :sql_last_value 即可. 其中 :sql_last_value 取得就是该文件中的值(10000)
			   last_run_metadata_path => "E:/my_work_space/geektime/logstash-7.12.1/mysql_tongbu/last_id.txt"
               
			   # 是否需要记录某个column的值。当该值设置成true时，系统会记录tracking_column参数所指定的列的最新的值，并在下一次管道执行时通过该列的值来判断需要更新的记录。
			   use_column_value => true
		       # 跟踪列的类型，numeric 表示数值类型, timestamp 表示时间戳类型
			   tracking_column_type => "numeric"
			   # 指定跟踪列，该列必须是递增的，一般是MySQL主键
			   tracking_column => "id"
			   #是否将 column 名称转小写
               #lowercase_column_names => false
               clean_run => false
               # 标识输入的类型，自定义字符串，可以用于输出类型选择
               type => "org"
			   #时区
			   jdbc_default_timezone => "Asia/Shanghai"
         }
   }
   filter {
   
		#mutate{
			#删除无效的字段
		#	remove_field => ["updated_at"]
		#}
		 
		# 因为时区问题需要修正时间
		#ruby { 
		#	code => "event.set('timestamp', event.get('@timestamp').time.localtime + 8*60*60)" 
		#}
		#ruby {
		#	code => "event.set('@timestamp',event.get('timestamp'))"
		#}
		#mutate {
		#	remove_field => ["timestamp"]
		#}
		 
		# 因为时区问题需要修正时间
		#ruby {
		#	code => "event.set('create_time', event.get('create_time').time.localtime + 8*60*60)" 
		#}    
		# 因为时区问题需要修正时间
		ruby {
			code => "event.set('updated_at', event.get('updated_at').time.localtime + 8*60*60)" 
		}    
		# 转换成日期格式
		#ruby {
		#	code => "event.set('start_date', event.timestamp.time.localtime.strftime('%Y-%m-%d'))"
		#}    
		# 转换成日期格式
		#ruby {
		#	code => "event.set('end_date', event.timestamp.time.localtime.strftime('%Y-%m-%d'))"
		#}    
		# 转换成日期格式
		#ruby {
		#	code => "event.set('sign_date', event.timestamp.time.localtime.strftime('%Y-%m-%d'))"
		#}    
	
   }

     output {
         if [type]=="org"{
             elasticsearch {
                 hosts => ["localhost:9200"]
				 index => "org"
                 document_type => "_doc"
                 # user => "elastic"
                 # password => "changeme"
                 document_id => "%{id}"
             }
         }
         stdout {
               codec => json_lines
        }
    }


```



增量导入

```
#增量导入
	input {
         stdin {}
         jdbc {
               # mysql 数据库链接
               jdbc_connection_string => "jdbc:mysql:"
               # 用户名和密码
               jdbc_user => ""
               jdbc_password => ""
               # 驱动
               jdbc_driver_library => "E:/my_work_space/geektime/logstash-7.12.1/mysql_tongbu/mysql-connector-java-8.0.20.jar"
               # 驱动类名
               jdbc_driver_class => "com.mysql.cj.jdbc.Driver"
			   #分页
               jdbc_paging_enabled => "true"
               jdbc_page_size => "2000"
			   
			   #直接执行sql语句
               #statement => "select id,code,simple_name as name,name as full_name,address,area,cooperate_status as cooperateStatus,updated_at from auction_organization where updated_at>= :sql_last_value"
               # 执行的sql 文件路径+名称
               statement_filepath => "E:/my_work_space/geektime/logstash-7.12.1/mysql_tongbu/org.sql"
               # 设置监听间隔  各字段含义（由左至右）分、时、天、月、年，全部为*默认含义为每分钟都更新
               schedule => "* * * * *"
			   # 每隔10分钟执行一次
			   #schedule => "*/10 * * * *"
			   # 是否记录上次执行结果, 如果为真,将会把上次执行到的 tracking_column 字段的值记录下来,保存到last_run_metadata_path
			   record_last_run => true
			   
			   # 指定最后运行时间文件存放的地址，记录最新的同步的offset信息
			   # 指定文件,来记录上次执行到的 tracking_column 字段的值
			   # 比如上次数据库有 10000 条记录,查询完后该文件中就会有数字 10000 这样的记录,下次执行 SQL 查询可以从 10001 条处开始.
               # 我们只需要在 SQL 语句中 WHERE MY_ID > :sql_last_value 即可. 其中 :sql_last_value 取得就是该文件中的值(10000)
			   last_run_metadata_path => "E:/my_work_space/geektime/logstash-7.12.1/mysql_tongbu/last_id.txt"
               
			   # 是否需要记录某个column的值。当该值设置成true时，系统会记录tracking_column参数所指定的列的最新的值，并在下一次管道执行时通过该列的值来判断需要更新的记录。
			   use_column_value => true
		       # 跟踪列的类型，numeric 表示数值类型, timestamp 表示时间戳类型
			   tracking_column_type => "timestamp"
			   # 指定跟踪列，该列必须是递增的，一般是MySQL主键
			   tracking_column => "updated_at"
			   #是否将 column 名称转小写
               #lowercase_column_names => false
               clean_run => false
               # 标识输入的类型，自定义字符串，可以用于输出类型选择
               type => "org"
			   #时区
			   jdbc_default_timezone => "Asia/Shanghai"
         }
	}
filter{
   
		 ruby { 
			code => "event.set('timestamp', event.get('@timestamp').time.localtime + 8*60*60)" 
		}
		ruby {
			code => "event.set('@timestamp',event.get('timestamp'))"
		}
		mutate {
			remove_field => ["timestamp"]
		}
}
output {
		if [type]=="org"{
            elasticsearch {
                 hosts => ["localhost:9200"]
				 index => "org"
                 document_type => "_doc"
                 # user => "elastic"
                 # password => "changeme"
                 document_id => "%{id}"
             }
         }
         stdout {
               codec => json_lines
        }
}

```



lot_item

```
PUT /dev_lotitem_v1
{
  "settings": {
     "number_of_replicas": 1,
     "number_of_shards": 3,
     "analysis": {
       "analyzer": {
         "pinyin_analyzer" :{
           "tokenizer":"my_pinyin"
         },
         "ngramIndexAnalyzer": {
                    "type": "custom",
                    "tokenizer": "keyword",
                    "filter": ["edge_ngram_filter","lowercase"],
                    "char_filter" : ["html_strip","charconvert","tsconvert"]
         },
         "ngramSearchAnalyzer": {
                    "type": "custom",
                    "tokenizer": "keyword",
                    "filter":["lowercase"],
                    "char_filter" : ["html_strip","charconvert","tsconvert"]
         }, 
         "ikIndexAnalyzer": {
                    "type": "custom",
                    "tokenizer": "ik_max_word",                   
                    "char_filter" : ["html_strip","charconvert","tsconvert"]
         },
         "ikSearchAnalyzer": {
                    "type": "custom",
                    "tokenizer": "ik_smart",                       
                    "char_filter" : ["html_strip","charconvert","tsconvert"]
         },
         "pinyiSimpleIndexAnalyzer":{                   
                    "tokenizer" : "keyword",
                    "filter": ["pinyin_simple_filter","edge_ngram_filter","lowercase"],
                    "char_filter" : ["html_strip","charconvert"]
         },                
         "pinyiSimpleSearchAnalyzer":{
                    "tokenizer" : "keyword",     
                    "filter": ["my_pinyin_filter","lowercase"],
                    "char_filter" : ["html_strip","charconvert"]
         },
         "pinyiFullIndexAnalyzer":{                   
                    "tokenizer" : "keyword",
                    "filter": ["pinyin_full_filter","lowercase"],
                    "char_filter" : ["html_strip","charconvert"]
         },                
         "pinyiFullSearchAnalyzer":{
                    "tokenizer" : "keyword",     
                    "filter": ["pinyin_full_filter","lowercase"],
                    "char_filter" : ["html_strip","charconvert"]
         },
         "user_name_analyzer" : {
                    "tokenizer" : "whitespace",
                    "filter" : "pinyin_first_letter_and_full_pinyin_filter",
                    "char_filter" : ["html_strip","charconvert"]
         }
       },
       "filter": {
         "edge_ngram_filter": { 
                    "type":     "edge_ngram",
                    "min_gram": 1,
                    "max_gram": 50                    
         }, 
         "pinyin_simple_filter":{
           "type" : "pinyin",
                    "keep_first_letter":true,
                    "keep_separate_first_letter" : false,
                    "keep_full_pinyin" : false,
                    "keep_original" : false,
                    "limit_first_letter_length" : 50,
                    "lowercase" : true
         },
         "pinyin_full_filter":{
                    "type" : "pinyin",
                    "keep_first_letter":false,
                    "keep_separate_first_letter" : false,
                    "keep_full_pinyin" : true,                        
                    "none_chinese_pinyin_tokenize":true,
                    "keep_original" : false,
                    "limit_first_letter_length" : 50,
                    "lowercase" : true
         },
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
         },
         "my_pinyin_filter" : {
            "type": "pinyin",
            "keep_first_letter" : true,
            "keep_separate_first_letter" : true,
            "keep_full_pinyin" : true,
            "keep_original" : false,
            "limit_first_letter_length" : 16,
            "lowercase" : true,
            "remove_duplicated_term" : false
         },
         "tsconvert" : {
                     "type" : "stconvert",
                     "delimiter" : "#",
                     "keep_both" : false,
                     "convert_type" : "t2s"
                 }
         
       }, 
       "tokenizer": {
         "my_pinyin":{
            "type": "pinyin",
            "keep_first_letter" : true,
            "keep_separate_first_letter" : true,
            "keep_full_pinyin" : true,
            "keep_original" : false,
            "limit_first_letter_length" : 16,
            "lowercase" : true,
            "remove_duplicated_term" : false
         },
         "tsconvert" : {
                    "type" : "stconvert",
                    "delimiter" : "#",
                    "keep_both" : false,
                    "convert_type" : "t2s"
                }
         
       },
      
        "char_filter" : {
                "tsconvert" : {
                    "type" : "stconvert",
                    "convert_type" : "t2s"
                },
                "charconvert" : {
                  "type" : "mapping",
                   "mappings": [
                                "\\n=>",
                                "?=>",
                                "\\r=>",
                                "@ =>",
                                "/=> ",
                                "(=> ",
                                ")=> ",
                                "（=> ",
                                "）=> ",
                                "à=>a",
                                "á=>a",
                                "â=>a",
                                "ä=>a",
                                "À=>a",
                                "Â=>a",
                                "Ä=>a",
                                "è=>e",
                                "é=>e",
                                "ê=>e",
                                "ë=>e",
                                "È=>e",
                                "É=>e",
                                "Ê=>e",
                                "Ë=>e",
                                "î=>i",
                                "ï=>i",
                                "Î=>i",
                                "Ï=>i",
                                "ô=>o",
                                "ö=>o",
                                "Ô=>o",
                                "Ö=>o",
                                "ù=>u",
                                "û=>u",
                                "ü=>u",
                                "Ù=>u",
                                "Û=>u",
                                "Ü=>u",
                                "ç=>c",
                                "œ=>c",
                                "&=>",
                                "^=>",
                                ".=>",
                                "·=>",
                                "-=>",
                                "'=>",
                                "’=>",
                                "‘=>"
                              ]
                }
            }
     }
  },
  "mappings": {
    "dynamic": false, 
    "properties": {
        "art_name": {
          "type": "keyword",
          "fields": {
            "words":{
               "type": "text",
                 "store": false,
                 "term_vector": "with_offsets",
                 "analyzer": "ngramIndexAnalyzer",
                 "search_analyzer": "ngramSearchAnalyzer"
              
            },
            "pinyin":{
               "type": "text",
                "store": false,
                "term_vector": "with_offsets",
                "analyzer": "pinyin_analyzer"
              
            },
            "s_pinyin":{
               "type": "text",
                "store": false,
                "term_vector": "with_offsets",
                "analyzer": "pinyiSimpleIndexAnalyzer",
                "search_analyzer": "pinyiSimpleSearchAnalyzer"
              
            },
            "f_pinyin":{
               "type": "text",
                "store": false,
                "term_vector": "with_offsets",
                "analyzer": "pinyiFullIndexAnalyzer",
                "search_analyzer": "pinyiFullSearchAnalyzer"
              
            },
            "ik":{
              "type": "text",
              "store": false,
              "term_vector": "with_positions_offsets",
              "analyzer": "ikIndexAnalyzer",
              "search_analyzer": "ikSearchAnalyzer"
            }
          }
        },
        "art_name_all": {
          "type": "keyword",
          "fields": {
            "words":{
               "type": "text",
                 "store": false,
                 "term_vector": "with_offsets",
                 "analyzer": "ngramIndexAnalyzer",
                 "search_analyzer": "ngramSearchAnalyzer"
              
            },
            "pinyin":{
               "type": "text",
                "store": false,
                "term_vector": "with_offsets",
                "analyzer": "pinyin_analyzer"
              
            },
            "s_pinyin":{
               "type": "text",
                "store": false,
                "term_vector": "with_offsets",
                "analyzer": "pinyiSimpleIndexAnalyzer",
                "search_analyzer": "pinyiSimpleSearchAnalyzer"
              
            },
            "f_pinyin":{
               "type": "text",
                "store": false,
                "term_vector": "with_offsets",
                "analyzer": "pinyiFullIndexAnalyzer",
                "search_analyzer": "pinyiFullSearchAnalyzer"
              
            },
            "ik":{
              "type": "text",
              "store": false,
              "term_vector": "with_positions_offsets",
              "analyzer": "ikIndexAnalyzer",
              "search_analyzer": "ikSearchAnalyzer"
            }
          }
        },
        "artist": {
          "type": "keyword",
          "fields": {
            "words":{
               "type": "text",
                 "store": false,
                 "term_vector": "with_offsets",
                 "analyzer": "ngramIndexAnalyzer",
                 "search_analyzer": "ngramSearchAnalyzer"
              
            },
            "pinyin":{
               "type": "text",
                "store": false,
                "term_vector": "with_offsets",
                "analyzer": "pinyin_analyzer"
              
            },
            "s_pinyin":{
               "type": "text",
                "store": false,
                "term_vector": "with_offsets",
                "analyzer": "pinyiSimpleIndexAnalyzer",
                "search_analyzer": "pinyiSimpleSearchAnalyzer"
              
            },
            "f_pinyin":{
               "type": "text",
                "store": false,
                "term_vector": "with_offsets",
                "analyzer": "pinyiFullIndexAnalyzer",
                "search_analyzer": "pinyiFullSearchAnalyzer"
              
            },
            "ik":{
              "type": "text",
              "store": false,
              "term_vector": "with_positions_offsets",
              "analyzer": "ikIndexAnalyzer",
              "search_analyzer": "ikSearchAnalyzer"
            }
          }
        },
        "spec_name": {
          "type": "keyword",
          "fields": {
            "words":{
               "type": "text",
                 "store": false,
                 "term_vector": "with_offsets",
                 "analyzer": "ngramIndexAnalyzer",
                 "search_analyzer": "ngramSearchAnalyzer"
              
            },
            "pinyin":{
               "type": "text",
                "store": false,
                "term_vector": "with_offsets",
                "analyzer": "pinyin_analyzer"
              
            },
            "s_pinyin":{
               "type": "text",
                "store": false,
                "term_vector": "with_offsets",
                "analyzer": "pinyiSimpleIndexAnalyzer",
                "search_analyzer": "pinyiSimpleSearchAnalyzer"
              
            },
            "f_pinyin":{
               "type": "text",
                "store": false,
                "term_vector": "with_offsets",
                "analyzer": "pinyiFullIndexAnalyzer",
                "search_analyzer": "pinyiFullSearchAnalyzer"
              
            },
            "ik":{
              "type": "text",
              "store": false,
              "term_vector": "with_positions_offsets",
              "analyzer": "ikIndexAnalyzer",
              "search_analyzer": "ikSearchAnalyzer"
            }
          }
        },
        "ses_name": {
          "type": "keyword",
          "fields": {
            "words":{
               "type": "text",
                 "store": false,
                 "term_vector": "with_offsets",
                 "analyzer": "ngramIndexAnalyzer",
                 "search_analyzer": "ngramSearchAnalyzer"
              
            },
            "pinyin":{
               "type": "text",
                "store": false,
                "term_vector": "with_offsets",
                "analyzer": "pinyin_analyzer"
              
            },
            "s_pinyin":{
               "type": "text",
                "store": false,
                "term_vector": "with_offsets",
                "analyzer": "pinyiSimpleIndexAnalyzer",
                "search_analyzer": "pinyiSimpleSearchAnalyzer"
              
            },
            "f_pinyin":{
               "type": "text",
                "store": false,
                "term_vector": "with_offsets",
                "analyzer": "pinyiFullIndexAnalyzer",
                "search_analyzer": "pinyiFullSearchAnalyzer"
              
            },
            "ik":{
              "type": "text",
              "store": false,
              "term_vector": "with_positions_offsets",
              "analyzer": "ikIndexAnalyzer",
              "search_analyzer": "ikSearchAnalyzer"
            }
          }
        },
        "org_full_name": {
          "type": "keyword",
          "fields": {
            "words":{
               "type": "text",
                 "store": false,
                 "term_vector": "with_offsets",
                 "analyzer": "ngramIndexAnalyzer",
                 "search_analyzer": "ngramSearchAnalyzer"
              
            },
            "pinyin":{
               "type": "text",
                "store": false,
                "term_vector": "with_offsets",
                "analyzer": "pinyin_analyzer"
              
            },
            "s_pinyin":{
               "type": "text",
                "store": false,
                "term_vector": "with_offsets",
                "analyzer": "pinyiSimpleIndexAnalyzer",
                "search_analyzer": "pinyiSimpleSearchAnalyzer"
              
            },
            "f_pinyin":{
               "type": "text",
                "store": false,
                "term_vector": "with_offsets",
                "analyzer": "pinyiFullIndexAnalyzer",
                "search_analyzer": "pinyiFullSearchAnalyzer"
              
            },
            "ik":{
              "type": "text",
              "store": false,
              "term_vector": "with_positions_offsets",
              "analyzer": "ikIndexAnalyzer",
              "search_analyzer": "ikSearchAnalyzer"
            }
          }
        },
        "recommend": {
          "type": "integer"
        },
        "art_code": {
          "type": "keyword"
        },
        "cooperateStatus": {
          "type": "integer"
        },
        "auction_time": {
          "type": "date"
        },
        "deleted_flag":{"type":"integer"}

    }
  }
}
```





