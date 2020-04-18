## 1.what

>中间件：应用是程序 + {中间软件} + 数据库（左中右）
>
>实现：天上的理念，落地的实践{mycat}；逻辑数据库，封装真实数据库
>
>mycat ：数据库 `中间件` 软件的具体 `实现` 
>
>历史：阿里 Cobar 演化而来
>
> 官网：http://www.mycat.io/

## 2.how
>读写分离

![1577793359709](D:\my_book\GrowthNotes\images\mycat\read_write.png)

>数据切分（水平拆分，垂直拆分） 数据瓶颈：单表500W,单库5000W

![1577673756753](D:\my_book\GrowthNotes\images\mycat\data_zone.png)

>多数据源整合（sql,nosql）

![1577674195716](D:\my_book\GrowthNotes\images\mycat\%5CUsers%5Cdangfeixiomany_datasource.png)

## 3.why

>mycat 的原理中最重要的一个动词是“拦截”，它拦截了用户发送过来的 SQL 语句，首先对 SQL 语句做了
>一些特定的分析：如分片分析、路由分析、读写分离分析、缓存分析等，然后将此 SQL 发往后端的真实数据库，并将返回的结果做适当的处理，最终再返回给用户。

![1577674557141](D:\my_book\GrowthNotes\images\mycat\why.png)

## 4. quick start

>依赖软件
>
>1. JDK ：https://blog.csdn.net/weixin_42266606/article/details/80863781
>2. mycli：https://blog.csdn.net/AnPHPer/article/details/80177105

```shell
# 修改主服务器授权
grant replication slave on  *.*  to  root@'%' identified by 'root';
# 刷新授权
flush privileges;
# 查看防火墙状态
firewall-cmd --state
# 停止firewall
systemctl stop firewalld.service
# 禁止firewall开机启动
systemctl disable firewalld.service 
# 下载 http://dl.mycat.io/
wget http://dl.mycat.io/1.6.5/Mycat-server-1.6.5-release-20180122220033-linux.tar.gz
# 解压
tar -zxvf Mycat-server-1.6.5-release-20180122220033-linux.tar.gz
# 移动
mv mycat /usr/local/
```

>修改配置文件 server.xml
>
>作用：修改用户信息，与MySQL区分

```xml
<user name="mycat">
    <property name="schemas">TESTDB</property>
	<property name="password">123456</property>
</user>
```

>修改配置文件 schema.xml
>
>作用：管理着 MyCat 的逻辑库、表、分片规则、DataNode 以及 DataSource

```xml
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">
<schema name="TESTDB" checkSQLschema="false" sqlMaxLimit="100" dataNode="dn1">
</schema>
<dataNode name="dn1" dataHost="host1" database="testdb" />
<dataHost name="host1" maxCon="1000" minCon="10" balance="0"
writeType="0" dbType="mysql" dbDriver="native" switchType="1" slaveThreshold="100">
	<heartbeat>select user()</heartbeat>
	<!-- can have multi write hosts -->
	<writeHost host="hostM1" url="192.168.140.128:3306" user="root" password="123123">
	<!-- can have multi read hosts -->
	<readHost host="hostS1" url="192.168.140.127:3306" user="root" password="123123" />
	</writeHost>
</dataHost>
</mycat:schema>
```

>启动

```shell
cd /usr/local/mycat/bin
# 控制台启动
./mycat console
# 后台启动
./mycat start
# 重启
./mycat restart
# 停止
./mycat stop
# 查看日志
tail -f /usr/local/mycat/logs/mycat.log
# 登录，访问 mycat 安装服务器
# 9066 管理维护 mycat
# 8066 查询数据
mysql -umycat -p123456 -P 9066/8066 -h 192.168.140.128
# 常用命令
show database
show @@help
```

## 5.read_write

> mycat 读写分离原理图

![1577794085039](D:\my_book\GrowthNotes\images\mycat\mycat_many_read_write.png)

>MYSQL 读写分离原理图

![mysql_read_write](D:\my_book\GrowthNotes\images\mycat\mysql_read_write.jpg)

>MYSQL 双主双从具体配置

