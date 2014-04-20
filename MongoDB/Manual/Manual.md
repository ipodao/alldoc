# MongoDB Manual 2.6

[toc]

## MongoDB CRUD 操作

### MongoDB CRUD 介绍

MongoDB documents are [BSON](http://docs.mongodb.org/manual/reference/glossary/#term-bson) documents, which is a binary representation of JSON with additional type information.

MongoDB stores all documents in collections. A collection is a group of related documents that have a set of shared common indexes. 集合相当于关系型数据库的表。

增删改数据可以作用域单个集合。

### MongoDB CRUD 概念

#### 读操作

##### 读操作概述

MongoDB中，查询指从单个集合中查询文档。

查询使用`db.collection.find()`方法。查询可以指定条件和投影。可以指定排序、limits 和 skips。

`db.collection.findOne()`只返回单个文档。

例子：

```javascript
db.users.find( { age: { $gt: 18 } }, { name: 1, address: 1 } ).limit(5)
```

查询的行为：
- 返回文档的顺序是不定的，除非指定了`sort()`。
- 更新操作的条件格式与查询的条件格式相同。
- 在 aggregation pipeline中，`$match` 阶段可以访问MongoDB的查询。

###### 投影

默认返回文档的所有字段。By projecting results with a subset of fields, applications reduce their network overhead and processing requirements.

> Except for excluding the `_id` field in inclusive projections, you cannot mix exclusive and inclusive projections.

投影的例子：

排除单个字段`history`
```javascript
db.records.find( { "user_id": { $lt: 42} }, { history: 0} )
```

返回两个字段和`_id`字段：
```javascript
db.records.find( { "user_id": { $lt: 42} }, { "name": 1, "email": 1} )
```

返回两个字段但包括`_id`字段：
```javascript
db.records.find( { "user_id": { $lt: 42} }, { "_id": 0, "name": 1 , "email": 1 } )
```

投影的行为：
- 除非显式排除，否则始终包含`_id`。
- 用于数组字段的投影运算符：`$elemMatch`, `$slice`, `$`.
- 在聚合pipeline的 *$project* 阶段可以使用投影。

##### 游标


在`mongo`命令行总，查询的主要方法`db.collection.find()`返回一个游标。如果不把这个游标赋给一个变量，游标将自动打印出最前面的20个文档。

> 若对20不满意，可以通过`DBQuery.shellBatchSize`改变迭代数量。See [Executing Queries](http://docs.mongodb.org/manual/tutorial/getting-started-with-the-mongo-shell/#mongo-shell-executing-queries) for more information.

To manually iterate the cursor to access the documents, see [Iterate a Cursor in the mongo Shell](http://docs.mongodb.org/manual/tutorial/iterate-a-cursor/).

###### 关闭不活动的游标

默认，当游标不活动10分钟，或客户端已经耗尽游标时，服务器将自动关闭游标。要改变这个行为，可以在查询时指定 *noTimeout* [wire protocol flag](http://docs.mongodb.org/meta-driver/latest/legacy/mongodb-wire-protocol) ；此时你需要手工关闭游标（或用尽它）。在`mongo` shell中，可以设置*noTimeout*标志：
```javascript
var myCursor = db.inventory.find().addOption(DBQuery.Option.noTimeout);
```

See your [driver](http://docs.mongodb.org/manual/applications/drivers/) documentation for information on setting the noTimeout flag. For the mongo shell, see [cursor.addOption()](http://docs.mongodb.org/manual/reference/method/cursor.addOption/#cursor.addOption) for a complete list of available cursor flags.

###### Cursor Isolation
Because the cursor is not isolated during its lifetime, intervening write operations on a document may result in a cursor that returns a document more than once if that document has changed. To handle this situation, see the information on [snapshot mode](http://docs.mongodb.org/manual/faq/developers/#faq-developers-isolate-cursors).

###### Cursor Batches

MongoDB 服务器批量返回查询结果。每批大小不会超过 [maximum BSON document size](http://docs.mongodb.org/manual/reference/limits/#limit-bson-document-size)。对于多数产需，首批返回101个文档或刚好超过1M的文档。后续每批大小4M。要覆盖批次大小，参见`batchSize()`和`limit()`。

如果查询需要排序但又没有索引，服务器需要将所有文档加载到内存中排序。于是第一个批次就会返回所有文档。

遍历游标时，当到达一个批次的尾部时，如果有更多结果，`cursor.next()`将触发[getmore](http://docs.mongodb.org/manual/reference/method/db.currentOp/#currentOp.op)操作，获取下一个批次。To see how many documents remain in the batch as you iterate the cursor, you can use the `objsLeftInBatch()` method, as in the following example:
```javascript
var myCursor = db.inventory.find();
var myFirstDocument = myCursor.hasNext() ? myCursor.next() : null;
myCursor.objsLeftInBatch();
```

##### 游标信息

可以利用 [cursorInfo](http://docs.mongodb.org/manual/reference/command/cursorInfo/#dbcmd.cursorInfo) 获取游标的信息：
- 已打开的游标总数
- 当前使用的客户端游标大小
- 自服务器启动以来，超时的游标个数

命令：
```javascript
db.runCommand( { cursorInfo: 1 } )
```

结果：
```javascript
{
  "totalOpen" : _number_,
  "clientCursors_size" : _number_,
  "timedOut" : _number_,
  "ok" : 1
}
```

##### 查询优化

索引可以提高读操作的性能。

例如，如果有查询`db.inventory.find( { type: typeValue } );`，可以建立索引：`db.inventory.ensureIndex( { type: 1 } )`。

###### 查询的选择性（Selectivity）

有些查询是不具选择性的（selective）。这些操作不能有效使用索引或完全无法使用索引。

不等操作`$nin`和`$ne`选择性不高。它们可能匹配索引中很大一部分。此时，使用索引与全表扫描相比，可能没有优势。

使用正则表达式的查询（内联Javascript正则表达式或`$regex`运算符），无法使用索引。但有一个例外：Queries that specify regular expression with anchors at the beginning of a string can use an index.

##### 索引覆盖查询

当：
- 查询中的所有字段都在索引中
- 匹配的文档的所有返回的字段在相同索引中。

加入集合`inventory`有一个索引：`{ type: 1, item: 1 }`。此索引能覆盖以下查询：
```javascript
db.inventory.find( { type: "food", item:/^c/ }, { item: 1, _id: 0 } )
```

注意，并不能覆盖下面的索引。由于`_id`也要求被返回：
```javascript
db.inventory.find( { type: "food", item:/^c/ }, { item: 1 } )
```

##### （未）查询计划

http://docs.mongodb.org/manual/core/query-plans/

The MongoDB query optimizer processes queries and chooses the most efficient query plan for a query given the available indexes. The query system then uses this query plan each time the query runs.

The query optimizer only caches the plans for those query shapes that can have more than one viable plan.

The query optimizer occasionally reevaluates query plans as the content of the collection changes to ensure optimal query plans. You can also specify which indexes the optimizer evaluates with *Index Filters*.

You can use the `explain()` method to view statistics about the query plan for a given query.

##### （未）分布式查询

http://docs.mongodb.org/manual/core/distributed-queries/

Sharded clusters allow you to partition a data set among a cluster of mongod instances in a way that is nearly transparent to the application.

#### 写操作



