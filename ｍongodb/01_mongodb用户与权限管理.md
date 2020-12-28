# mongodb 用户与权限管理
## mongodb常用权限
|权限名 | 描述|
| :-: | :-|
|`read` | 允许用户读取指定数据库|
|`readWrite` | 允许用户读写指定数据库|
|`dbAdmin` | 允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问`system.profile`|
|`userAdmin` | 允许用户向`system.users`集合写入，可以在指定数据库里创建、删除和管理用户|
|`clusterAdmin` | 只在`admin`数据库中可用，赋予用户所有分片和复制集相关函数的管理权限|
|`readAnyDatabase` | 只在`admin`数据库中可用，赋予用户所有数据库的读权限|
|`readWriteAnyDatabase` | 只在`admin`数据库中可用，赋予用户所有数据库的读写权限|
|`userAdminAnyDatabase` | 只在`admin`数据库中可用，赋予用户所有数据库的`userAdmin`权限|
|`dbAdminAnyDatabase` | 只在`admin`数据库中可用，赋予用户所有数据库的`dbAdmin`权限|
|`root` | 只在`admin`数据库中可用。超级账号，超级权限|

## 用户管理
> 对于一个新的`mongodb`服务器来说, 如果需要开启用户认证,在启动的时候要指定`auth=true`, 在命令行启动或者配置文件中增加相应的配置进行启动, 否则设置了用户名密码不生效

> `mongodb`提供了`Localhost Exception`机制来创建系统中第一个用户,

> 也就是说当系统中不存在用户的时候,我们可以使用`mongo --host localhost --port 27017`来登录系统并且创建第一个用户.

> 它要求第一个用户必须具有创建其他用户的权限，例如具有`userAdmin`或`userAdminAnyDatabase`角色的用户. 使用localhost异常的连接只能访问在admin数据库上创建第一个用户的权限。所以第一个用户直接创建一个`root`作为超级管理员就好
### 切换至`admin`库
对用户进行操作前, 必须要使用`use admin`先切换至`admin`库
```
use admin
```

