### 公共资源

> 1. 下载地址：https://www.elastic.co/cn/downloads/past-releases#elasticsearch
> 2. 文档地址：https://www.elastic.co/guide/en/elasticsearch/reference/7.2/index.html
> 3. 版本支持：https://www.elastic.co/cn/support/matrix
> 4. 参考资料（阮一鸣）：https://github.com/onebirdrocks/geektime-ELK

### 定义

>elasticsearch： 提供海量数据存储，近实时数据搜索与聚合功能
>
>beats：轻量的数据收集器
>
>logstash：做数据转换
>
>kibana：数据可视化

### 使用场景

>搜索，日志管理，安全分析，指标分析，业务分析

### 安装说明

>1. linux install V7.2.1
>2. elasticsearch 7.2.x ==> JDK1.8已上版本
>3. 创建组与用户：Caused by: java.lang.RuntimeException: can not run elasticsearch as root
>4. 安装根目录：/opt/software/

```shell
# 创建单独组与用户
groupadd elk
useradd elk -g elk
passwd elk
su elk
```
> `由于 elasticsearch + kibana 安装相对麻烦, 强烈建议通过 docker 构建环境 !!!`

### 安装 `elasticsearch`

```shell
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.2.1-linux-x86_64.tar.gz
tar -xzf elasticsearch-7.2.1-linux-x86_64.tar.gz
cd elasticsearch-7.2.1/
./bin/elasticsearch | ./bin/elasticsearch -d
```

### test `elasticsearch`

> curl http://192.168.114.131:9200

```json
{
  "name" : "localhost",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "sQ5_LL1MS8iKaVMAXsdIMA",
  "version" : {
    "number" : "7.2.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "fe6cb20",
    "build_date" : "2019-07-24T17:58:29.979462Z",
    "build_snapshot" : false,
    "lucene_version" : "8.0.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

### 安装 `kibana`

```shell
wget https://artifacts.elastic.co/downloads/kibana/kibana-7.2.1-linux-x86_64.tar.gz
tar -xzf kibana-7.2.1-linux-x86_64.tar.gz
# 备份配置文件,连接es
cd kibana-7.2.1-linux-x86_64/config
cp kibana.yml kibana.yml.bak
vim kibana.yml
# lasticsearch.hosts: ["http://192.168.114.131:9200"]
cd /opt/software/kibana-7.2.1-linux-x86_64/
./bin/kibana -c /opt/software/kibana-7.2.1-linux-x86_64/config/kibana.yml
```

### test `kibana`

> 浏览器访问：http://192.168.114.131:5601

### 装中文分词插件

>网址：https://github.com/medcl/elasticsearch-analysis-ik

```shell
cd elasticsearch-7.2.1/plugins/
wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.2.1/elasticsearch-analysis-ik-7.2.1.zip
yum install unzip
unzip elasticsearch-analysis-ik-7.2.1.zip -d ik/
rm -rf elasticsearch-analysis-ik-7.2.1.zip
```

>重启es，中文分词效果测试

```json
GET _analyze
{
  "analyzer":"ik_smart",
  "text":"美国留给伊拉克的是个烂摊子吗"
}

GET _analyze
{
  "analyzer":"ik_max_word",
  "text":"美国留给伊拉克的是个烂摊子吗"
}
```

>es 分词器分类
>
>英文空格切分
>
>中文 `icu_analyzer`全球化分词支持，  `ik`，  `THU`
>
>ICU：https://github.com/elastic/elasticsearch-analysis-icu
>
>IK：https://github.com/medcl/elasticsearch-analysis-ik
>
>THULAC：https://github.com/microbun/elasticsearch-thulac-plugin

```json
# standard 按词(空格)切分，小写处理
GET _analyze
{
  "analyzer": "standard",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}

# simple 按照非字母切分（符号被过滤），小写处理
GET _analyze
{
  "analyzer": "simple",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}

# whitespace 按照空格切分，不转小写
GET _analyze
{
  "analyzer": "whitespace",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}

# stop 小写处理，停用词过滤（the，a，is）
GET _analyze
{
  "analyzer": "stop",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}

# Keyword 不分词，直接将输入当作输出
GET _analyze
{
  "analyzer": "keyword",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}

