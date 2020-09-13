## what is docker?

> We have a complete container solution for you - no matter who you are and where you are on your containerization journey.

## study url

-   官网：<https://www.docker.com>
-   远程仓库：<https://hub.docker.com>

## linux install docker

> 注: `sudo` 命令可以省略

1.文档地址

```shell
Centos 安装地址：
https://docs.docker.com/engine/install/centos/
```

2.Uninstall old versions

```shell
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

3.install

```shell
# Install using the repository
sudo yum install -y yum-utils

sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

# install docker engine
sudo yum install docker-ce docker-ce-cli containerd.io
```

4.add aliyuncs repository

```shell
sudo mkdir -p /etc/docker

# 下面配置是从阿里云网站获取
# https://homenew.console.aliyun.com
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://5qor11s0.mirror.aliyuncs.com"]
}
EOF

# 重新加载服务的配置文件
sudo systemctl daemon-reload
```

5.server command

```shell
# 启动 docker 服务
systemctl start docker
# 停止 docker 服务
systemctl stop docker
# 重启 docker 服务
systemctl restart docker
# 查看 docker 服务状态
systemctl status docker
# 设置 docker 服务开机启动
systemctl enable docker
```

6.image command

```shell
# 命令怎样使用 help 就够
docker --help
# 查看本地 docker 镜像列表
docker images
# 搜索 docker 远程仓库指定软件名称的镜像列表
docker search mysql
# 从远程仓库拉取指定软件版本镜像
docker pull mysql:5.7
```

7.container command

```shell
# 显示当前正在运行的容器
docker ps 
# 显示所有状态的容器及容器大小
docker ps -as
# 启动容器
docker container start <容器ID>
# 停止容器
docker container stop <容器ID>
# 删除容器
docker container rm <容器ID>
# 进入正在运行的容器
docker exec -it mysql /bin/bash
```

## software install

1.mysql

```shell
# 远程仓库下载 mysql 5.7 版本镜像 
docker pull mysql:5.7

# 启动镜像
# 命令参考：
# https://hub.docker.com/_/mysql?tab=description
docker run -p 3307:3306 --name mysql5.7 \
-v /opt/docker/mysql5.7:/etc/mysql/conf.d \
-v /opt/docker/mysql5.7/logs:/var/log/mysql \
-v /opt/docker/mysql5.7/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
-d mysql:5.7

# 查看容器
docker ps
# 查看容器启动日志
docker logs -f <容器名称>
# 进入容器
docker exec -it mysql5.7 /bin/bash
# 连接 mysql
mysql -u root -P 3306 -p123456
```

> mysql 启动镜像参数说明

```shell
--name mysql5.7 
	mysql的实例设置别名
-p 3307:3306
	3307 容器对外暴露的端口，3306 容器内部端口
-e MYSQL_ROOT_PASSWORD 
	设置mysql登录密码  
-d 
	以守护进程运行（后台运行） 最后 mysql5.7 是镜像名称
-v /opt/docker/mysql5.7/logs:/var/log/mysql
   挂载宿主目录到容器目录[重点:软件与数据隔离]
```

2.redis

```shell
# 远程仓库下载 redis 5.0 版本镜像
docker pull redis:5.0

# 创建目录(:/etc/redis/redis.conf没有这个文件, 需要提前创建)
mkdir -p /opt/docker/redis5.0/conf
touch /opt/docker/redis5.0/conf/redis.conf

# 启动镜像
# 命令参考：https://hub.docker.com/_/redis?tab=description
docker run --name redis5.0 -p 6379:6379 \
-v /opt/docker/redis5.0/data:/data \
-v /opt/docker/redis5.0/conf/redis.conf:/etc/redis/redis.conf \
-d redis:5.0 redis-server /etc/redis/redis.conf

# 查看容器
docker ps
# 查看容器启动日志
docker logs -f <容器名称>
# 1.进入容器启动
docker exec -it redis5.0 /bin/bash
# 2.连接 redis
redis-cli -h localhost -p 6379
# 以上2步可简化为下面1步
docker exec -it redis5.0 redis-cli -h localhost -p 6379
# redis 配置项说明地址
https://raw.githubusercontent.com/redis/redis/5.0/redis.conf
```

> redis 启动镜像参数说明

```shell
--name redis5.0
	redis 实例的别名
-p 6379:6379
	6379 linux端口，6379 容器端口
-d 
	以守护进程运行 redis:5.0 是镜像名称
