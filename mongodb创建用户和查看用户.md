## mongodb
### 权限介绍

|权限名称|描述|
| -- | -- |
|read|允许用户读取指定数据库|
|readWrite|允许用户读写指定数据库|
|dbAdmin|允许用户再指定数据库中执行管理函数, 如索引创建,删除, 查看统计或访问system.profile|
|userAdmin|允许用户想system.users集合中写入, 可以再指定数据库里创建, 删除和管理用户|
|clusterAdmin|只在admin数据库中可用, 赋予用户所有分片和复制集相关函数的管理权限|
|readAnyDatabase|只在admin数据库中可用, 赋予用户所有数据库的读权限|
|readWriteAndyDatabase|只在admin数据库中可用, 赋予用户所有数据库的读写权限|
|userAdminAnyDatabase|只在admin数据库中可用, 赋予用户所有数据库的userAdmin权限|
|dbAdminAnyDatabase|只在admin数据库中可用, 赋予用户所有数据库的dbAdmin权限|
|root|只在admin数据库中可用, 超级账号拥有任意权限|

### 创建用户
1. 创建用户需要在`admin`库中执行，`db.system.users.find()`查看当前所有用户，`db.createUser({user:"root",pwd:"admin",roles:[{role:"userAdminAnyDatabase",db:"admin"}]})`
2. 开启权限，`mongodb.conf`配置文件中添加`auth=true`，重启`mongodb`。
3. 客户端再次登录需要`db.auth("root","admin")`
4. 创建某数据库的普通用户，`db.createUser({user:"stevenyin",pwd:"123456",roles:[{role:"readWrite",db:"testDb"}]})`，再次登录用户，然后切换数据库。


```bash
    db.createUser({
        "user": "root",
        "pwd": "root",
        "roles": [{
            "role": "root",
            "db": "admin"
        }],
        "customData": {
            "information": "first mongodb user"
        }
    });
```

创建完成后的返回结果: 会告诉你新增的用户信息, 当然`pwd`不会回显回来.
```bash
Successfully added user: {
        "user" : "root",
        "roles" : [
                {
                        "role" : "root",
                        "db" : "admin"
                }
        ],
        "customData" : {
                "information" : "first mongodb user"
        }
}
```

### 查看所有用户`show users`
```bash
show users
{
        "_id" : "admin.root",
        "userId" : UUID("947f166b-16ba-4ac7-8afe-083bffb95fae"),
        "user" : "root",
        "db" : "admin",
        "customData" : {
                "information" : "first mongodb user"
        },
        "roles" : [
                {
                        "role" : "root",
                        "db" : "admin"
                }
        ],
        "mechanisms" : [
                "SCRAM-SHA-1",
                "SCRAM-SHA-256"
        ]
}
```
### 查看所有用户`db.system.users.find()`
```bash
#查看所有table
> show tables
system.users
system.version
# 对system.users表执行查询全部操作
> db.system.users.find()
{ "_id" : "admin.root", "userId" : UUID("947f166b-16ba-4ac7-8afe-083bffb95fae"), "user" : "root", "db" : "admin", "credentials" : { "SCRAM-SHA-1" : { "iterationCount" : 10000, "salt" : "KVSkWypwi6RTrJfSCfK+ew==", "storedKey" : "TShonrWexxwoRBlU0XYZ9UQTdCU=", "serverKey" : "KVcEUM62sOAALYdS/zEEreeMOeQ=" }, "SCRAM-SHA-256" : { "iterationCount" : 15000, "salt" : "pNZspSaWbbcRMfu4Lb//qjbomqxe8E35lbZ0tg==", "storedKey" : "LYvB2ipYh40glZmLNhs/NS6HepYUuVq1h9ILCPx9PxQ=", "serverKey" : "WjJiFQMkFKpkWOPmb/kG7OsNlDRmU0CUtrkauGPFdNY=" } }, "customData" : { "information" : "first mongodb user" }, "roles" : [ { "role" : "root", "db" : "admin" } ] }
```
```properties

fork=true
auth=true
```

    db.createUser({
        "user": "test",
        "pwd": "123456",
        "roles": [{
            "role": "readWrite",
            "db": "melody"
        }],
        "customData": {
            "information": "melody admin"
        }
    });