# Patter 正则表达式，默认 \W+ (非字符分隔)
GET _analyze
{
  "analyzer": "pattern",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}

# Language – 提供了30多种常见语言的分词器
GET _analyze
{
  "analyzer": "english",
  "text": "2 running Quick brown-foxes leap over lazy dogs in the summer evening."
}
```

> 基本概念：
>
> 分词：将文本转换为倒排索引中的 terms 的过程，文本 ==> terms 
>
> 分词过程：character_filter[字符过滤器] ==>  tokenizer[分词器]  ==> token filetr [token过滤器]
>
> char_filter：html标签
>
> token filetr：字母大小写转换，删除停用词
>
> es 与 RDBMS对比

| RDBMS  | elasticsearch | desc                                    |
| ------ | ------------- | --------------------------------------- |
| db(table)  | index(type)   | es7以上只支持一个type，相似文档结构集合 |
| row    | document      |                                         |
| column | field         |                                         |
| schema | mapping       |                                         |
| SQL    | DSL           | Domain Specific Language：领域特定语言  |

>正排索引：文档ID到文档内容与单词的关联
>
>倒排索引：单词到文档ID的关联

| 文档ID | 文档内容                 |      | term          | count | document:postion |
| ------ | ------------------------ | ---- | ------------- | ----- | ---------------- |
| 1      | master elasticsearch     |      | elasticsearch | 3     | 1:1 2:0 3:0      |
| 2      | elasticsearch server     |      | master        | 1     | 1:0              |
| 3      | elasticsearch essentials |      | server        | 1     | 2:1              |
|        |                          |      | essentials    | 1     | 3:1              |

>term distionary（单词词典），记录所有文档单词，单词到倒排列表的关联关系，B+/hash拉链法实现
>
>posting list（倒排列表），记录了单词对应的文档组合，文档ID，词频，位置，偏移（单词高亮）

| 文档ID | 文档内容                 |      | docId | TF   | postion | offset |
| ------ | ------------------------ | ---- | ----- | ---- | ------- | ------ |
| 1      | master elasticsearch     |      | 1     | 1    | 1       | <7,20> |
| 2      | elasticsearch server     |      | 2     | 1    | 0       | <0,13> |
| 3      | elasticsearch essentials |      | 3     | 1    | 0       | <0,13> |


> ES CRUD 操作说明及操作

```json
1. index ==> 先删除，再创建
PUT my_index/_doc/1 
{"user":"mike", "comment":"test 111"}

2. create ==> 1. 如果id已经存在，报错  2. 自动生成 _id 
PUT my_index/_create/1 
{"user":"mike", "comment":"test 111"} 
POST my_index/_doc
{"user":"mike", "comment":"test 111"}

3. read
GET my_index/_doc/1

4. updat ==> 添加字段
POST my_index/_update/1
{"doc":{"user":"zs","comment":"test 222","message":"123"}}

5. delete
DELETE my_index/_doc/1

######## Create Document ########
#create document. 自动生成 _id
POST users/_doc
{
    "user" : "Mike",
    "post_date" : "2019-04-15T14:12:12",
    "message" : "trying out Kibana"
}

#create document. 指定Id。如果id已经存在，报错
PUT users/_doc/1?op_type=create
{
    "user" : "Jack",
    "post_date" : "2019-05-15T14:12:12",
    "message" : "trying out Elasticsearch"
}

#create document. 指定 ID 如果已经存在，就报错
PUT users/_create/1
{
    "user" : "Jack",
    "post_date" : "2019-05-15T14:12:12",
    "message" : "trying out Elasticsearch"
}

######## Get Document ########
#Get the document by ID
GET users/_doc/1

####### Update Document ######
#Update 指定 ID  (先删除，在写入)
PUT users/_doc/1
{
    "user" : "Mike"
}

#在原文档上增加字段
POST users/_update/1/
{
    "doc":{
        "post_date" : "2019-05-15T14:12:12",
        "message" : "trying out Elasticsearch"
    }
}

####### Delete Document ######
#Delete by Id
DELETE users/_doc/1
```
> SE URI search 查询

```json
# q 指定查询语句
# df指定查询字段
# sort 排序
# from/size 分页
# profile 查看乬是如何被执行的
GET /kibana_sample_data_ecommerce/_search?q=ZO0299602996&df=sku&sort=customer_id:desc&from=0&size=5
{
  "profile":"true"
}