### 使用`db.createUser(user, [writeConcern])`创建用户.
官方文档在[这里](https://docs.mongodb.com/manual/reference/method/db.createUser/ "createUser")

|参数 | 类型 | 描述 |
|--|--|--|
|`user`||document | 描述用户具体信息的document, 其中定义了将要创建的这个用户的权限. 详见下文|
|`writeConcern`|document|可选参数(默认为1). 定义这个用户操作数据时的[写入策略](/mongo-db-write-concerns).|

#### `user`参数详解
这是一段example:
```javascript
{
  user: "steven",
  pwd: "ThisIsPassword", // passwordPrompt(),      // Or  "<cleartext password>"
  customData: { info: "Some custom infomation" },
  roles: [
    { role: "<role>", db: "user" } | "<role>",
  ],
  authenticationRestrictions: [
     {
       clientSource: ["<IP>" | "<CIDR range>", ...],
       serverAddress: ["<IP>" | "<CIDR range>", ...]
     },
  ],
  mechanisms: [ "<SCRAM-SHA-1|SCRAM-SHA-256>", ... ],
  passwordDigestor: "<server|client>"
}
```
|参数 |类型 |描述|
|:-:|:-:|:-|
| `user` |`string`| 用户名.|
| `pwd`|`string`, `passwordPrompt()`| 用户密码, 可以明文将密码怼在这个参数中, 也可以使用`passwordPrompt()`参数(`version` >= 4.2), 这样的话, 在我们输入完参数按下回车之后, 会提示让你输入密码而不展示在`shell`中, 起到不明文显示密码的作用|
|`customData`|`document`|可选参数. 可以填写对这个用户的额外描述, `mongodb`会原样显示|
|`roles`|`array`|为这个用户赋予的权限,数组中可以直接填写权限名称如`["readWrite"]`, 也可以针对不同的数据库定义不同的权限如`[{ role: "<role>", db: "<database>" }]`|
|`authenticationRestrictions`|`array`|可选参数. Version >= 3.6. 限制该用户只能在特定的IP或者[CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing)范围内登录
|`mechanisms`|`array`|可选参数. version >= 4.0 指定用户登录时的认证机制, 用户必须匹配相同的认证机制才能生效. 如果mongodo服务器设置了服务器级别的`authenticationMechanisms`参数, 那么这个参数要设置成`authenticationMechanisms`参数的子集才会生效, 否则报错.<ul><li>当`featureCompatibilityVersion`为4.0时,这个参数默认是`SCRAM-SHA-1`和 `SCRAM-SHA-256`.</li><li>当`featureCompatibilityVersion`为3.6时默认是 `SCRAM-SHA-1`.</li></ul>|
|`passwordDigestor`|`string`| 可选参数. 设置服务器`"server"`还是客户端`"client"`提取密码.默认是`"server"`|

### 创建用户
```bash
> use admin
switched to db admin # 切换到admin库进行操作
> db.createUser({user: "stevenyin", pwd: "kpmg123", roles:[{db: "testdb", role: "readWrite"}]})
Successfully added user: {
        "user" : "stevenyin",
        "roles" : [
                {
                        "db" : "testdb",
                        "role" : "readWrite"
                }
        ]
}
```
### 使用新建的用户登录
```bash
> use admin
switched to db admin # 切换到admin库进行操作
> db.auth("stevenyin", "kpmg123")
1 # 1代表登录成功
```

### 查看用户信息
#### `db.getUser(userName)`
```
> use admin
switched to db admin
> db.getUser("stevenyin")
{
        "_id" : "admin.stevenyin",
        "userId" : UUID("98a75652-3dad-490e-b942-73879e2e5ad0"),
        "user" : "stevenyin",
        "db" : "admin",
        "roles" : [
                {
                        "role" : "readWrite",
                        "db" : "testdb"
                }
        ],
        "mechanisms" : [
                "SCRAM-SHA-1",
                "SCRAM-SHA-256"
        ]
}
```

#### `show users`
```
> use admin
switched to db admin
> show users
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
{
        "_id" : "admin.stevenyin",
        "userId" : UUID("98a75652-3dad-490e-b942-73879e2e5ad0"),
        "user" : "stevenyin",
        "db" : "admin",
        "roles" : [
                {
                        "role" : "readWrite",
                        "db" : "testdb"
                }
        ],
        "mechanisms" : [
                "SCRAM-SHA-1",
                "SCRAM-SHA-256"
        ]
}
```
#### 查看所有用户`db.system.users`
```
> use admin
switched to db admin
> db.system.users.find()
{ "_id" : "admin.root", "userId" : UUID("947f166b-16ba-4ac7-8afe-083bffb95fae"), "user" : "root", "db" : "admin", "credentials" : { "SCRAM-SHA-1" : { "iterationCount" : 10000, "salt" : "KVSkWypwi6RTrJfSCfK+ew==", "storedKey" : "TShonrWexxwoRBlU0XYZ9UQTdCU=", "serverKey" : "KVcEUM62sOAALYdS/zEEreeMOeQ=" }, "SCRAM-SHA-256" : { "iterationCount" : 15000, "salt" : "pNZspSaWbbcRMfu4Lb//qjbomqxe8E35lbZ0tg==", "storedKey" : "LYvB2ipYh40glZmLNhs/NS6HepYUuVq1h9ILCPx9PxQ=", "serverKey" : "WjJiFQMkFKpkWOPmb/kG7OsNlDRmU0CUtrkauGPFdNY=" } }, "customData" : { "information" : "first mongodb user" }, "roles" : [ { "role" : "root", "db" : "admin" } ] }
{ "_id" : "test.stevenyin", "userId" : UUID("f2a29e20-4187-469b-9476-581e44dac65f"), "user" : "stevenyin", "db" : "test", "credentials" : { "SCRAM-SHA-1" : { "iterationCount" : 10000, "salt" : "6a0sSJwsBjzaKH7qy3AFuw==", "storedKey" : "nVx8SEGeZu+NYYplp/yaRV4ev0c=", "serverKey" : "d6Xlu7i63ML2j5WzLyMnZzRDrGA=" }, "SCRAM-SHA-256" : { "iterationCount" : 15000, "salt" : "yueGclMO4EzcHVOmIwZ4GiC1wo6xbcbHZvzCKQ==", "storedKey" : "EZEoErUTClAF30Km7LOZfQ4hzsVBNN1+nojZOcvk/vc=", "serverKey" : "K2DFt3jbCDCyX4FSyt44sBU05bJCADeTN2sa7r4zR3g=" } }, "roles" : [ { "role" : "readWrite", "db" : "testdb" } ] }
```

### 更新用户信息`db.updateUser(username, update, [writeConcern])`
|参数|类型|描述|
|:-:|:-:|:-|
|`username`|`string`|需要更新的用户名|
|`update`|`document`|需要更新的用户信息, 全量替换之前的用户信息, 具体参数参见创建用户部分|
|`writeConcern`|`document`|可选参数(默认为1). 定义这个用户操作数据时的[写入策略](/mongo-db-write-concerns).|

### 修改密码`db.changeUserPassword(username, password)`
|参数|类型|描述|
|:-:|:-:|:-|
|`username`|`string`|需要更新的用户名|
|`password`|`string`, `passwordPrompt()`|修改后的密码, 同样可以使用`passwordPrompt()`来隐藏输入|

### 删除用户`db.dropUser(username, [writeConcern])`
|参数|类型|描述|
|:-:|:-:|:-|
|`username`|`string`|待删除的用户名|
|`writeConcern`|`document`|可选参数(默认为1). 定义这个用户操作数据时的[写入策略](/mongo-db-write-concerns).|

```
> use admin
switched to db admin
> db.dropUser("stevenyin")
true
> db.system.users.find()
{ "_id" : "admin.root", "userId" : UUID("947f166b-16ba-4ac7-8afe-083bffb95fae"), "user" : "root", "db" : "admin", "credentials" : { "SCRAM-SHA-1" : { "iterationCount" : 10000, "salt" : "KVSkWypwi6RTrJfSCfK+ew==", "storedKey" : "TShonrWexxwoRBlU0XYZ9UQTdCU=", "serverKey" : "KVcEUM62sOAALYdS/zEEreeMOeQ=" }, "SCRAM-SHA-256" : { "iterationCount" : 15000, "salt" : "pNZspSaWbbcRMfu4Lb//qjbomqxe8E35lbZ0tg==", "storedKey" : "LYvB2ipYh40glZmLNhs/NS6HepYUuVq1h9ILCPx9PxQ=", "serverKey" : "WjJiFQMkFKpkWOPmb/kG7OsNlDRmU0CUtrkauGPFdNY=" } }, "customData" : { "information" : "first mongodb user" }, "roles" : [ { "role" : "root", "db" : "admin" } ] }
```
