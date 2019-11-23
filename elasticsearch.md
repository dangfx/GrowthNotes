### 公共资源

下载地址:https://www.elastic.co/cn/downloads/past-releases#elasticsearch
文档地址:https://www.elastic.co/guide/en/elasticsearch/reference/7.2/index.html
版本支持:https://www.elastic.co/cn/support/matrix

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

### 装中文分词插件

>网址：https://github.com/medcl/elasticsearch-analysis-ik

```shell
cd elasticsearch-7.2.1/plugins/
wget https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.2.1/elasticsearch-analysis-ik-7.2.1.zip
yum install unzip
unzip elasticsearch-analysis-ik-7.2.1.zip -d ik/
rm -rf elasticsearch-analysis-ik-7.2.1.zip
```

>重启es

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

中文分词效果测试

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

### `es` query

```json
// 集群健康状况
GET _cat/health
// 所有节点
GET _cat/nodes
// 查看所有索引
GET _cat/indices

// create mapping

```

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

