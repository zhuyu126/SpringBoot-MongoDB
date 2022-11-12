# MongoDB 分片集群
## MongoDB下载与安装

### 1.MongoDB下载

```bash
# 1.下载社区版 MongoDB 4.1.3
#   下载地址：https://www.mongodb.com/download-center#community
yum install wget
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-4.1.3.tgz
# 2.将压缩包解压即可
mkdir /usr/local/mongodb/
tar -zxvf mongodb-linux-x86_64-rhel70-4.1.3.tgz -C /usr/local/hero/

# 3.创建数据目录和日志目录
cd /usr/local/mongodb/mongodb-linux-x86_64-rhel70-4.1.3
mkdir datas
mkdir logs
mkdir conf
touch logs/mongodb.log
# 4.创建mongodb.conf文件
vim /usr/local/mongodb/mongodb-linux-x86_64-rhel70-4.1.3/conf/mongo.conf
# 5.指定配置文件方式的启动服务端
/usr/local/mongodb/mongodb-linux-x86_64-rhel70-4.1.3/bin/mongod -f /usr/local/mongodb/mongodb-linux-x86_64-rhel70-4.1.3/conf/mongo.conf
```

### 2. 配置文件

````bash
#监听的端口，默认27017
port=27017
#数据库目录，默认/data/db
dbpath=/usr/local/mongodb/mongodb-linux-x86_64-rhel70-4.1.3/datas
#日志路径
logpath=/usr/local/mongodb/mongodb-linux-x86_64-rhel70-4.1.3//logs/mongodb.log
#是否追加日志
logappend=true
#是否已后台启动的方式登陆
fork=true
#监听IP地址，默认全部可以访问
bind_ip=0.0.0.0
# 是开启用户密码登陆
auth=false
````

### 3. 启动脚本-start-mongo.sh

```bash
vim start-mongo.sh
## 脚本内容
#! /bin/bash
clear
/usr/local/hero/mongodb-linux-x86_64-rhel70-4.1.3/bin/mongod -f /usr/local/hero/mongodb-linux-x86_64-rhel70-4.1.3/conf/mongo.conf
echo "start mongo..."
ps -ef | grep mongodb
## 脚本赋权
chmod 755 start-mongo.sh
```

### 4.关闭脚本-stop-mongo.sh

```bash
vim  stop-mongo.sh
## 脚本内容
#! /bin/bash
clear
/usr/local/hero/mongodb-linux-x86_64-rhel70-4.1.3/bin/mongod --shutdown -f /usr/local/hero/mongodb-linux-x86_64-rhel70-4.1.3/conf/mongo.conf
echo "stop mongo..."
ps -ef | grep mongodb
## 脚本内容
chmod 755 stop-mongo.sh
```

## MongoDB分片集群

### 1. config 节点集群

#### 1.1 初始化config节点集群相关信息

```bash
# 初始化集群数据文件存储目录和日志文件
mkdir -p /data/mongo/config1
mkdir -p /data/mongo/config2
mkdir -p /data/mongo/config3
# 初始化日志文件
touch /data/mongo/logs/config1.log
touch /data/mongo/logs/config2.log
touch /data/mongo/logs/config3.log
# 创建集群配置文件目录
mkdir /root/mongoconfig
```

#### 1.2 创建配置文件并启动config节点集群 

```bash
# 创建配置文件
# 节点1 config-17017.conf  
tee /root/mongoconfig/config-17017.conf <<-'EOF'
# 数据库文件位置
dbpath=/data/mongo/config1
#日志文件位置
logpath=/data/mongo/logs/config1.log
# 以追加方式写入日志
logappend=true
# 是否以守护进程方式运行
fork = true
bind_ip=0.0.0.0
port = 17017
# 表示是一个配置服务器
configsvr=true
#配置服务器副本集名称
replSet=configsvr
EOF
# 节点2 config-17018.conf  
tee /root/mongoconfig/config-17018.conf <<-'EOF'
# 数据库文件位置
dbpath=/data/mongo/config2
#日志文件位置
logpath=/data/mongo/logs/config2.log
# 以追加方式写入日志
logappend=true
# 是否以守护进程方式运行
fork = true
bind_ip=0.0.0.0
port = 17018
# 表示是一个配置服务器
configsvr=true
#配置服务器副本集名称
replSet=configsvr
EOF
# 节点3 config-17019.conf  
tee /root/mongoconfig/config-17019.conf <<-'EOF'
# 数据库文件位置
dbpath=/data/mongo/config3
#日志文件位置
logpath=/data/mongo/logs/config3.log
# 以追加方式写入日志
logappend=true
# 是否以守护进程方式运行
fork = true
bind_ip=0.0.0.0
port = 17019
# 表示是一个配置服务器
configsvr=true
#配置服务器副本集名称
replSet=configsvr
EOF
```

#### 1.3 config节点集群启动集群脚本

