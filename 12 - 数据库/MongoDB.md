[TOC]



# 基础概念

### 集合--collection

集合是MongoDB中的文档组，类似于传统关系型数据库中的表。

### 文档--document

一条数据就是一个文档，是一个数据结构，由字段和值构成键值对。

特性：

 * 文档是一组键值对，叫做BSON(类似于Json)
 * 同一个集合中，允许不用的文档拥有不同的字段(即数据结构不同)，这是区别于关系型数据库的
 * 同一个集合中，在不同的文档中，相同的字段也可以是不同的数据类型
 * 一个数据就是一个文档，与其他的文档独立，没有必然联系

### 字段--filed

字段就是文档中具体数据的字段

### 区别对比

|   SQL概念   | MongoDB概念 |       解释       |
| :---------: | :---------: | :--------------: |
|  database   |  database   |      数据库      |
|    table    | collection  |  数据库表/集合   |
|     row     |  document   |   一条完整数据   |
|   column    |    filed    |     字段/域      |
|    index    |    index    |       索引       |
| table joins |     --      |      表连接      |
| primary key | primary key | 主键(mongo为_id) |





# DB操作

*   显示所有数据库：show dbs
*   显示当前数据库对象或集合：db
*   切换到一个数据库，不存在则创建：use dbName
*   删除当前数据库：db.dropDatabase()





# 基本操作语法

### 创建集合

创建集合的语法为：db.createCollection(name, options)

options是创建集合时一些可选的属性

```shell
# 创建一个普通集合，不带任何属性
db.createCollection("ChatRecord")
# 创建一个集合，集合为固定集合，自动在_id字段创建索引，最大字节数为6142800，最多包含10000条数据
db.createCollection("mycol", {capped:true, autoIndexId:true, size:6142800, max:10000})
```



|    字段     | 类型 |                             描述                             |
| :---------: | :--: | :----------------------------------------------------------: |
|   capped    | 布尔 | true则创建固定集合，false创建普通集合(固定集合必须制定size参数) |
| autoIndexId | 布尔 |        true则自动在_id字段创建索引(3.2版本后不再支持)        |
|    size     | 数值 |               为集合指定一个最大值(按字节计算)               |
|     max     | 数值 |                 指定集合中包含文档的最大数量                 |



### 删除集合

```shell
db.collectionName.drop()
```



### 插入数据

##### insert插入

```shell
# 例子1：直接插入
db.collectionName.insert({name:"Zihao Jian",ID:"Uzi",club:"RNG"})
# 例子2：将数据定义成变量，然后插入
document = {name:"Zihao Jian",ID:"Uzi",club:"RNG"}
db.collectionName.insert(document)
```

##### save插入

语法与上面例子2相同，用save方法插入有如下两种情况：

*   指定_id字段，更新该\_id的数据
*   不指定_id字段，普通的插入数据



### 查询数据

```shell
# 查询MsgKey为Iverson-Kobe，并且side为2的数据
db.ChatRecord.find({"MsgKey":"Iverson-Kobe", "Side":2})

# 查询MsgKey以Iverson开头并且side为2，或者，以Iverson结尾并且side为1的数据
db.ChatRecord.find({$or:[{"MsgKey":{"$regex":"^Iverson"},"Side":2},{"MsgKey":{"$regex":"Iverson$"},"Side":1}]}).pretty()
```

##### 查询条件

*   如果只有多个查询条件需要同时满足，则每个之间用 "," 隔开就好
*   如果只需满足其中一个，则需要用到 $or，像上面的例子那样

##### 查询比较

*   等于直接用 ":"即可：db.CharRecord.find( {"MsgID" : 6} )
*   小于：$lt -------------- db.CharRecord.find( {"MsgID" : {$lt : 6} } )
*   小于等于：$lte ------ db.CharRecord.find( {"MsgID" : $lte : 6} )
*   大于：$gt ------------- db.CharRecord.find( {"MsgID" : $gt : 6} )
*   大于等于：$gte ----- db.CharRecord.find( {"MsgID" : $gte : 6} )
*   不等于：$nt ---------- db.CharRecord.find( {"MsgID" : $n: 6} )

##### 字符串比较

*   等于：直接用 ":"即可
*   模糊查询：用正则表达式 $regex，像上述例子一样



### 更新数据

##### 基本语法

db.*collectionName*.update(*query*, *update*, {*option*...})

*   query：update的查询条件，类似于sql中的where
*   update：更新的内容

options属性：

*   upsert：表示query查询没有相应的记录，是否要插入，默认为false
*   multi：表示query查询到多条记录，是更新一条数据还是多条，默认为false
*   writeConcern：抛出异常的级别

##### 数值类型更新的操作类型

数值类型，可以设置一个新的值，也可以在原来值的基础上累加

*   $set：设置新值
*   $inc：在原来值的基础上累加



### 聚合

```shell
db.ChatRecord.aggregate({$group:{_id:{"MsgKey":"$MsgKey"} , "MsgKey":{"$last":"$MsgKey"}}})
```



### 字符串拼接

```shell
# 查找MsgKey的值为Iverson-Kobe，在后面追加字符串
db.ChatRecord.update({"MsgKey":"Iverson-Kobe"},[{$set:{"MsgKey":{$concat:["$MsgKey","Bryant"]}}}])
```





# 数组

### 插入语法

```shell
users={"UserID":"James", "UserNick":"chossen-one", "UserHead":"", "BlockList":[{"UserID":"Pierce"},{"UserID":"West"},{"UserID":"Green"}]}
db.UserInfo.insert(users)
```

### 追加数组元素

```shell
# $push: 不管数据中是否已经有此元素，直接追加
db.UserInfo.update({"UserID":"James"},{$push:{"BlockList":{"UserID":"Wade"}}})
# $addToSet: 若数据中已经有此元素，则不追加；没有才会追加
db.UserInfo.update({"UserID":"James"},{$addToSet:{"BlockList":{"UserID":"Wade"}}})
```

### 删除数组元素

```shell
# pop删除数组的第一个和最后一个元素，-1表示第一个，1表示最后一个
db.UserInfo.update({},{$pop:{"BlockList":1}})
# pull删除指定元素的数组
db.UserInfo.update({},{$pull:{"BlockList":{"UserID":"Wade"}}})
```



# 固定集合

### 创建固定集合

```shell
# 普通创建
db.createCollection("collectionName",{capped:true,size:10000})
# 设置最大文档个数为1000
db.createCollection("collectionName",{capped:true,size:10000,max:1000})
```

### 判断、设置固定集合

```shell
# 判断是否为固定集合
db.collectionName.isCapped()
# 将集合转换成固定集合
db.runCommand({"convertToCapped":"collectionName",size:10000})
```

### 固定和特点及属性

*   固定集合文档是按照插入顺序存储的，默认情况下查询也按插入顺序返回（也可以用$natural调整法返回顺序）
*   可以插入及更新，但是不允许删除，要想删除只能用drop()删除
*   对固定集合进行插入速度极快
*   按照插入顺序的查询输出速度极快
*   能够在插入最新数据时，淘汰最早的数据