# 指定单字段查询
GET /kibana_sample_data_ecommerce/_search?q=sku:ZO0299602996
{
  "profile":"true"
}

# PhraseQuery 等效 Eddie AND Underwood 与顺序一致
GET /kibana_sample_data_ecommerce/_search?q=customer_full_name:"Eddie Underwood"
{
  "profile":"true"
}

# 验证与顺序有关
GET /kibana_sample_data_ecommerce/_search?q=customer_full_name:"Underwood Eddie"
{
  "profile":"true"
}

# BooleanQuery 与顺序无关
GET /kibana_sample_data_ecommerce/_search?q=customer_full_name:(Underwood AND Eddie)
{
  "profile":"true"
}

# 等效于 Eddie OR MALE
GET /kibana_sample_data_ecommerce/_search?q=customer_full_name:Eddie MALE
{
  "profile":"true"
}

# customer_full_name:underwood -customer_full_name:eddie
GET /kibana_sample_data_ecommerce/_search?q=customer_full_name:(Underwood NOT Eddie)
{
  "profile":"true"
}

# 区间查询
GET /kibana_sample_data_ecommerce/_search?q=customer_id：<= 38
{
  "profile":"true"
}

GET /kibana_sample_data_ecommerce/_search?q=customer_id:[39 TO 39]
{
  "profile":"true"
}

#通配符查询
GET /kibana_sample_data_ecommerce/_search?q=currency:EUR*
{
  "profile":"true"
}

# 模糊匹配&近似度匹配
GET /kibana_sample_data_ecommerce/_search?q=category:Clothin~1
{
  "profile":"true"
}

# "Lord Rings" 单词中间可以间隔2单词
GET /kibana_sample_data_ecommerce/_search?q=category:"Lord Rings"~2
{
  "profile":"true"
}
```

>bulk的格式：
>{action:{metadata}}\n
>{requstbody}\n (请求体)
>
>action：(行为)，包含create（文档不存在时创建）、update（更新文档）、index（创建新文档或替换已用文档）、delete（删除一个文档）。
>create和index的区别：如果数据存在，使用create操作失败，会提示文档已存在，使用index则可以成功执行。
>metadata：(行为操作的具体索引信息)，需要指明数据的_index、_type、_id。

```json
# 示例说明1 bulk
{ "index" : { "_index" : "test", "_id" : "1" } }  //行为，索引信息
{ "field1" : "value1" }               //请求体         
{ "delete" : { "_index" : "test", "_id" : "2" } } //删除的批量操作不需要请求体
{ "create" : { "_index" : "test2", "_id" : "3" } }  //如果记录存在，创建失败
{ "field1" : "value3" }               //请求体
{ "update" : {"_id" : "1", "_index" : "test"} }   //更新不能缺失_id，文档不存在更新将会失败
{ "doc" : {"field2" : "value2"} }         //请求体

# 示例说明2 Multi Get
https://www.elastic.co/guide/en/elasticsearch/reference/7.1/docs-multi-get.html

GET /_mget
{
    "docs" : [
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "1"
        },
        {
            "_index" : "test",
            "_type" : "_doc",
            "_id" : "2"
        }
    ]
}
```

>mapping
>
>1. mapping 相当于 数据库中的 schema 
>2. 一个 type 有一个 mapping 定义
>3. es7.0 开始不需要在 mapping 中指定 type 信息

字段的数据类型：

| 数据类型                  | 类型描述 |
| ------------------------- | -------- |
| Text / keyword            | 简单类型 |
| Date                      | 简单类型 |
| Integer / Long / Floating | 简单类型 |
| Boolean                   | 简单类型 |
| IPV4 / IPV6               | 简单类型 |
| 嵌套类型 {}               | 复杂类型 |
| geo_point / geo_hash      | 特殊类型 |

类型的自动识别：

| JSON类型 | ES类型                         |
| -------- | ------------------------------ |
| string   | 1. 匹配日期格式设置为Date; 2.匹配数字格式设置为float/long; 3.设置Text,增加keyword子字段 |
| 布尔值 | boolean |
| 浮点值 | float |
| 整型值 | long |
| 对象 | object |
| 数组 | 有第一个非空数值的类型决定 |
| 空值 | 忽略 |

设置 `dynamic mappings`

|               | "true" | "false" | "static" |
| ------------- | ------ | ------- | -------- |
| 文档可索引    | Y      | Y       | N        |
| 字段可索引    | Y      | N       | N        |
| mapping被更新 | Y      | N       | N        |

> 测试脚本

```json
#默认Mapping支持dynamic，写入的文档中加入新的字段
PUT dynamic_mapping_test/_doc/1
{
  "newField":"someValue"
}

