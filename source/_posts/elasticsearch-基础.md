---
title: elasticsearch 基础
date: 2021-08-05 10:37:04
tags: elasticsearch 
---


# 简单概念 含义

## 关于 Index、Type、Document

### 含义
Index：索引。复数是 Indices。
Type：类型。
Document：文档。文档是 JSON 类型的。

### 与 MySQL 类比
可以将 ES 中的这三个概念和 MySQL 类比：

 - Index 对应 MySQL 中的 Database；
 - Type 对应 MySQL 中的 Table；(在7.0后面的版本中已经废弃)
 - Document 对应 MySQL 中表的记录。

一个MySQL实例中可以创建多个 Database，一个Database中可以创建多个Table。

### 从 ES 7.0 开始，Type 被废弃

在 7.0 以及之后的版本中 Type 被废弃了。一个 index 中只有一个默认的 type，即 _doc。

ES 的Type 被废弃后，库表合一，Index 既可以被认为对应 MySQL 的 Database，也可以认为对应 table。

也可以这样理解：

- ES 实例：对应 MySQL 实例中的一个 Database。
- Index 对应 MySQL 中的 Table 。
- Document 对应 MySQL 中表的记录。

### 分片 副本

创建名为movie 的索引
```
PUT movie 
```

查询索引信息

```
GET movie

# 响应
{
  "movie" : {
    "aliases" : { },
    "mappings" : { },
    "settings" : {
      "index" : {
        "creation_date" : "1572843447230",
        "number_of_shards" : "1",
        "number_of_replicas" : "1",
        "uuid" : "AzEtRiBaQqO5JDN_l7WG-w",
        "version" : {
          "created" : "7020099"
        },
        "provided_name" : "movie"
      }
    }
  }
}
```

可以看到，分片数量number_of_shards 为1，副本数量为number_of_replicas 1 。


### 数据类型

详情参考官方文档：https://www.elastic.co/guide/en/elasticsearch/reference/7.2/mapping-types.html。


#### 字符串类型 keyword 、text

在 ElasticSearch 5.0 之前，字符串类型是 string。从 5.0 版本开始，string 类型被废弃，引入了 keyword 、text 两种类型。

##### 两者的主要区别是：

- keyword 不支持全文搜索。所以，只能是使用精确匹配进行查询，比如 term 查询。
- text 默认支持全文搜索。


##### 长度区别：

- keyword 的最长长度是 32766 字节。（原因应该是底层lucene做倒排索引时，限制了单词的长度。UTF-8中，英文字母是1个字节，中文一般是3个字节，表情符号是4个字节）
- text 无长度限制。（但被分析器处理后的单词不应该超过 32766 个字节。-> 这是我推论出来的，待验证）。




### 创建和删除索引
#### 创建索引

```
PUT movie
```

响应

```
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "movie"
}
```


查看索引信息

```
GET movie
```

响应

```
{
  "movie" : {
    "aliases" : { },
    "mappings" : { },
    "settings" : {
      "index" : {
        "creation_date" : "1628152862791",
        "number_of_shards" : "1",
        "number_of_replicas" : "1",
        "uuid" : "rE8l-P_uSqufXyuLUgu1xA",
        "version" : {
          "created" : "7030099"
        },
        "provided_name" : "movie"
      }
    }
  }
}
```


#### 创建索引时指定 mapping 和 settings
```
PUT movie
{
  "mappings" : {
    "properties" : {
      "name" : {
        "type" : "keyword"
      }
    }
  },
  "settings" : {
    "index" : {
      "number_of_shards" : 1,
      "number_of_replicas" : 2
    }
  }
}
```

#### 创建文档时自动创建索引

当向一个不存在的索引中写入文档时，会自动创建索引。

```
POST student/_doc/1
{
  "name": "张三"
}
```

查询创建的索引
```
GET student

响应：
{
  "student" : {
    "aliases" : { },
    "mappings" : {
      "properties" : {
        "name" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }
      }
    },
    "settings" : {
      "index" : {
        "creation_date" : "1628153905287",
        "number_of_shards" : "1",
        "number_of_replicas" : "1",
        "uuid" : "tAXU8LmSQpOo-ktpmsdMJw",
        "version" : {
          "created" : "7030099"
        },
        "provided_name" : "student"
      }
    }
  }
}

```

#### 删除索引
```
DELETE movie
```

响应
```
{
  "acknowledged" : true
}
```
如果再删除之后,再次删除,会因为找不到索引而报错

```
{
  "error" : {
    "root_cause" : [
      {
        "type" : "index_not_found_exception",
        "reason" : "no such index [movie]",
        "resource.type" : "index_or_alias",
        "resource.id" : "movie",
        "index_uuid" : "_na_",
        "index" : "movie"
      }
    ],
    "type" : "index_not_found_exception",
    "reason" : "no such index [movie]",
    "resource.type" : "index_or_alias",
    "resource.id" : "movie",
    "index_uuid" : "_na_",
    "index" : "movie"
  },
  "status" : 404
}
```

