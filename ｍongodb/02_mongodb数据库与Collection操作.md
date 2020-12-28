# mongodb数据库与Collection操作

## 数据库操作
### 创建数据库
在MongoDB中创建数据库的命令使用的是use命令。该命令有两层含义：
1. 切换到指定数据库。
2. 如果切换的数据库不存在，则创建该数据库。
我们使用use命令创建一个名为`sxttest`的数据库。
```bash
use sxttest
```
### 查看数据库
我们可以通过`show dbs` 或者 `show databases` 命令查看当前MongoDB中的所有数据库。
### 删除数据库
在MongoDB中使用`db.dropDatabase()`函数来删除数据库。在删除数据库之前，需要使用具备`dbAdminAnyDatabase`角色的管理员用户登录，然后切换到需要删除的数据库，执行`db.dropDatabase()`函数即可。删除成功后会返回一个`{ "ok" : 1 }`的JSON字符串。

## Collection操作
### 查看集合
可以使用`show collections`或`show tables`来查看当前选中的数据库中的集合
```bash
> use admin
switched to db admin
> show  dbs
admin   0.000GB
config  0.000GB
local   0.000GB
melody  0.000GB
test    0.000GB
> show collections
learning
system.users
system.version
> show tables
learning
system.users
system.version
```
### `db.<collectionName>.stats()`查看集合详情
```
> db.learning.stats()
{
        "ns" : "admin.learning",
        "size" : 0,
        "count" : 0,
        "storageSize" : 4096,
        "capped" : false,
        "wiredTiger" : {
                "metadata" : {
                        "formatVersion" : 1
                },
                "creationString" : "....",
                "type" : "file",
                "uri" : "statistics:table:collection-34-5663533932999800395",
    .....
    ....
    ....
        }
    .....
}
```
### 插入数据的方式, 默认创建集合
在MongoDB中，我们也可以不用创建集合，当我们插入一些数据时，会自动创建集合，并且会使用文档管理命令中的集合名称作为集合的名称。文档管理命令后续课程详解。
向库中插入一条数据
```bash
# 查看当前数据库, 没有test库
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
melody  0.000GB
# 在没有test库的情况下直接use, 不会报错
> use test
switched to db test
# 对test库中不存在的coll1集合直接进行插入数据的操作, 不会报错并且提示插入成功
> db.coll1.insert({"name":"stevenyin"})
WriteResult({ "nInserted" : 1 })
# 再次查看数据库发现test库自动创建成功
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
melody  0.000GB
test    0.000GB
# 由于之前已经执行过use test, 这里直接show tables或者show collections就是查看test库下面的集合
> show tables
coll1
> show collections
coll1
# 查询coll1中的数据, 可以成功查询到刚刚插入的数据
> db.coll1.find()
{ "_id" : ObjectId("5fe1fdafa8e43518e25006b9"), "name" : "stevenyin" }
```
### 创建集合`db.createCollection(name, options)`
|参数|类型|描述|
|:-:|:-:|:-|
|`name`|`string`|集合名称|
|`options`|`document`|可选参数. 详见下表|

`options`中支持下列参数(仅部分常用字段):
|Field|Type|Description|
|:-:|:-:|:-|
|`capped`|`boolean`|可选参数. 如果为 `true`，则创建固定集合。固定集合是指有着固定大小的集合，当达到最大值时，它会自动覆盖最早的文档。当该值为 `true` 时，必须指定 `size` 参数。|
|`autoIndexId`|`boolean`| 如果为false, 代表取消自动为_id字段创建索引. 不过从MongoDB 4.0开始，在本地数据库以外的数据库中创建集合时，不能将选项autoIndexId设置为false。|
|`size`|`number`|可选参数. 指定上限集合的最大大小（以字节为单位）。 一旦上限集合达到最大大小，MongoDB就会删除较旧的文档以为新文档腾出空间。 如果想要这个字段生效, 则一定要`capped`是`true`, 否则无效|
|`max`|`number`|Optional. 固定大小集合(`capped`)中最大的文档数量. `size`选项的优先级比此参数更高, 如果指定了`size`参数则忽略当前参数. 如果固定集合再达到最大文档数量之前,先触发了size字段限制的大小, 则会开始删除旧文档. 如果一定要触发这个参数就要确保`size`的大小能够撑到触发`max`数量
```
> db.createCollection('test2', {'capped':true, 'size':2000000, 'max':1000});
{ "ok" : 1 }
> show collections
coll1
test2
```

### 删除集合
```
> show tables
learning
system.users
system.version
> db.learning.drop()
true
> show collections
system.users
system.version
```