#该字段可以被搜索，数据也在_source中出现
POST dynamic_mapping_test/_search
{
  "query":{
    "match":{
      "newField":"someValue"
    }
  }
}

#修改为dynamic false
PUT dynamic_mapping_test/_mapping
{
  "dynamic": false
}

#新增 anotherField
PUT dynamic_mapping_test/_doc/10
{
  "anotherField":"someValue"
}

#该字段不可以被搜索，因为dynamic已经被设置为false
POST dynamic_mapping_test/_search
{
  "query":{
    "match":{
      "anotherField":"someValue"
    }
  }
}

#修改为strict
PUT dynamic_mapping_test/_mapping
{
  "dynamic": "strict"
}

#写入数据出错，HTTP Code 400
PUT dynamic_mapping_test/_doc/12
{
  "lastField":"value"
}

DELETE dynamic_mapping_test
```

>elasticsearch 聚合
>
>https://github.com/onebirdrocks/geektime-ELK/blob/master/part-1/3.14-Elasticsearch%E8%81%9A%E5%90%88%E5%88%86%E6%9E%90%E7%AE%80%E4%BB%8B/README.md


### `es` query

```json
// 集群健康状况
GET _cat/health
// 所有节点
GET _cat/nodes
// 查看所有索引
GET _cat/indices
```

> `实例_01`
> create mapping
>
> text 与 keyword 选择 ：https://blog.csdn.net/ahwsk/article/details/101272455

```json
PUT my_geo
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  },
  "mappings": {
    "properties": {
      "category":{
        "type": "keyword"
      },
      "location":{
        "type": "geo_point"
      },
      "address_name":{
        "type": "text",
        "analyzer": "ik_max_word",
        "fields": {
            "keyword": {
                "type": "keyword",
                "ignore_above": 256
            }
        }
      },
      "city":{
        "type": "keyword"
      },
      "desc":{
        "type": "text"
      }
    }
  }
}
```

> insert data

```json
PUT my_geo/_doc/1
{
  "address_name": "望京西地铁站",
  "city": "bj",
  "category": "1",
  "location": { 
    "lat": 40.001066,
    "lon": 116.456838
  },
  "desc": "途径公交车：471路; 547路; 571路; 851路; 854路; 866路; 882路; 905路; 928路; 970路快车; 980路快车区间; 985路; 987路; 丰宁城际班车; 快速直达专线197路; 快速直达专线29路; 滦平班车; 滦平城际班车; 兴隆班车; 兴隆城际班车; 专112路"
}

PUT my_geo/_doc/2
{
  "address_name": "望京地铁站",
  "city": "bj",
  "category": "1",
  "location": { 
    "lat": 40.004378,
    "lon": 116.476996
  },
  "desc": "途径公交车：621路; 855路; 专15路"
}

PUT my_geo/_doc/3
{
  "address_name": "将台地铁站",
  "city": "bj",
  "category": "1",
  "location": { 
    "lat": 39.97696,
    "lon": 116.496253
  }
}

PUT my_geo/_doc/4
{
  "address_name": "霍营地铁站",
  "city": "bj",
  "category": "1",
  "location": { 
    "lat": 40.075504,
    "lon": 116.368189
  },
  "desc": "途径地铁：地铁14号线东段"
}

PUT my_geo/_doc/5
{
  "address_name": "天通苑地铁站",
  "city": "bj",
  "category": "1",
  "location": { 
    "lat": 40.081478,
    "lon": 116.419233
  },
  "desc": "地址：北京市昌平区5号线天通苑站一层底商"
}

PUT my_geo/_doc/6
{
  "address_name": "TESTING-02 地铁站",
  "city": "bj",
  "category": "1",
  "location": "drm3btev3e86",
  "desc": "地址：test-02 address"
}

