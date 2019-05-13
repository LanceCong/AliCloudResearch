[TOC]

# 第2章 MongoDB基础知识

## 2.1 文档
* MongoDB区分大小写，文档不能有相同的key
* \0是结束的标志，不能作为集合名或者key的名

## 2.2 集合
### 2.2.2 命名
* .和$是保留字
* 子集合，用 . 分隔

## 2.5 MongoDB shell简介
### 2.5.1 运行shell
### 2.5.2 MongoDB客户端
* db 查看所用数据库
* show dbs 查看数据库
* use db 使用某个数据库
* show collections 查看集合

### 2.5.3 shell种的基本操作
insert 创建

```js
post = {
"title": "My Blog Post",
"content": "Here's my blog post",
"date": new Date()
}

db.blog.insert(post);

```

update 更新

```js
post = {
"title": "My Blog Post",
"content": "Here's my blog post",
"date": new Date()
}
 post.comments = [];

db.blog.update({title: "My Blog Post"}, post);

```

remove 删除
```js
db.blog.remove(query) //没有query会全部删除
```

## 2.6 数据类型
### 2.6.1 基本数据类型
* null  
null 用于表示空值或者不存在的字段
```js{"x" : null}```
* boolean  
```{"x" : true}```
* number  
The shell defaults to using 64-bit floating point numbers. Thus, these numbers look
“normal” in the shell:
```js
{"x" : 3.14}
//or:
{"x" : 3}
```
For integers, use the NumberInt or NumberLong classes, which represent 4-byte or
8-byte signed integers, respectively.
```js
{"x" : NumberInt("3")}
{"x" : NumberLong("3")}
```
* string  
Any string of UTF-8 characters can be represented using the string type:
```{"x" : "foobar"}```
* date  
Dates are stored as milliseconds since the epoch. The time zone is not stored:
```{"x" : new Date()}```
* regular expression  
Queries can use regular expressions using JavaScript’s regular expression syntax:
```{"x" : /foobar/i}```
* array  
Sets or lists of values can be represented as arrays:
```{"x" : ["a", "b", "c"]}```
* embedded document 内嵌文档  
Documents can contain entire documents embedded as values in a parent
document:
```{"x" : {"foo" : "bar"}}```  
Data Types | 17
Download from Wow! eBook <www.wowebook.com>
* object id  
An object id is a 12-byte ID for documents. See the section “_id and ObjectIds” on
page 20 for details:  
```{"x" : ObjectId()}```  
There are also a few less common types that you may need, including:
* binary data 二进制数据  
Binary data is a string of arbitrary bytes. It cannot be manipulated from the shell.
Binary data is the only way to save non-UTF-8 strings to the database.
* code 代码  
Queries and documents can also contain arbitrary JavaScript code:  
```{"x" : function() { /* ... */ }}```  
There are a few types that are mostly used internally (or superseded by other types).
These will be described in the text as needed.
For more information on MongoDB’s data format, see Appendix B.

### 2.6.2 日期

### 2.6.3 数组
查询的时候，可以查询数组里的某个元素  
存在users:['1261651','321321354']  
例如query={users: '15915154578'}  

### 2.6.4 内嵌文档
### 2.6.5 _id和ObjectId
* _id 在集合中必须唯一，是ObjectId对象
* 创建文档没有_id键的时候系统会帮你创建一个

## 2.7 使用MongoDB shell
### 2.7.4 定制shell提示

# 第3章 创建、更新和删除文档
## 3.1插入并保存文档
```> db.foo.insert({bar:"barz"})```

### 3.1.1 批量插入
* batchInsert
```
> db.foo.batchInsert([{},{},{}])
```
* continueOnError 选项

### 3.1.2 插入校验
* 16M 当前文档最大值
* 容易插入非法值

## 3.2删除文档
* remove(query)
* 不能撤销，不能恢复
```
//插入100万条数据
for(var i = 0; i < 1000000;i++){
 db.rms.insert({foo:'bar',users:[],create_time:new Date()});
}
//删除
var timeRemoves = function() {
 var start = (new Date()).getTime();
 db.rms.remove();
 db.findOne(); // makes sure the remove finishes before continuing

  var timeDiff = (new Date()).getTime() - start;
  print("Remove took: "+timeDiff+"ms");
  }
  
> timeRemoves()
```

## 3.3 更新文档
update至少要2个参数，一个定位，一个是修改器modifier，用于说明要对找到的文档进行哪些修改。
### 3.3.1 文档替换
db.col.update(query,newdata)
### 3.3.2 使用修改器
部分文档需要更新的时候。  
更新修改器 update modifier 是种特殊的键。  
  