```bash
tee /root/mongoconfig/start-mongo-config.sh <<-'EOF'
#! /bin/bash
clear
/usr/local/mongodb/mongodb-linux-x86_64-rhel70-4.1.3/bin/mongod -f /root/mongoconfig/config-17017.conf
/usr/local/mongodb/mongodb-linux-x86_64-rhel70-4.1.3/bin/mongod -f /root/mongoconfig/config-17018.conf
/usr/local/mongodb/mongodb-linux-x86_64-rhel70-4.1.3/bin/mongod -f /root/mongoconfig/config-17019.conf
echo "start mongo config cluster..."
ps -ef | grep mongodb
EOF
# 脚本赋权
chmod 755 /root/mongoconfig/start-mongo-config.sh
```

#### 1.4 config节点集群关闭集群脚本

```bash
tee /root/mongoconfig/stop-mongo-config.sh  <<-'EOF'
#! /bin/bash
clear
/usr/local/mongodb/mongodb-linux-x86_64-rhel70-4.1.3/bin/mongod --shutdown -f /root/mongoconfig/config-17017.conf
/usr/local/mongodb/mongodb-linux-x86_64-rhel70-4.1.3/bin/mongod --shutdown -f /root/mongoconfig/config-17018.conf
/usr/local/mongodb/mongodb-linux-x86_64-rhel70-4.1.3/bin/mongod --shutdown -f /root/mongoconfig/config-17019.conf
echo "stop mongo config cluster..."
ps -ef | grep mongodb
EOF
# 脚本赋权
chmod 755 /root/mongoconfig/stop-mongo-config.sh
```

进入任意节点的mongo shell 并添加 配置节点集群

注意use admin  

```bash
/usr/local/mongodb/mongodb-linux-x86_64-rhel70-4.1.3/bin/mongo --host=172.17.187.80 --port=17017
use admin
var cfg ={"_id":"configsvr",
        "members":[
            {"_id":1,"host":"192.168.56.4:17017"},
            {"_id":2,"host":"192.168.56.4:17018"},
            {"_id":3,"host":"192.168.56.4:17019"}]
        };
rs.initiate(cfg)
rs.status()
```

### 2.配置shard1和shard2集群

#### 2.1 shard1集群搭建37017到37019

```bash
# 1）初始化集群数据文件存储目录和日志文件
mkdir -p /data/mongo/datashard/server1
mkdir -p /data/mongo/datashard/server2
mkdir -p /data/mongo/datashard/server3

mkdir /data/mongo/logs/datashard
touch /data/mongo/logs/datashard/server1.log
touch /data/mongo/logs/datashard/server2.log
touch /data/mongo/logs/datashard/server3.log

mkdir /root/mongoshard

# 2）主节点配置 mongo_37017.conf
tee /root/mongoshard/mongo_37017.conf <<-'EOF'
# 主节点配置
dbpath=/data/mongo/datashard/server1
bind_ip=0.0.0.0
port=37017
fork=true
logpath=/data/mongo/logs/datashard/server1.log
# 集群名称
replSet=shard1
shardsvr=true
EOF

# 2）从节点1配置 mongo_37018.conf  
tee /root/mongoshard/mongo_37018.conf <<-'EOF'
dbpath=/data/mongo/datashard/server2
bind_ip=0.0.0.0
port=37018
fork=true
logpath=/data/mongo/logs/datashard/server2.log
replSet=shard1
shardsvr=true
EOF

# 3）从节点2配置 mongo_37019.conf
tee /root/mongoshard/mongo_37019.conf <<-'EOF'
dbpath=/data/mongo/datashard/server3
bind_ip=0.0.0.0
port=37019
fork=true
logpath=/data/mongo/logs/datashard/server3.log
replSet=shard1
shardsvr=true
EOF
```

#### 2.2 shard1集群启动脚本

```bash
tee /root/mongoshard/start-mongo-shard1.sh <<-'EOF'
#! /bin/bash
clear
/usr/local/mongodb/mongodb-linux-x86_64-rhel70-4.1.3/bin/mongod -f /root/mongoshard/mongo_37017.conf
/usr/local/mongodb/mongodb-linux-x86_64-rhel70-4.1.3/bin/mongod -f /root/mongoshard/mongo_37018.conf
/usr/local/mongodb/mongodb-linux-x86_64-rhel70-4.1.3/bin/mongod -f /root/mongoshard/mongo_37019.conf 
echo "start mongo shard1 cluster..."
ps -ef | grep mongodb
EOF
# 脚本赋权
chmod 755 /root/mongoshard/start-mongo-shard1.sh
```

#### 2.3 shard1 集群关闭脚本

```bash
tee /root/mongoshard/stop-mongo-shard1.sh <<-'EOF'
#! /bin/bash
clear
/usr/local/hero/mongodb-linux-x86_64-rhel70-4.1.3/bin/mongod --shutdown -f /root/mongoshard/mongo_37017.conf
/usr/local/hero/mongodb-linux-x86_64-rhel70-4.1.3/bin/mongod --shutdown -f /root/mongoshard/mongo_37018.conf
/usr/local/hero/mongodb-linux-x86_64-rhel70-4.1.3/bin/mongod --shutdown -f /root/mongoshard/mongo_37019.conf
echo "stop mongo shard1 cluster..."
ps -ef | grep mongodb
EOF
# 脚本赋权
chmod 755 /root/mongoshard/stop-mongo-shard1.sh
```