PUT my_geo/_doc/7
{
  "address_name": "TESTING-02 地铁站",
  "city": "bj",
  "category": "1",
  "location": "drm3btev3e86",
  "desc": "地址：test-02 address"
}
```

> simple query

```json
# term单词精确查询
GET my_geo/_search
{
  "query": {
    "term": {
      "address_name.keyword": "望京西地铁站"
    }
  }
}

# match针对text类型字段进行全文检索
GET my_geo/_search
{
  "query": {
    "match": {
      "address_name.keyword": "望京西地铁站"
    }
  }
}

# and query
GET my_geo/_search
{
  "query": {
    "bool": {
      "must": [
        {"term":{"category" : "1"}},
        {"term":{"address_name": "望京西地铁站"}}
      ]
    }
  }
}
```

>地理位置 query

```json
# 经纬度 + 距离 [圆形]
GET my_geo/_search
{
  "query": {
    "bool": {
      "must": [
        {"match_all": {}}
      ],
      "filter": {
        "geo_distance": {
          "distance": "10km",
          "location": {
            "lat": 40.004378,
            "lon": 116.476996
          }
        }
      }
    }
  }
}

# geohash
GET my_geo/_search
{
  "query": {
    "bool": {
      "must": [
        {"match_all": {}}
      ],
      "filter": {
        "geo_distance": {
          "distance": "1km",
          "location": "drm3btev3e86"
        }
      }
    }
  }
}
```

> `实例_02`
> create mapping
>

```json
# delete index
DELETE /sms-log-index

# create index
PUT /sms-log-index
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 2
  },
  "mappings": {
    "properties": {
      "corpName":{
        "type": "keyword"
      },
      "createDate":{
        "type": "date",
        "format": "yyyy-MM-dd HH:mm:ss||strict_date_optional_time||epoch_millis"
      },
      "fee":{
        "type": "long"
      },
      "ipAddr":{
        "type": "ip"
      },
      "longCode":{
        "type": "keyword"
      },
      "mobile":{
        "type": "keyword"
      },
      "operateId":{
        "type": "integer"
      },
      "province":{
        "type": "keyword"
      },
      "replyTatal":{
        "type": "integer"
      },
      "sendDate":{
        "type": "date"
      },
      "smsContent":{
        "type": "text",
        "analyzer": "ik_max_word"
      },
      "state":{
        "type": "integer"
      }
    }
  }
}
```

> init data
> `one doc` and `many doc`

```json
# delete doc by id
DELETE sms-log-index/_doc/1

# create one doc
PUT sms-log-index/_create/1
{
    "corpName" : "58同城",
    "createDate" : "2020-07-10",
    "sendDate" : "2020-07-11",
    "longCode" : "10690000988",
    "mobile" : "17710738946",
    "province" : "北京",
    "replyTatal" : 10,
    "smsContent" : "58同城作为中国的分类信息网站，本地化、自主且免费、真实高效是58同城网的三大特色。其服务覆盖生活的各个领域，提供房屋租售、招聘求职、二手买卖、汽车租售、宠物票务、餐饮娱乐、旅游交友等多种生活信息，覆盖中国所有大中城市。同时还为商家建立了全方位的市场营销解决方案，提供网站、直投杂志《生活圈》《好生活》、杂志展架、LED广告屏“社区快告”等多项服务，并为商家提供精准定向推广的多种产品，如“网邻通”、“名店推荐”等等。",
    "ipAddr" : "10.126.2.9",
    "operateId" : 1,
    "state" : 0,
    "fee" : 3
}