| 序号 | 角色    | IP              |
| ---- | ------- | --------------- |
| 1    | Master1 | 192.168.140.128 |
| 2    | Slave1  | 192.168.140.127 |
| 3    | Master2 | 192.168.140.126 |
| 4    | Slave2  | 192.168.140.125 |

>Master1 配置
>
> 修改配置文件：vim /etc/my.cnf

```
#主服务器唯一ID
server-id=1

#启用二进制日志
log-bin=mysql-bin

#设置不要复制的数据库(可设置多个) 
binlog-ignore-db=mysql 
binlog-ignore-db=information_schema

#设置需要复制的数据库
binlog-do-db=需要复制的主数据库名字

#设置logbin格式
binlog_format=STATEMENT

#在作为从数据库的时候，有写入操作也要更新二进制日志文件
log-slave-updates

#表示自增长字段每次递增的量，指自增字段的起始值，其默认值是1，取值范围是1 .. 65535
auto-increment-increment=2

#表示自增长字段从哪个数开始，指字段一次递增多少，他的取值范围是1 .. 65535 
auto-increment-offset=1
```

>Master2 配置
>
>修改配置文件：vim /etc/my.cnf

```
#主服务器唯一ID
server-id=3

#启用二进制日志
log-bin=mysql-bin

#设置不要复制的数据库(可设置多个)
binlog-ignore-db=mysql
binlog-ignore-db=information_schema

#设置需要复制的数据库
binlog-do-db=需要复制的主数据库名字

#设置logbin格式
binlog_format=STATEMENT

#在作为从数据库的时候，有写入操作也要更新二进制日志文件
log-slave-updates

#表示自增长字段每次递增的量，指自增字段的起始值，其默认值是1，取值范围是1 .. 65535 
auto-increment-increment=2

#表示自增长字段从哪个数开始，指字段一次递增多少，他的取值范围是1 .. 65535 
auto-increment-offset=2
```

>Slave1配置
>
>修改配置文件：vim /etc/my.cnf

```
#从服务器唯一ID
server-id=2

#启用中继日志
relay-log=mysql-relay
```

>Slave2配置
>
>修改配置文件：vim /etc/my.cnf

```
#从服务器唯一ID
server-id=4

#启用中继日志
relay-log=mysql-relay
```

>其他配置

```
#在主机MySQL里执行授权命令
GRANT REPLICATION SLAVE ON *.* TO 'slave'@'%' IDENTIFIED BY '123123';

#查询Master1/Master2的状态
show master status;

#在从机上配置需要复制的主机(mysql命令行执行)，Slava1 复制 Master1，Slava2 复制 Master2
#在两个主机互相复制(mysql命令行执行)，Master2 复制 Master1，Master1 复制 Master2
CHANGE MASTER TO MASTER_HOST='主机的IP地址',
MASTER_USER='slave',
MASTER_PASSWORD='123123',
MASTER_LOG_FILE='mysql-bin.具体数字',MASTER_LOG_POS=具体值;

#启动两台从服务器复制功能
start slave;

#查看从服务器状态
show slave status\G

#下面两个参数都是Yes，则说明主从配置成功！
#	Slave_IO_Running: Yes
#	Slave_SQL_Running: Yes

### 注：以下命令是重置命令 ###
#如何停止从服务复制功能
stop slave;
#如何重新配置主从
stop slave;
reset master;
```

>修改 schema.xml 文件
>
>负载均衡类型，目前的取值有4 种：
>
>1. balance="0", 不开启读写分离机制，所有读操作都发送到当前可用的 writeHost 上。
>2. balance="1"，全部的 readHost 与 stand by writeHost 参与 select 语句的负载均衡，简单的说，当双主双从模式(M1->S1，M2->S2，并且 M1 与 M2 互为主备)，正常情况下，M2,S1,S2 都参与 select 语句的负载均衡。
>3. balance="2"，所有读操作都随机的在 writeHost、readhost 上分发。
>4. balance="3"，所有读请求随机的分发到 readhost 执行，writerHost 不负担读压力
>
>writeType配置：
>
>1. writeType="0"， 所有写操作发送到配置的第一个writeHost，第一个挂了切到还生存的第二个
>2. writeType="1"，所有写操作都随机的发送到配置的 writeHost，1.5 以后废弃不推荐
>
>switchType配置：
>
>1. switchType="1": 1 默认值，自动切换
>2. switchType="-1": -1 表示不自动切换
>3. switchType="2": 2 基于 MySQL 主从同步的状态决定是否切换

