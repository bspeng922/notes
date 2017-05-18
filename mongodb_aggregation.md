# Mongo Aggregation

MongoDB中聚合(aggregate)主要用于处理数据(诸如统计平均值,求和等)，并返回计算后的数据结果。

MongoDB中有三种聚合方法，每种有不同的适用场景和优点。

+ 聚合管道

聚合管道是一个基于数据处理管道概念的框架，它用于执行聚合任务。使用这个框架，MongoDB可以让一个集合中的多个文档通过一个管道。这个管道会将这组文档转化为聚合结果，你可以通过数据库的 aggregate 命令来访问这些结果。

+ 映射化简

映射化简是一种处理大量数据的，多阶段的泛化数据聚合的方法。MongoDB提供了 mapReduce 的数据库命令。

+ 单一用途的聚合

MongoDB提供了一些针对特定数据的聚合操作，以支持一些公共的数据聚合的功能。这些操作包含返回文档个数、对某个字段的值去重、和简单的归类操作。

curl -d '{"auth":{"passwordCredentials":{"username": "admin", "password": "123456"}}}' -H "Content-type: application/json" http://192.168.99.9:35357/v2.0/tokens

curl -X POST http://192.168.99.9:5000/v2.0/tokens -d '{"auth":{"passwordCredentials":{"username": "admin", "password":"123456"}}}' -H "Content-type: application/json"