-v /opt/docker/redis5.0/data:/data
	数据目录挂载配置
-v /opt/docker/redis5.0/conf/redis.conf:/etc/redis/redis.conf
	配置文件挂载配置
```

3.mongodb4.4

添加配置文件

> 创建配置文件：vi /opt/docker/mongo4.4/conf/mongod.conf
>
> 配置参考地址：https://docs.mongodb.com/v4.2/reference/configuration-options/index.html

```yml
systemLog:
   destination: file
   path: /data/log/mongod.log
   logAppend: true
storage:
   dbPath: /data/db
   journal:
      enabled: true
net:
   bindIp: 127.0.0.1
   port: 27017
setParameter:
   enableLocalhostAuthBypass: false
replication:
   replSetName: rs0
```

docker 启动 step

```shell
# 远程仓库下载 mongo 4.4 版本镜像
docker pull mongo:4.4
# 启动镜像
# 命令参考：https://hub.docker.com/_/mongo
docker run -p 27017:27017 --name mongo4.4 \
-v /opt/docker/mongo4.4/data:/data/db \
-v /opt/docker/mongo4.4/log:/data/log \
-v /opt/docker/mongo4.4/conf/mongod.conf:/etc/mongod.conf \
-d mongo:4.4 --replSet "rs0"
# 查看容器	
docker ps
# 查看容器启动日志
docker logs -f <容器名称>
# 1.进入容器启动
docker exec -it mongo4.4 /bin/bash
# 2.连接 mongodb
mongo --host 127.0.0.1 --port 27017
# 以上2步可简化为下面1步
docker exec -it mongo4.4 mongo --host 127.0.0.1 --port 27017
# 配置副本集
rs.initiate(
   {
      _id: "rs0",
      version: 1,
      members: [
         { _id: 0, host : "127.0.0.1:27017" }
      ]
   }
)
# 查看副本集
rs.status()
# 客户端工具地址
https://www.mongodb.com/try/download/compass
```

【一主二从】`mongodb` 复制集配置 

```shell
# 创建 mongo 所需挂在文件
mkdir -p /opt/docker/mongo4.4/{27017..27019}/data
mkdir -p /opt/docker/mongo4.4/{27017..27019}/log
mkdir -p /opt/docker/mongo4.4/{27017..27019}/conf

# 创建 27017/mongod.conf
cat > /opt/docker/mongo4.4/27017/conf/mongod.conf <<EOF
systemLog:
   destination: file
   path: /data/log/mongod.log
   logAppend: true
storage:
   dbPath: /data/db
   journal:
      enabled: true
net:
   bindIp: 0.0.0.0
   port: 27017
setParameter:
   enableLocalhostAuthBypass: false
replication:
   replSetName: rs0
EOF

# 创建 27018/mongod.conf
cat > /opt/docker/mongo4.4/27018/conf/mongod.conf <<EOF
systemLog:
   destination: file
   path: /data/log/mongod.log
   logAppend: true
storage:
   dbPath: /data/db
   journal:
      enabled: true
net:
   bindIp: 0.0.0.0
   port: 27018
setParameter:
   enableLocalhostAuthBypass: false
replication:
   replSetName: rs0
EOF

# 创建 27019/mongod.conf
cat > /opt/docker/mongo4.4/27019/conf/mongod.conf <<EOF
systemLog:
   destination: file
   path: /data/log/mongod.log
   logAppend: true
storage:
   dbPath: /data/db
   journal:
      enabled: true
net:
   bindIp: 0.0.0.0
   port: 27019
setParameter:
   enableLocalhostAuthBypass: false
replication:
   replSetName: rs0
EOF

```

【一主二从】`docker` 启动

```shell
# 虚拟出一块网卡(通信)
docker network create mongonetwork
# 查看网络配置
docker network inspect mongonetwork

# run docker
docker run -p 27017:27017 --name mongo4.4_27017 --net mongonetwork \
-v /opt/docker/mongo4.4/27017/data:/data/db \
-v /opt/docker/mongo4.4/27017/log:/data/log \
-v /opt/docker/mongo4.4/27017/conf/mongod.conf:/etc/mongod.conf \
-d mongo:4.4 --replSet "rs0"

