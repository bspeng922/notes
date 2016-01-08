# MongoDB学习笔记 - day2


## 关系

> Relationships in MongoDB represent how various documents are logically related to each other. Relationships can be modeled via Embedded and Referencedapproaches. Such relationships can be either 1:1, 1: N, N: 1 or N: N.


### 嵌入式关系（ Embedded Relationships）

比如有一个用户document，一个地址document
```
# user
{
   "_id":ObjectId("52ffc33cd85242f436000001"),
   "name": "Tom Hanks",
   "contact": "987654321",
   "dob": "01-01-1991"
}

#　address
{
   "_id":ObjectId("52ffc4a5d85242602e000000"),
   "building": "22 A, Indiana Apt",
   "pincode": 123456,
   "city": "Los Angeles",
   "state": "California"
} 
```

那么嵌入式关系的形式如下：
```
{
   "_id":ObjectId("52ffc33cd85242f436000001"),
   "contact": "987654321",
   "dob": "01-01-1991",
   "name": "Tom Benzamin",
   "address": [
      {
         "building": "22 A, Indiana Apt",
         "pincode": 123456,
         "city": "Los Angeles",
         "state": "California"
      },
      {
         "building": "170 A, Acropolis Apt",
         "pincode": 456789,
         "city": "Chicago",
         "state": "Illinois"
      }
   ]
}  
```


### 引用关系（Referenced  Relationships）

引用关系其实就是将嵌入式关系中对应的值换成相应的document对象。

```
{
   "_id":ObjectId("52ffc33cd85242f436000001"),
   "contact": "987654321",
   "dob": "01-01-1991",
   "name": "Tom Benzamin",
   "address_ids": [
      ObjectId("52ffc4a5d85242602e000000"),
      ObjectId("52ffc4a5d85242602e000001")
   ]
}
```



## 数据库引用（Database  References）

数据库引用有三个字段

> $ref: This field specifies the collection of the referenced document
> 
> $id: This field specifies the _id field of the referenced document
> 
> $db: This is an optional field and contains name of the database in which the referenced document lies

```
# user
{
   "_id":ObjectId("53402597d852426020000002"),
   "address": {
   "$ref": "address_home",
   "$id": ObjectId("534009e4d852427820000002"),
   "$db": "tutorialspoint"},
   "contact": "987654321",
   "dob": "01-01-1991",
   "name": "Tom Benzamin"
}
```

tutorialspoint 为另一个db，ref为collection， id为对应的document记录

```
# tutorialspoint.address_home
{
   "_id" : ObjectId("534009e4d852427820000002"),
   "building" : "22 A, Indiana Apt",
   "pincode" : 123456,
   "city" : "Los Angeles",
   "state" : "California"
}
```



## 覆盖查询（Covered  Query）

> As per the official MongoDB documentation, a covered query is a query in which:
> 
>   + call the fields in the query are part of an index and
>   + all the fields returned in the query are in the same index

覆盖查询的条件是传入的查询参数和返回的参数都在index中，查询的时候并不会涉及到collection的查询，所以覆盖查询的速度是非常快的。

```
# create index
db.users.ensureIndex({gender:1,user_name:1})

# query from index
db.users.find({gender:"M"},{user_name:1,_id:0})
```

> remember that an index cannot cover a query if:
> 
>    + any of the indexed fields is an array
>    + any of the indexed fields is a subdocument



## 分析查询（Analyzing  Queries）

分析查询主要包括 $explain 和 $hint 两种 , explain 主要是对查询过程进行分析，比如扫描了多少document，匹配了多少等等

> The $explain operator provides information on the query, indexes used in a query and other statistics. It is very useful when analyzing how well your indexes are optimized.

> The $hint operator forces the query optimizer to use the specified index to run a query. This is particularly useful when you want to test performance of a query with different indexes. 



## 原子操作（Atomic  Operations）

> MongoDB does not support multi-document atomic transactions. However, it does provide atomic operations on a single document. 

```
> db.products.findAndModify({ 
   query:{_id:2,product_available:{$gt:0}}, 
   update:{ 
      $inc:{product_available:-1}, 
      $push:{product_bought_by:{customer:"rob",date:"9-Jan-2014"}} 
   }    
})
```

| 修改器 | 作用 |
| ----- |:------:| 
| $inc | 相当于编程语言中的 “+=”, $inc修改器不能应用于非数字数据|
| $set | 用来指定一个键并更新键值，若键不存在并创建| 
| $unset | 主要是用来删除键| 
| $push|  数组修改器| 
| $ne/$addToSet | 数组修改器，主要给数组类型键值添加一个元素时，避免在数组中产生重复数据| 
| $pop | 从数组的头或者尾删除数组中的元素| 
| $pull | 从数组中删除满足条件的元素| 



## 高级索引（Advanced  Indexing）

### 索引数组字段

> Creating an index on array in turn creates separate index entries for each of its fields.