#### 2.4 初始化集群命令

启动三个节点 然后进入 Primary 节点 运行如下命令：

```bash
/usr/local/mongodb/mongodb-linux-x86_64-rhel70-4.1.3/bin/mongo --host=172.17.187.80 --port=37017
var cfg ={"_id":"shard1",
            "protocolVersion" : 1,
            "members":[
                {"_id":1,"host":"192.168.56.4:37017"},
                {"_id":2,"host":"192.168.56.4:37018"},
                {"_id":3,"host":"192.168.56.4:37019"}
            ]
        }
```

#### 2.5 shard2集群搭建47017到47019

shard2集群的搭建与shard1一致不做赘述

![集群2状态](http://image.crystallee.top/%E9%9B%86%E7%BE%A42%E7%8A%B6%E6%80%81.png)

### 3. 路由节点的配置和启动

```bash
tee /root/mongoshard/route-27017.conf <<-'EOF'
port=27017
bind_ip=0.0.0.0
fork=true
logpath=/data/mongo/logs/route.log
configdb=configsvr/192.168.56.4:17017,192.168.56.4:17018,192.168.56.4:17019
EOF
```

⚠️注意:启动路由节点使用 mongos （注意不是mongod） 

```bash
/usr/local/mongodb/mongodb-linux-x86_64-rhel70-4.1.3/bin/mongos -f /root/mongoshard/route-27017.conf
```

### 4.mongos(路由)中添加分片节点

进入路由mongos  节点

![路由节点](http://image.crystallee.top/%E8%B7%AF%E7%94%B1%E8%8A%82%E7%82%B9.png)

```bash
/usr/local/mongodb/mongodb-linux-x86_64-rhel70-4.1.3/bin/mongo --port 27017
sh.status()
sh.addShard("shard1/192.168.56.4:37017,192.168.56.4:37018,192.168.56.4:37019");
sh.addShard("shard2/192.168.56.4:47017,192.168.56.4:47018,192.168.56.4:47019");
sh.status()
```

![添加集群](http://image.crystallee.top/%E6%B7%BB%E5%8A%A0%E9%9B%86%E7%BE%A4.png)

![分片集群后](http://image.crystallee.top/%E5%88%86%E7%89%87%E9%9B%86%E7%BE%A4%E5%90%8E.png)

### 5. 开启数据库和集合分片(指定片键)

```bash
# 为数据库开启分片功能
use admin
db.runCommand( { enablesharding :"myRangeDB"});
# 为指定集合开启分片功能
db.runCommand( { shardcollection : "myRangeDB.coll_shard",key : {_id: 1} } )

# 查看分片情况
db.coll_shard.stats();
sharded  true
# 可以观察到当前数据全部分配到了一个shard集群上。这是因为MongoDB并不是按照文档的级别将数据散落在各个分片上的，而是按照范围分散的。也就是说collection的数据会拆分成块chunk，然后分布在不同的shard
# 这个chunk很大，现在这种服务器配置，只有数据插入到一定量级才能看到分片的结果
# 默认的chunk大小是64M，可以存储很多文档

# 查看chunk大小：
use config
db.settings.find()
# 修改chunk大小
db.settings.save( { _id:"chunksize", value: NumberLong(128)} )

# 使用hash分片
use admin
db.runCommand({"enablesharding":"myHashDB"})
db.runCommand({"shardcollection":"myHashDB.coll_shard","key":{"_id":"hashed"}})
```

## SpringBoot 对MongoDB分片集群进行相关操作

### 1. 添加

![添加](http://image.crystallee.top/%E6%B7%BB%E5%8A%A0.png)

### 2.查询所有

![查找所有](http://image.crystallee.top/%E6%9F%A5%E6%89%BE%E6%89%80%E6%9C%89.png)

### 3. 按条件查询

#### 3.1 根据名称查询

![根据名称查找](http://image.crystallee.top/%E6%A0%B9%E6%8D%AE%E5%90%8D%E7%A7%B0%E6%9F%A5%E6%89%BE.png)

#### 3.2 根据ID查询

![根据ID查找](http://image.crystallee.top/%E6%A0%B9%E6%8D%AEID%E6%9F%A5%E6%89%BE.png)

### 4. 更新

![更新](http://image.crystallee.top/%E6%9B%B4%E6%96%B0.png)

更新后查询

![更新后查询](http://image.crystallee.top/%E6%9B%B4%E6%96%B0%E5%90%8E%E6%9F%A5%E8%AF%A2.png)

### 5 删除

![删除](http://image.crystallee.top/%E5%88%A0%E9%99%A4.png)

删除后查询

![删除后查询](http://image.crystallee.top/%E5%88%A0%E9%99%A4%E5%90%8E%E6%9F%A5%E8%AF%A2.png)