docker run -p 27018:27017 --name mongo4.4_27018 --net mongonetwork \
-v /opt/docker/mongo4.4/27018/data:/data/db \
-v /opt/docker/mongo4.4/27018/log:/data/log \
-v /opt/docker/mongo4.4/27018/conf/mongod.conf:/etc/mongod.conf \
-d mongo:4.4 --replSet "rs0"

docker run -p 27019:27017 --name mongo4.4_27019 --net mongonetwork \
-v /opt/docker/mongo4.4/27019/data:/data/db \
-v /opt/docker/mongo4.4/27019/log:/data/log \
-v /opt/docker/mongo4.4/27019/conf/mongod.conf:/etc/mongod.conf \
-d mongo:4.4 --replSet "rs0"

# 查看 mongo 服务端口信息
netstat -anp | grep  270*
# 进入容器 查看具体 docker 容器 ip 地址
docker exec -it mongo4.4_27017 mongo --host 172.19.0.2 --port 27017
# 查看副本集
rs.status()
# 配置副本集
rs.initiate(
   {
      _id: "rs0",
      version: 1,
      members: [
         { _id: 0, host : "172.19.0.2:27017" },
         { _id: 1, host : "172.19.0.3:27017" },
         { _id: 2, host : "172.19.0.4:27017" }
      ]
   }
)
```

4.zookeeper

```shell
# 远程仓库下载 3.6.1 版本镜像
# https://zookeeper.apache.org/releases.html#download
# https://hub.docker.com/_/zookeeper?tab=description
docker pull zookeeper:3.6.1
# 临时启动镜像，获取zoo.cfg
mkdir -p zookeeper3.6.1/conf
docker run --name zookeeper3.6.1 -d zookeeper:3.6.1
docker exec -it zookeeper3.6.1 /bin/bash
# cp 容器中文件到物理机中
docker cp 2d7338ea06e3:/conf/zoo.cfg /opt/docker/zookeeper3.6.1/conf/
# 启动容器
docker run --name zookeeper3.6.1 \
-v /opt/docker/zookeeper3.6.1/data:/data \
-v /opt/docker/zookeeper3.6.1/log:/datalog \
-v /opt/docker/zookeeper3.6.1/conf/zoo.cfg:/conf/zoo.cfg \
-d zookeeper:3.6.1
# 查看容器
docker ps
# 查看容器启动日志
docker logs -f <容器名称>
# 进入容器启动-连接
docker exec -it zookeeper3.6.1 /bin/bash zkCli.sh
```

5.elasticsearch:7.6.2 / kibana:7.6.2

> 1.数据与配置无挂载方式

```shell
# 【选做】安装前为保证环境干净，删除所有容器
docker container stop $(docker ps -qa)
docker container rm $(docker ps -qa)

# 远程仓库下载 elasticsearch:7.6.2 版本镜像
docker pull elasticsearch:7.6.2

# 创建网络
docker network create somenetwork

# 启动镜像
docker run -d --name elasticsearch --net somenetwork -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms256m -Xmx256m" elasticsearch:7.6.2

# 查看是否启动
curl http://127.0.0.1:9200

# 进入容器
docker exec -it elasticsearch /bin/bash

########################################

# 远程仓库下载 kibana:7.6.2 版本镜像
docker pull kibana:7.6.2

# 启动镜像
docker run --name kibana --net somenetwork -p 5601:5601 -d kibana:7.6.2

# 查看是否启动
浏览器访问 http://ip:5601

# 进入容器
docker exec -it kibana /bin/bash

```

> 2.数据与配置挂载目录方式

```shell

# 创建网络 
# 会在物理机上虚拟出一块网卡 可以通过 ip addr 命令查看
docker network create somenetwork

# 查看网络配置 重要!!!
docker network inspect somenetwork

# 远程仓库下载 elasticsearch:7.6.2 版本镜像
docker pull elasticsearch:7.6.2

# 启动镜像
docker run --name elasticsearch --net somenetwork -p 9200:9200 -p 9300:9300 --privileged=true \
-v /opt/docker/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /opt/docker/elasticsearch/data:/usr/share/elasticsearch/data \
-v /opt/docker/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms256m -Xmx256m" \
-d elasticsearch:7.6.2

# 删除配置目录
rm -rf /opt/docker/elasticsearch/config/elasticsearch.yml

# 创建配置文件
cat > /opt/docker/elasticsearch/config/elasticsearch.yml <<EOF
http.host: 0.0.0.0
http.cors.enabled: true
http.cors.allow-origin: "*"
EOF

# 修改目录权限
chmod 777 /opt/docker/elasticsearch/data/