#### 1."$set" 修改器入门
* "$set"用来指定一个字段的值。如果这个字段不存在，则创建它。
``` db.col.update(query,{"$set": {"key":"value"}) ```  
* 也可以可以更新字段的类型，例如把String字段改成数组。
``` db.col.update(query,{"$set": {"key":["",""]}}) ```  
* 用 ```"$unset"```将键完全删除：  
``` db.col.update(query,{"$set":{"key":1}}) ```  
* 用"$set"修改内嵌文档  
``` db.col.update(query,{"$set":{"key.subkey":"value"}) ```

#### 2."$inc" 增加或减少
给一个人加50分，如果不存在score键会创建  
``` db.games.findOne(query,{"$inc": {"score": 50}}) ```  
#### 3. 数组修改器

#### 4. 添加元素
用"$push"给数组末尾添加一个元素，如果不存在该数组，会创建再添加。  
特别用法：
* 使用 "$each" 子操作符，用 "$push" 一次添加多个元素
``` 
db.man.update(query,{"$push": {"books":{
    "$each":["书名1","书名2","书名3"]
}}}) 
```
* 保证数组不会超出设定的最大长度。
```
db.device.update(query,{"$push":{
    "users": {
        "$each": ["lance","alice"]
        ,"$slice": -10
    }
}})
```
以上例子，保证了一个设备最多10个最新添加的用户

#### 5.将数组作为数据集使用
保证数组内的元素不会重复，使用 $ne 或 $addToSet  
例如，如果作者不在引文列表中，就添加进去  
```
db.papers.update({"authors cited": {"$ne":"Richie"}},
    {"$push": {"authors cited": "Richie"}}
)
```
$addToSet 也可以实现  
```db.papers.update(query,{"$addToSet": {"emails": "654789123@qq.com"}})```  
如果要添加多个  
```
db.papers.update(query,{"$addToSet": 
{"emails":{"$each":["654789123@qq.com","123456789@qq.com"]}}})
```  

#### 6.删除元素 "$pop" "$pull"
```{"$pop":{"key":1}}```从数组末尾删除一个元素  
```{"$pop":{"key":-1}}```从数组头部删除一个元素  
删除制定元素  
```db.lists.insert({"todo": ["dishes", "laundry", "dry cleaning"]})```  
完成了洗衣服laundry，删除它  
```db.lists.update({},{"$pull": {"todo": "laundry"}})``` 通过查找，就只剩下2个元素  
```
db.lists.find().pretty()
>{
    "todo": ["dishes", "dry cleaning"]
}
```
注意如果数组中有重复的匹配元素，$pull会把它们都删除。  

#### 7.基于位置的数组修改器  
例如有一个post，包含多个评论  
```
> db.blog.posts.findOne()
{
    "_id" : ObjectId("4b329a216cc613d5ee930192"),
Updating Documents | 41
    "content" : "...",
    "comments" : [
        {
            "comment" : "good post",
            "author" : "John",
            "votes" : 0
        },
        {
            "comment" : "i thought it was too short",
            "author" : "Claire",
            "votes" : 3
        },
        {
            "comment" : "free watches",
            "author" : "Alice",
            "votes" : -1
        }
    ]
}
```
如果要增加一个评论的点赞数，可以这么做  
``` db.blog.update({"post" : post_id},{"$inc" : {"comments.0.votes" : 1}})```  
如果不想先查文档，可以用定位操作符$
``` db.blog.update({"comments.author" : "John"}, {"$set" : {"comments.$.author" : "Jim"}})```  
它只能更新第一个。

#### 8.修改器的速度
文档存的时候是相邻的。  
创建一个集合，插入几个文档，然后修改中间的，让它变大，会发现它会跑到最后去。

### 3.3.3 upsert 特殊的更新
upsert 是一个选项，是特殊的更新。如果没有找到符合更新条件的文档，就会以这个条件和更新文档为基础，创建一个新的文档。同一套代码可以用于创建文档，又可以更新文档。  
* update最后一个参数就是upsert  
eg:``` db.users.update({"req": 25}, {"$inc": {"rep":3}}, true) ```
* $setOnInsert    当update方法使用upsert选项执行insert操作时，$setOnInsert操作符给相应的字段赋值  
eg:```db.users.update(query,{"$setOnInsert": {"createAt": new Date()}}, true) ```

### 3.3.4 更新多个文档
默认情况下，更新只能对符合匹配条件的第一个文档执行操作。多个符合条件，只会修改第一个。如果要更新多个，把update的第4个参数设置为true。  