删除后再次使用 GET movie 查询索引信息，会报错：
```
{
  "error" : {
    "root_cause" : [
      {
        "type" : "index_not_found_exception",
        "reason" : "no such index [movie]",
        "resource.type" : "index_or_alias",
        "resource.id" : "movie",
        "index_uuid" : "_na_",
        "index" : "movie"
      }
    ],
    "type" : "index_not_found_exception",
    "reason" : "no such index [movie]",
    "resource.type" : "index_or_alias",
    "resource.id" : "movie",
    "index_uuid" : "_na_",
    "index" : "movie"
  },
  "status" : 404
}
```
### 查看索引

#### 查看所有索引
```
GET _cat/indices?v
```

#### 查看s开头的索引
```
GET _cat/indices/s*?v
```


### 添加和更新文档

在 ES 7 中新增索引：
```
PUT student
{
  "mappings" : {
    "properties" : {
      "name" : {
        "type" : "keyword"
      },
      "age" : {
        "type" : "integer"
      }
    }
  },
  "settings" : {
    "index" : {
      "number_of_shards" : 1,
      "number_of_replicas" : 0
    }
  }
}
```


#### 使用 POST
POST 用于更新数据，如果不存在，则会创建。

##### 添加数据示例1：
```
# 请求
POST student/_doc
{
  "name": "张三"
}

# 响应
{
  "_index" : "student",
  "_type" : "_doc",
  "_id" : "gCJ8Tm4Buf-uwlbZzC7C",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```


##### 添加数据示例2：
指定 _id 为 2。
```
# 请求
POST student/_doc/2
{
  "name": "李四"
}

# 响应
{
  "_index" : "student",
  "_type" : "_doc",
  "_id" : "2",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}
```

#### 一个错误的更新数据方式

```
# 请求
GET student/_doc/2

# 响应
{
  "_index" : "student",
  "_type" : "_doc",
  "_id" : "2",
  "_version" : 1,
  "_seq_no" : 1,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "李四"
  }
}
```
可以看到没有 age 字段。

更新 age：

```
# 请求
POST student/_doc/2
{
  "age": 10
}

# 响应
{
  "_index" : "student",
  "_type" : "_doc",
  "_id" : "2",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 2,
  "_primary_term" : 1
}
```

再次请求数据：
```
GET student/_doc/2

# 响应
{
  "_index" : "student",
  "_type" : "_doc",
  "_id" : "2",
  "_version" : 2,
  "_seq_no" : 2,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "age" : 10
  }
}
```

结果是 version 从1变成了2，而 name 字段不见了。

原因是 POST student/_doc/2 这种语法的效果是覆盖数据。可以理解为先把原文档删除，再索引新文档。

POST student/_doc/2 的效果相同。


#### 使用 _update 更新文档

如何在 name 不消失的情况下更新 age 呢？用 _update。

```
# 请求1
POST student/_doc/2
{
  "name": "李四"
}

# 请求2: 新增 age
POST student/_doc/2/_update
{
  "doc": {
    "age": 10
  }
}

# 查询
GET student/_doc/2

# 查询结果
{
  "_index" : "student",
  "_type" : "_doc",
  "_id" : "2",
  "_version" : 6,
  "_seq_no" : 6,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "李四",
    "age" : 10
  }
}
```

但是在使用 POST student/_doc/2/_update  这种方式会报错：

```
#! Deprecation: [types removal] Specifying types in document update requests is deprecated, use the endpoint /{index}/_update/{id} instead.
```


所以，建议用下面的方法：

```
POST student/_update/2
{
  "doc": {
    "age": 11
  }
}
```

使用 _update 时，ES 做了下面几件事：

- 从旧文档构建 JSON
- 更改该 JSON
- 删除旧文档
- 索引一个新文档

#### 使用 _update_by_query 更新文档
```
POST student/_update_by_query
{
  "query": { 
    "match": {
      "_id": 2
    }
  },
  "script": {
    "source": "ctx._source.age = 12"
  }
}
```



### 通过 _bulk 批量添加文档

创建索引：
```
PUT student
{
  "mappings" : {
    "properties" : {
      "name" : {
        "type" : "keyword"
      },
      "age" : {
        "type" : "integer"
      }
    }
  }
}
```
使用 _bulk 创建文档
```
POST _bulk
{ "index" : { "_index" : "student", "_id" : "1" } }
{ "name" : "张三" }
{ "index" : { "_index" : "student", "_id" : "2" } }
{ "name" : "李四", "age": 10 }
{ "index" : { "_index" : "student", "_id" : "3" } }
{ "name" : "王五", "age": 11 }


响应：

{
  "took" : 21,
  "errors" : false,
  "items" : [
    {
      "index" : {
        "_index" : "student",
        "_type" : "_doc",
        "_id" : "1",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 0,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "index" : {
        "_index" : "student",
        "_type" : "_doc",
        "_id" : "2",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 1,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "index" : {
        "_index" : "student",
        "_type" : "_doc",
        "_id" : "3",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 2,
        "_primary_term" : 1,
        "status" : 201
      }
    }
  ]
}
```





参考自：https://www.letianbiji.com/elasticsearch