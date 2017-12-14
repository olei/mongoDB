## MongoDB 配置

### 安装
`sudo brew install mongodb --devel`

### 进入 /usr/local/bin
执行以下命令
```
$ sudo mongod --dbpath “/mongodb/data/db” --logpath “/mongodb/data/log/MongoDB.log” —auth

>use admin
>db.createUser({'user': 'root', 'pwd': '123456', 'roles': [{ role: "clusterAdmin", db: "admin" }, { role: "readAnyDatabase", db: "admin" }, “readWrite"]})
```
### 重启mongodb服务器
另外开一个终端进入/usr/local/bin
```
$ mongo
>db.auth(‘root’, ‘123456’)
>show dbs
```


**Built-In Roles（内置角色）：**

    1. 数据库用户角色：read、readWrite;
    
    2. 数据库管理角色：dbAdmin、dbOwner、userAdmin；
    
    3. 集群管理角色：clusterAdmin、clusterManager、clusterMonitor、hostManager；
    
    4. 备份恢复角色：backup、restore；
    
    5. 所有数据库角色：readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、dbAdminAnyDatabase
    
    6. 超级用户角色：root  
    
    *这里还有几个角色间接或直接提供了系统超级用户的访问（dbOwner 、userAdmin、userAdminAnyDatabase）*
    
    7. 内部角色：__system
    
    *PS：关于每个角色所拥有的操作权限可以点击上面的内置角色链接查看详情。*
    
  
**例如：在products数据库创建用户root，并给该用户admin数据库上clusterAdmin和readAnyDatabase的角色，products数据库上readWrite角色。**
```
> use products
> db.createUser({ 
    "user": "root",
    "pwd": "123456",
    "customData": { employeeId: 12345 },
    "roles" : [
        {role: "clusterAdmin", db: "admin"},
        {role: "readAnyDatabase", db: "admin" },
        "readWrite"
    ]
  },
  {w: "majority" , wtimeout: 5000})
```

## 导入导出库

### 导出
```
mongoexport -d dbname -c collectionname -o file --type json/csv -f field
```
* 参数说明：
    
 * -d ：数据库名

 * -c ：collection名

 * -o ：输出的文件名

 * --type ： 输出的格式，默认为json

 * -f ：输出的字段，如果-type为csv，则需要加上-f "字段名"
     
```
$ sudo mongoexport -u 'root' -p '123456' -d admin -c col -o '/baseBK/bk.json' --type json -f  "_id,user_id,user_name,age,status"
```

### 导入
```
mongoimport -d dbname -c collectionname --file filename --headerline --type json/csv -f field
```
* 参数说明：
* -d ：数据库名
* -c ：collection名
* --type ：导入的格式默认json
* -f ：导入的字段名
* --headerline ：如果导入的格式是csv，则可以使用第一行的标题作为导入的字段
* --file ：要导入的文件
```
$ sudo mongoimport -u 'root' -p '123456' -d admin -c col --file /baseBK/col.json --type json
```


## 设置主从数据库

### 创建Sharding复制集 rs0
```
$ mkdir /data/log
$ mkdir /data/db1
$ nohup mongod --port 27020 --dbpath=/data/db1 --logpath=/data/log/rs0-1.log --logappend --fork --shardsvr --replSet=rs0 &

$ mkdir /data/db2
$ nohup mongod --port 27021 --dbpath=/data/db2 --logpath=/data/log/rs0-2.log --logappend --fork --shardsvr --replSet=rs0 &
```

### 1.1 复制集rs0配置
```
$ mongo localhost:27020
> rs.initiate({_id: 'rs0', members: [{_id: 0, host: 'localhost:27020'}, {_id: 1, host: 'localhost:27021'}]})
> rs.isMaster() #查看主从关系
```
### 2. 创建Sharding复制集 rs1
```
$ mkdir /data/db3
$ nohup mongod --port 27030 --dbpath=/data/db3 --logpath=/data/log/rs1-1.log --logappend --fork --shardsvr --replSet=rs1 &
$ mkdir /data/db4
$ nohup mongod --port 27031 --dbpath=/data/db4 --logpath=/data/log/rs1-2.log --logappend --fork --shardsvr --replSet=rs1 &
```

### 2.1 复制集rs1配置
```
$ mongo localhost:27030
> rs.initiate({_id: 'rs1', members: [{_id: 0, host: 'localhost:27030'}, {_id: 1, host: 'localhost:27031'}]})
> rs.isMaster() #查看主从关系
```

### 3. 创建Config复制集 conf
```
$ mkdir /data/conf1
$ nohup mongod --port 27100 --dbpath=/data/conf1 --logpath=/data/log/conf-1.log --logappend --fork --configsvr --replSet=conf &
$ mkdir /data/conf2
$ nohup mongod --port 27101 --dbpath=/data/conf2 --logpath=/data/log/conf-2.log --logappend --fork --configsvr --replSet=conf &
```

### 3.1 复制集conf配置
```
$ mongo localhost:27100
> rs.initiate({_id: 'conf', members: [{_id: 0, host: 'localhost:27100'}, {_id: 1, host: 'localhost:27101'}]})
> rs.isMaster() #查看主从关系
```

### 4. 创建Route
```
$ nohup mongos --port 40000 --configdb conf/localhost:27100,localhost:27101 --fork --logpath=/data/log/route.log --logappend & 
```

### 4.1 设置分片
```
$ mongo localhost:40000
> use admin
> db.runCommand({ addshard: 'rs0/localhost:27020,localhost:27021'})
> db.runCommand({ addshard: 'rs1/localhost:27030,localhost:27031'})
> db.runCommand({ enablesharding: 'test'})
> db.runCommand({ shardcollection: 'test.user', key: {name: 1}})
```