# create many doc
PUT sms-log-index/_bulk
{"index":{"_id":"2"}}
{"corpName":"自如","createDate":"2011-03-24T12:34:56","sendDate":"2016-07-08T23:32:12","longCode":"10690000945","mobile":"13710738989","province":"北京","replyTatal":18,"smsContent":"自如作为提供租房产品及服务的O2O互联网品牌，旗下现拥有自如友家、自如整租、自如寓、自如驿、自如民宿及业主直租六大产品线，自如友家和自如整租所有房屋均经过设计，实行统一装修、家居及家电配置。自如还提供包括保洁、家修、搬家及自如优品多项服务。","ipAddr":"10.134.5.6","operateId":2,"state":0,"fee":8,"location":{"lon":"116.505473","lat":"39.979685"}}
{"index":{"_id":"3"}}
{"corpName":"链家","createDate":"2009-02-24T12:34:56","sendDate":"2018-07-08T14:02:07","longCode":"10690000956","mobile":"5510738989","province":"北京","replyTatal":21,"smsContent":"北京链家房地产经纪有限公司（以下简称“链家”）成立于2001年，是一家集房产交易服务、资产管理服务为一体以数据驱动的价值链房产服务平台，业务覆盖二手房交易、新房交易、租赁、装修服务等。","ipAddr":"10.151.6.6","operateId":2,"state":0,"fee":9}
{"index":{"_id":"4"}}
{"corpName" : "蚂蚁金服","createDate" : "2010-11-14T12:34:56","sendDate" : "2015-03-08T12:32:12","longCode" : "10690000967","mobile" : "18510738989","province" : "杭州","replyTatal" : 1,"smsContent" : "蚂蚁金融服务集团（以下称“蚂蚁金服”）起步于2004年成立的支付宝。2013年3月，支付宝的母公司宣布将以其为主体筹建小微金融服务集团（以下称“小微金服”），小微金融（筹）成为蚂蚁金服的前身。2014年10月，蚂蚁金服正式成立  蚂蚁金服以“让信用等于财富”为愿景，致力于打造开放的生态系统，通过“互联网推进器计划” 助力金融机构和合作伙伴加速迈向“互联网+”，为小微企业和个人消费者提供普惠金融服务。依靠移动互联、大数据、云计算为基础，为中国践行普惠金融的重要实践。","ipAddr" : "10.122.5.6","operateId" : 2,"state" : 0,"fee" : 9}
{"index":{"_id":"5"}}
{"corpName" : "支付宝","createDate" : "2008-03-11T12:34:56","sendDate" : "2014-03-08T12:32:12","longCode" : "10690000967","mobile" : "18710738989","province" : "杭州","replyTatal" : 1,"smsContent" : "支付宝（中国）网络技术有限公司  是国内的第三方支付平台，致力于提供“简单、安全、快速”的支付解决方案 。支付宝公司从2004年建立开始，始终以“信任”作为产品和服务的核心。旗下有“支付宝”与“支付宝钱包”两个独立品牌。自2014年第二季度开始成为当前全球最大的移动支付厂商。","ipAddr" : "10.122.5.6","operateId" : 2,"state" : 0,"fee" : 5}
{"index":{"_id":"6"}}
{"corpName" : "华为","createDate" : "2001-03-14T12:34:56","sendDate" : "2019-03-08T12:32:12","longCode" : "10690000968","mobile" : "18910738989","province" : "深圳","replyTatal" : 25,"smsContent" : "华为技术有限公司（HUAWEI TECHNOLOGIES CO., LTD.）成立于1987年，总部位于中国广东省深圳市龙岗区。  华为是全球领先的信息与通信技术（ICT）解决方案供应商，专注于ICT领域，坚持稳健经营、持续创新、开放合作，在电信运营商、企业、终端和云计算等领域构筑了端到端的解决方案优势，为运营商客户、企业客户和消费者提供有竞争力的ICT解决方案、产品和服务，并致力于实现未来信息社会、构建更美好的全联接世界。2013年，华为首超全球第一大电信设备商爱立信，排名《财富》世界500强第315位。截至2016年底，华为有17万多名员工，华为的产品和解决方案已经应用于全球170多个国家，服务全球运营商50强中的45家及全球1/3的人口。","ipAddr" : "10.122.5.6","operateId" : 2,"state" : 0,"fee" : 5}
```

> qury doc
> `查询文档, 使用 POST {indexname}/_search 就可以了`

```json
# 2种分页查询

# 1.实时分页查询
POST sms-log-index/_search/
{
  "query": {
    "match_all": {}
  },
  "from": 0,
  "size": 6
}

# 2.从 es 上下文中拉取数据,类似游标 cursor
## 1>. scroll 将符合条件的数据ID保存到es的上下文中,下次查询直接去上下文中获取ID,查询数据, 1m代表es上下文缓存1分钟
POST sms-log-index/_search?scroll=1m
{
  "query": {
    "match_all": {}
  },
  "size": 2,
  "sort": [
    {
      "fee": {
        "order": "desc"
      }
    }
  ]
}

