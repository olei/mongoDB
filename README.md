##MongoDB 配置

###安装
`sudo brew install mongodb --devel`

###进入 /usr/local/bin
执行以下命令
```
sudo mongod --dbpath “/mongodb/data/db” --logpath “/mongodb/data/log\MongoDB.log” —auth

>use admin
>db.createUser({'user': 'root', 'pwd': '123456', 'roles': [{ role: "clusterAdmin", db: "admin" }, { role: "readAnyDatabase", db: "admin" }, “readWrite"]})
```
###重启mongodb服务器
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