# 下载中文分词器插件
wget -P /opt/docker/elasticsearch https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.6.2/elasticsearch-analysis-ik-7.6.2.zip

# 解压插件
yum install -y unzip zip

# 注意:最后一个目录名称固定 "analysis-ik"
unzip /opt/docker/elasticsearch/elasticsearch-analysis-ik-7.6.2.zip -d /opt/docker/elasticsearch/plugins/analysis-ik

# 重新启动 es
docker container start elasticsearch

# 查看日志差价是否加载  ==>  "loaded plugin [analysis-ik]"
docker logs -f es | grep 'ik'

# 查看是否启动
curl http://0.0.0.0:9200
浏览器访问：http://192.168.5.128:9200/

# 进入容器
docker exec -it elasticsearch /bin/bash

########################################
# 远程仓库下载 kibana:7.6.2 版本镜像
docker pull kibana:7.6.2

# 启动镜像
docker run --name kibana --net somenetwork -p 5601:5601 \
-v /opt/docker/kibana/config/kibana.yml:/usr/share/kibana/config/kibana.yml \
-d kibana:7.6.2

# 删除配置目录
rm -rf /opt/docker/kibana/config/kibana.yml

##### [hosts 怎样确定] #####

# 选项说明
--net somenetwork

1. 指定了同一个自定义网络容器都在同一网段下
2. 容器中 /etc/hosts 文件会添加一个容器ID与给当前容器划分的IP
3. 容器间互连，可以通过IP[因为在同一个网段]，也可以通过容器名，docker 已经维护好了映射关系

# 查看网络配置
docker network inspect somenetwork

##### [hosts 怎样确定] #####

# 创建配置文件
cat > /opt/docker/kibana/config/kibana.yml <<EOF
server.name: kibana
server.host: "0.0.0.0"
elasticsearch.hosts: [ "http://172.18.0.2:9200" ]
xpack.monitoring.ui.container.elasticsearch.enabled: true
i18n.locale: "zh-CN"
EOF

# 或者 
cat > /opt/docker/kibana/config/kibana.yml <<EOF
server.name: kibana
server.host: "0.0.0.0"
elasticsearch.hosts: [ "http://elasticsearch:9200" ]
xpack.monitoring.ui.container.elasticsearch.enabled: true
i18n.locale: "zh-CN"
EOF

# 启动容器
docker container restart kibana

# 查看是否启动
浏览器访问 http://ip:5601

# 进入容器
docker exec -it kibana /bin/bash
```

6.prometheus/grafana

参考文档：

https://www.bookstack.cn/books/prometheus_practice

https://www.voidking.com/categories/

```shell
# ====> 安装 prometheus ====>
docker pull prom/prometheus:v2.21.0
mkdir -p /opt/docker/prometheus2.21.0/conf

# 创建 prometheus.yml 文件
cat > /opt/docker/prometheus2.21.0/conf/prometheus.yml <<EOF
global:
  scrape_interval: 15s #默认采集监控数据时间间隔
  external_labels:
    monitor: 'my-monitor'
scrape_configs:  #监控对象设置
  - job_name: prometheus #任务名称
    scrape_interval: 5s #每隔5s获取一次监控数据
    static_configs: #监控对象地址
      - targets: ['192.168.5.128:9090']  # 将自己加入到监控对象中
  - job_name: 'container'
    static_configs:
      - targets: ['192.168.5.128:8080']  # 本地 cadvisor 访问地址
EOF

# 创建 rules.yml 文件
cat > /opt/docker/prometheus2.21.0/conf/rules.yml <<EOF
EOF

# run prometheus
docker run --name=prometheus -d \
-p 9090:9090 \
-v /opt/docker/prometheus2.21.0/conf/prometheus.yml:/etc/prometheus/prometheus.yml \
-v /opt/docker/prometheus2.21.0/conf/rules.yml:/etc/prometheus/rules.yml \
prom/prometheus:v2.21.0 --config.file=/etc/prometheus/prometheus.yml --web.enable-lifecycle	

# ====> 安装 grafana ====>
docker pull grafana/grafana:7.1.5
docker run -d --name=grafana -p 3000:3000 grafana/grafana:7.1.5

# ====> 安装 cadvisor [监控docker] ====>
docker pull google/cadvisor:latest
docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  --privileged \
  --device=/dev/kmsg \
  google/cadvisor:latest
```