## 2> .获取下一页内容
POST /_search/scroll
{
  "scroll_id":"DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAEFYWY3o3VUJNWnVTd0tjYWhJaDM4Y2pBdw==",
  "scroll":"1m"
}

# term 单字段-单值 精确匹配查询
POST sms-log-index/_search
{
  "query": {
    "term": {
      "province": {
        "value": "杭州"
      }
    }
  }
}

# terms 单字段-多值查询 精确匹配查询
POST sms-log-index/_search
{
  "query": {
    "terms": {
      "province": [
        "杭州",
        "深圳"
      ]
    }
  }
}

# match 底层封装多个 term, keyword不会分词, text会分词
POST sms-log-index/_search
{
  "query": {
    "match_all": {}
  }
}

# 飞 熊 公司
GET _analyze
{
  "analyzer":"ik_max_word",
  "text":"解决方案"
}

# 将条件分词后再去匹配结果(使用 "公司" 去指定字段查询)
POST sms-log-index/_search
{
  "query": {
    "match": {
      "smsContent": "飞熊公司"
    }
  }
}

# 单 key 多字段 and 查询
POST sms-log-index/_search
{
  "query": {
    "match": {
      "smsContent": {
        "query": "支付宝 解决方案",
        "operator": "and"
      }
    }
  }
}

# 单 key 多字段 or 查询
POST sms-log-index/_search
{
  "query": {
    "match": {
      "smsContent": {
        "query": "支付宝 解决方案",
        "operator": "or"
      }
    }
  }
}

# 多 key 单字段查询
POST sms-log-index/_search
{
  "query": {
    "multi_match": {
      "query": "蚂蚁金服",
      "fields": ["corpName", "smsContent"]
    }
  }
}

# 多条件 and 查询 
POST sms-log-index/_search
{
  "query": {
    "bool": {
      "must": [
        {"term": {
          "corpName": {
            "value": "蚂蚁金服"
          }
        }},
        {"term": {
          "province": {
            "value": "杭州"
          }
        }}
      ]
    }
  }
}

# 多条件 or 查询
POST sms-log-index/_search
{
  "query": {
    "bool": {
      "should": [
        {"term": {
          "corpName": {
            "value": "蚂蚁金服"
          }
        }},
        {"term": {
          "province": {
            "value": "北京"
          }
        }}
      ]
    }
  }
}

# bool and[must] or[should] not[must not]
POST sms-log-index/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "operateId": 2
          }
        },
        {
          "match": {
            "fee": 5
          }
        },
        {
          "term": {
            "corpName": {
              "value": "华为"
            }
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "province": "北京"
          }
        }
      ]
    }
  }
}

# id 查询
POST sms-log-index/_search
{
  "query": {
    "ids": {
      "values": [1,2,3]
    }
  }
}

# 前缀查询 "corpName" 是 keyword 不会分词, match只能精确匹配
POST sms-log-index/_search
{
  "query": {
    "prefix": {
      "corpName": {
        "value": "自如"
      }
    }
  }
}

# 模糊查询(可以弱弱的纠错字)
POST sms-log-index/_search
{
  "query": {
    "fuzzy": {
      "corpName": {
        "value": "58同成"
        , "prefix_length": 3
      }
    }
  }
}

# 通配查询 * ? [58?? / 58*]
POST sms-log-index/_search
{
  "query": {
    "wildcard": {
      "corpName": {
        "value": "自*"
      }
    }
  }
}

# 正则表达式查询
POST sms-log-index/_search
{
  "query": {
    "regexp": {
      "mobile": "177[0-9]{8}"
    }
  }
}

# 范围查询 range
POST sms-log-index/_search
{
  "query": {
    "range": {
      "fee": {
        "gte": 8,
        "lte": 20
      }
    }
  }
}

# 将查询结果按照 score 重新排序
# positive 获取匹配结果集
# negative 匹配条件
# negative_boost 匹配条件数据的score *= negative_boost
POST sms-log-index/_search
{
  "query": {
    "boosting": {
      "positive": {
        "match": {
          "province": "杭州"
        }
      },
      "negative": {
        "match": {
          "smsContent": "蚂蚁金融"
        }
      },
      "negative_boost": 0.5
    }
  }
}