```xml
<dataNode name="dn1" dataHost="host1" database="testdb" />
<dataHost name="host1" maxCon="1000" minCon="10" balance="1" writeType="0" dbType="mysql" dbDriver="native" switchType="1" slaveThreshold="100" >
<heartbeat>select user()</heartbeat>

<!-- can have multi write hosts -->
<writeHost host="hostM1" url="192.168.140.128:3306" user="root" password="123123">
	<!-- can have multi read hosts -->
	<readHost host="hostS1" url="192.168.140.127:3306" user="root" password="123123" />
</writeHost>

<writeHost host="hostM2" url="192.168.140.126:3306" user="root" password="123123">
	<!-- can have multi read hosts -->
	<readHost host="hostS2" url="192.168.140.125:3306" user="root" password="123123" />
</writeHost>
</dataHost>
```

> 验证

```shell
# 重启 mycat
./mycat restart
# 在写主机 Master1 数据库表 mytbl 中插入带系统变量数据，造成主从数据不一致
INSERT INTO mytbl VALUES(3,@@hostname);
# 在 mycat 中查询数据
select * from mytbl; 
```

## 6.split_data

>如何划分数据
>
>问题1：两台主机上的两个数据库中的表，不可以关联查询
>
>分库的原则：有紧密关联关系的表应该在一个库里，相互没有关联关系的表可以分到不同的库里
>
>全局表：真实业务系统中，类似字典表（标签）
>
>1. 变动不频繁
>2. 数据量总体变化不大
>3. 数据规模不大，很少有超过数十万条记录
>
>需要注意的是，全局表每个分片节点上都要有运行创建表的 DDL 语句。

```xml
# 修改 schema.xml 配置文件
<table name="company" primaryKey="ID" type="global" dataNode="dn1,dn2,dn3" />
```

>ER表
>
>彻底解决 JION 的效率和性能问 题，提出了基于 E-R 关系的数据分片策略，子表的记录与所关联的父表记录存放在同一个数据分片上。

```xml
# 修改 schema.xml 配置文件
<table name="orders" dataNode="dn1,dn2"	rule="mod_rule" >
	<childTable name="orders_detail" primaryKey="id" joinKey="order_id" parentKey="id" /> </table>
```

>数据分片
>
>取模：此规则为对分片字段求摸运算。也是水平分表最常用规则。

```xml
<tableRule name="rule1">
<rule>
<columns>user_id</columns>
<algorithm>func1</algorithm>
</rule>
</tableRule>
<function name="func1" class="io.mycat.route.function.PartitionByLong">
<property name="partitionCount">2,1</property>
<property name="partitionLength">256,512</property>
</function>
```

>分片枚举：通过在配置文件中配置可能的枚举 id，全国省份区县固定的，这类业务使用本条规则.

```xml
#（1）修改schema.xml配置文件
<table name="orders_ware_info" dataNode="dn1,dn2" rule="sharding_by_intfile" ></table>

#（2）修改rule.xml配置文件
<tableRule name="sharding_by_intfile">
	<rule>
		<columns>areacode</columns>
		<algorithm>hash-int</algorithm>
	</rule>
</tableRule>

<function name="hash-int" class="io.mycat.route.function.PartitionByFileMap">
	<property name="mapFile">partition-hash-int.txt</property>
	<property name="type">1</property>
	<property name="defaultNode">0</property>
</function>

#	columns：分片字段，algorithm：分片函数
#	mapFile：标识配置文件名称，type：0为int型、非0为String，
#	defaultNode：默认节点:小于 0 表示不设置默认节点，大于等于 0 表示设置默认节点，
#	设置默认节点如果碰到不识别的枚举值，就让它路由到默认节点，如不设置不识别就报错

#（3）修改partition-hash-int.txt配置文件
110=0
120=1
```