### 3.3.5 返回被更新的文档
可以通过findAndModify命令得到被跟新的文档。  
返回的文档类似  
```
{
    "OK": 1,
    "value": {
        "_id": ObjectId("45s6d4f65sd4f6a5sd4f")
        ,"priority": 1
        ,"status": "READY"
    }
}
```
操作示例，修改一个过程，操作完成之后，改成完成状态
```
ps = db.runCommand({"findAndModify": "processes",
"query": {"status": "READY"},
"sort": {"priority": -1},
"update": {"$set": {"status": "RUNNING"}}
    
}).value

do_something(ps)
db.process.update({"_id": ps._id},{"$set": {"status": "DONE"}})
```
findAndModify 命令有很多可以使用的字段。
* findAndModify  
  字符串，集合名
* query  
  查询文档，检索条件
* sort  
  排序条件
* update  
  修改器文档，用于对匹配的文档进行更新（update和remove必须指定一个）。
* remove
  布尔类型，表示是否删除文档。
* new
  布尔类型，表示返回更新前的文档还是更新后的文档。默认是更新前的。
* fields
  文档中需要返回的字段（可选）
* upsert
  布尔类型

## 写入安全机制

# 第4章 查询
## 4.1 find简介

### 4.1.1 指定返回的键
1是返回，0不返回，可以指定  
db.col.find({},{"_id":0})

## 4.2 查询条件

### 4.2.1 查询条件
* $lt 小于
* $lte 小于等于
* $gt 大于
* $gte 大于等于

### 4.2.2 OR查询
* $in 指定范围
``` 
db.col.find({"birthday":{
    "$in":["1991","1992","1993"]    
}}) 
//相反的条件是$nin
eg:略
```
* $or 多个条件
```
db.col.find({"$or":[query1,])
```
### 4.2.3 $not
### 4.2.4 条件句的规则

## 4.3 特定于类型的查询
### 4.3.1 null



## 4.4 $where查询

## 4.5 游标

## 4.6 游标内幕












# 第9章 创建副本集（replica set）
* 副本集的概念
* 副本集d创建方法
* 副本集成员的可用选项

## 9.1 复制简介
一个mongod服务进程会不安全，不适合生产环境。应当使用复制功能，将数据副本保存到多台服务器上。  