```
# user
{
   "address": {
      "city": "Los Angeles",
      "state": "California",
      "pincode": "123"
   },
   "tags": [
      "music",
      "cricket",
      "blogs"
   ],
   "name": "Tom Benzamin"
}
```

比如我我们想根据tag来查询用户，那我们需要创建tags的索引，但实际上mongo会根据tags里面的值分别创建索引，所以我们可以根据下面的命令来查询用户

```
> db.users.ensureIndex({"tags":1})
> db.users.find({tags:"cricket"})
```


### 索引子document

假如我们想根据city来查询用户，但是city是user的子document address里面的字段，这时我们就可以通过创建子document的方法来实现。

```
> db.users.ensureIndex({"address.city":1,"address.state":1,"address.pincode":1})
> db.users.find({"address.city":"Los Angeles"})
```



## 索引限制（Indexing  Limitations）

> Every index occupies some space as well as causes an overhead on each insert, update and delete. So if you rarely use your collection for read operations, it makes sense not to use indexes.
> 
> Since indexes are stored in RAM, you should make sure that the total size of the index does not exceed the RAM limit. If the total size increases the RAM size, it will start deleting some indexes and hence causing performance loss.
> 
> Maximum Ranges:
> 
>    + A collection cannot have more than 64 indexes.
>    + The length of the index name cannot be longer than 125 characters
>    + A compound index can have maximum 31 fields indexed



## 对象ID（ObjectId）

对象ID的组成： 4位秒数 + 3位机器识别 + 2位进程ID + 3位随机数

> MongoDB uses ObjectIds as the default value of _id field of each document which is generated while creation of any document. The complex combination of ObjectId makes all the _id fields unique.

因为每个对象ID都有时间戳位，所以你不需要添加创建时间字段，可以直接从对象ID中获取
```
> ObjectId("5349b4ddd2781d08c09890f4").getTimestamp()
```



## Map Reduce

> As per the MongoDB documentation, Map-reduce is a data processing paradigm for condensing large volumes of data into useful aggregated results. MongoDB uses mapReduce command for map-reduce operations. MapReduce is generally used for processing large data sets.

to be continued...



## 文本搜索（Text  Search）

2.6之后的版本默认启用文本搜索，之前的需要输入命令启用
```
> db.adminCommand({setParameter:true,textSearchEnabled:true})
```

假如有一个post collection，想要搜索 post_text 中包含 tutorialspoint 的 document
```
# post collection
{
   "post_text": "enjoy the mongodb articles on tutorialspoint",
   "tags": [
      "mongodb",
      "tutorialspoint"
   ]
}
> db.posts.ensureIndex({post_text:"text"})

> db.posts.find({$text:{$search:"tutorialspoint"}})
```



## 正则表达式（Regular  Expression）

> MongoDB also provides functionality of regular expression for string pattern matching using the $regex operator. MongoDB uses PCRE (Perl Compatible Regular Expression) as regular expression language.

比如上面的文本搜索也可以使用正则表达式来实现

```
> db.posts.find({post_text:{$regex:"tutorialspoint"}})
```

正则表达式也支持数组字段

```
# user
{
   "tags": [
      "music",
      "cricket",
      "blogs"
   ],
   "name": "Tom Benzamin"
}

> db.posts.find({tags:{$regex:"tutorial"}})
```



## GridFS

> GridFS is the MongoDB specification for storing and retrieving large files such as images, audio files, video files, etc. It is kind of a file system to store files but its data is stored within MongoDB collections. GridFS has the capability to store files even greater than its document size limit of 16MB.

GridFS会将文件分片并存储到不通的document中，分片大小最大不超过255k

GridFS默认有 **fs.files** 和 **fs.chunks** 两个collection，分别用来存储文件的 metadata，和分片（chunks）, fs.chunks 中的 files_id 字段即对应 fs.files 中相应文件的_id

```
# fs.files
{
   _id: ObjectId('534a811bf8b4aa4d33fdf94d'), 
   filename: "song.mp3", 
   chunkSize: 261120, 
   uploadDate: new Date(1397391643474), md5: "e4f53379c909f7bed2e9d631e15c1c41",
   length: 10401959 
}

# fs.chunks
{
   "files_id": ObjectId("534a811bf8b4aa4d33fdf94d"),
   "n": NumberInt(0),
   "data": "Mongo Binary Data"
}
```



## 自增序列（Auto-Increment Sequence）

> MongoDB does not have out-of-the-box auto-increment functionality like SQL databases. By default, it uses the 12-byte ObjectId for the _id field as primary key to uniquely identify the documents. However, there may be scenarios where we may want the _id field to have some auto-incremented value other than the ObjectId.
>
>Since this is not a default feature in MongoDB, we will programmatically achieve this functionality by using a counters collection as suggested by the MongoDB documentation.

自增序列的原理就是在每次创建document的时候，将创建的序列+1保存到序列collection中，这样再次创建document的时候，只需要到序列collection中取一下这个值就可以了。不过这种方法涉及到js函数，比较复杂。