# filter 不为查询结果计算分数, 常用查询结果会缓存
# 性能高于 query 查询
POST sms-log-index/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "term": {
            "province": "杭州"
          }
        },
        {
          "range": {
            "fee": {
              "gte": 1,
              "lte": 20
            }
          }
        }
      ]
    }
  }
}

# 查询结果高亮显示
POST sms-log-index/_search
{
  "query": {
    "match": {
      "smsContent": "中国"
    }
  },
  "highlight": {
    "fields": {
      "smsContent": {
        "fragment_size": 10,
        "number_of_fragments": 1
      }
    },
    "pre_tags": "<font color='red'>",
    "post_tags": "</font>"
  }
}

```

> aggregations doc 文档信息统计

```json
# 范围统计查询
POST sms-log-index/_search
{
  "aggs": {
    "agg": {
      "range": {
        "field": "fee",
        "ranges": [
          {
            "to": 2
          },
          {
            "from": 2,
            "to": 5
          },
          {
            "from": 5
          }
        ]
      }
    }
  }
}

# 时间统计
POST sms-log-index/_search
{
  "aggs": {
    "agg": {
      "date_range": {
        "field": "createDate",
        "format": "yyyy-MM-dd", 
        "ranges": [
          {
            "from": "2001-01-01",
            "to": "2005-01-01"
          },
          {
            "from": "2005-01-01",
            "to": "2010-01-01"
          },
          {
            "from": "2010-01-01",
            "to": "now"
          }
        ]
      }
    }
  }
}

# IP 统计
POST sms-log-index/_search
{
  "aggs": {
    "agg": {
      "ip_range": {
        "field": "ipAddr",
        "ranges": [
          {
            "from": "10.151.6.6"
          },
          {
            "to": "10.151.6.6"
          }
        ]
      }
    }
  }
}

# 统计查询
POST sms-log-index/_search
{
  "aggs": {
    "agg": {
      "extended_stats": {
        "field": "fee"
      }
    }
  }
}
```

> geo doc
> - geo_distance 圆形区域检索[一个点]
> - geo_bounding_box 矩形区域检索[二个点]
> - geo_polygon 多边形区域检索[多个点]

```json
# 为索引添加 "location" 字段
PUT /sms-log-index/_mapping
{
  "properties":{
    "location":{
      "type":"geo_point"
    }
  }
}

# 为已有数据添加 "location" 数据
POST sms-log-index/_update/1
{
  "doc":{
      "location":{
      "lon": "116.513881",
      "lat": "39.991122"
    }
  }
}
POST sms-log-index/_update/2
{
  "doc":{
      "location":{
      "lon": "116.505473",
      "lat": "39.979685"
    }
  }
}
POST sms-log-index/_update/3
{
  "doc":{
      "location":{
      "lon": "116.505066",
      "lat": "39.979806"
    }
  }
}
POST sms-log-index/_update/4
{
  "doc":{
      "location":{
      "lon": "120.115002",
      "lat": "30.274505"
    }
  }
}
POST sms-log-index/_update/5
{
  "doc":{
      "location":{
      "lon": "120.115181",
      "lat": "30.273753"
    }
  }
}
POST sms-log-index/_update/6
{
  "doc":{
      "location":{
      "lon": "114.064078",
      "lat": "22.661593"
    }
  }
}

# geo 检索数据
# 一个点作为圆心检索
POST sms-log-index/_search
{
  "query": {
    "geo_distance":{
      "location":{
        "lon":"116.487323",
        "lat":"40.002242"
      },
      "distance":3000,
      "distance_type":"arc"
    }
  }
}

# 2个点构建矩形区域检索
POST sms-log-index/_search
{
  "query": {
    "geo_bounding_box":{
      "location":{
        "top_left":{
          "lon": "116.484915",
          "lat": "39.999672"
        },
        "bottom_right":{
          "lon": "116.519554",
          "lat": "39.97861"
        }
      }
    }
  }
}

# 多个点构建一个多边形区域检索
POST sms-log-index/_search
{
  "query": {
    "geo_polygon":{
      "location":{
        "points":[
          {"lon": "116.518692","lat": "39.97861"},
          {"lon": "116.502458","lat": "39.991889"},
          {"lon": "116.522293","lat": "40.004049"}
          ]
      }
    }
  }
}
```