## 9.2 建立副本集
使用mongo --nodb 启动一个shell  
通过以下命令创建一个副本集：  
``` replicaSet = new ReplSetTest({"nodes":3}) ```  
创建了包含3个服务器的副本集：一个主服务器和两个备份服务器。  
执行下面命令，启动它们：  
``` 
//启动3个mongod进程
replicaSet.startSet()
//配置复制功能
replicaSet.initiate()
```
可以看到3个mongod进程，分别运行在3个端口。  
连接到一个进程  
```
conn1 = new Mongo("1270.0.01:20000")
primaryDB = conn1.getDB("test")
//查看是否master
> primaryDB.isMaster()
{
	"hosts" : [
		"localhost.localdomain:20000",
		"localhost.localdomain:20001",
		"localhost.localdomain:20002"
	],
	"setName" : "testReplSet",
	"setVersion" : 1,
	"ismaster" : true,
	"secondary" : false,
	"primary" : "localhost.localdomain:20000",
	"me" : "localhost.localdomain:20000",
	"electionId" : ObjectId("7fffffff0000000000000001"),
	"maxBsonObjectSize" : 16777216,
	"maxMessageSizeBytes" : 48000000,
	"maxWriteBatchSize" : 1000,
	"localTime" : ISODate("2017-01-17T02:15:14.189Z"),
	"maxWireVersion" : 4,
	"minWireVersion" : 0,
	"ok" : 1
}

//插入1000条数据
> for (var i = 0;i < 1000;i++){primaryDB.coll.insert({count:i})}
WriteResult({ "nInserted" : 1 })
> primaryDB.coll.count()
1000
> conn2 = new Mongo("127.0.0.1:20001")
connection to 127.0.0.1:20001
> secondaryDB = conn2.getDB("test")
test
> sendaryDB.coll.find()
2017-01-17T10:20:07.414+0800 E QUERY    [thread1] ReferenceError: sendaryDB is not defined :
@(shell):1:1

> secondaryDB.coll.find()
Error: error: { "$err" : "not master and slaveOk=false", "code" : 13435 }

//需要设置一下，才可以查看
> conn2.setSlaveOk()
> secondaryDB.coll.find()
{ "_id" : ObjectId("587d7edd2a05a4a452c2e1c4"), "count" : 0 }
{ "_id" : ObjectId("587d7edd2a05a4a452c2e1c6"), "count" : 2 }
{ "_id" : ObjectId("587d7edd2a05a4a452c2e1c5"), "count" : 1 }
{ "_id" : ObjectId("587d7edd2a05a4a452c2e1db"), "count" : 23 }
{ "_id" : ObjectId("587d7edd2a05a4a452c2e1de"), "count" : 26 }
{ "_id" : ObjectId("587d7edd2a05a4a452c2e1e7"), "count" : 35 }
{ "_id" : ObjectId("587d7edd2a05a4a452c2e1f4"), "count" : 48 }
{ "_id" : ObjectId("587d7edd2a05a4a452c2e1fa"), "count" : 54 }
{ "_id" : ObjectId("587d7edd2a05a4a452c2e1d7"), "count" : 19 }
{ "_id" : ObjectId("587d7edd2a05a4a452c2e1ff"), "count" : 59 }
{ "_id" : ObjectId("587d7edd2a05a4a452c2e208"), "count" : 68 }
{ "_id" : ObjectId("587d7edd2a05a4a452c2e210"), "count" : 76 }
{ "_id" : ObjectId("587d7edd2a05a4a452c2e232"), "count" : 110 }
{ "_id" : ObjectId("587d7edd2a05a4a452c2e23d"), "count" : 121 }
{ "_id" : ObjectId("587d7edd2a05a4a452c2e1da"), "count" : 22 }
{ "_id" : ObjectId("587d7edd2a05a4a452c2e23e"), "count" : 122 }
{ "_id" : ObjectId("587d7edd2a05a4a452c2e1e8"), "count" : 36 }
{ "_id" : ObjectId("587d7edd2a05a4a452c2e1f7"), "count" : 51 }
{ "_id" : ObjectId("587d7edd2a05a4a452c2e242"), "count" : 126 }
{ "_id" : ObjectId("587d7edd2a05a4a452c2e247"), "count" : 131 }
Type "it" for more
> secondaryDB.coll.count()
1000

//手动出异常
> primaryDB.adminCommand({"shutdown":1})
//可以看到master就变了
> secondaryDB.isMaster()
{
	"hosts" : [
		"localhost.localdomain:20000",
		"localhost.localdomain:20001",
		"localhost.localdomain:20002"
	],
	"setName" : "testReplSet",
	"setVersion" : 1,
	"ismaster" : false,
	"secondary" : true,
	"primary" : "localhost.localdomain:20002",
	"me" : "localhost.localdomain:20001",
	"maxBsonObjectSize" : 16777216,
	"maxMessageSizeBytes" : 48000000,
	"maxWriteBatchSize" : 1000,
	"localTime" : ISODate("2017-01-17T02:23:24.682Z"),
	"maxWireVersion" : 4,
	"minWireVersion" : 0,
	"ok" : 1
}
> 

```


## 9.3 配置副本集
* 生成一个base64串的keyFile，并设置权限  
``` chmod 600 keyFile ```
* 配置文件mongodb.conf
```
port = 20000
dbpath = /data/db
fork = true
replSet = spock
logpath = /data/logs
logappend = true
# auth = true
keyFile = /opt/software/mongodb/keyFile
```
* 所有服务器运行，启动mongod进程  
``` mongod -f mongodb.conf ```
* 设置副本集的config
```
config = {
    "_id" : "spock",
    "members" : [
        {"_id" : 0, "host" : "ip1:20000"},
        {"_id" : 1, "host" : "ip2:20000"},
        {"_id" : 2, "host" : "ip3:20000"}
    ]
}
//记得把上面的端口都开了
```

* 添加用户
```
//添加管理员用户。
//添加之后，需要切换到这个用户才能操作rs
use admin
db.createUser(
    {
      user: "admin",
      pwd: "niot1235",
      roles: [
         { role: "clusterManager", db: "admin" },
         { role: "userAdminAnyDatabase", db: "admin" }
      ]
    }
)
//先认证到admin数据库后，添加普通数据库用户
//添加具体数据库用户
use test
db.createUser(
  {
    user: "test",
    pwd: "niot1235",
    roles: ["readWrite", "dbAdmin"]
  }
)

```

* 选一个服务器，配置副本集
```
rs.initconfig(config)
//重新配置
rs.reconfig(config)
```

* 认证用户
```
db.auth('username','pwd')
//如果提示没认证，先db.auth('admin','')认证，再添加一个用户。
```

### 9.3.1 rs辅助函数

### 9.3.2 网络注意事项

## 9.4 修改副本集配置

## 9.5 设计副本集

## 9.6 成员配置选项
### 9.6.1 选举仲裁者

### 9.6.2 优先级

### 9.6.3 隐藏成员

### 9.6.4 延迟备份节点

### 9.6.5 创建索引




























