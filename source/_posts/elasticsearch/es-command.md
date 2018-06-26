---
title: Elasticsearch 常用命令
categories: elasticsearch
tags: #文章標籤 可以省略
  - es
---
#### 计算集群中文档的数量: _count

输入:

```js
GET _count
```

输出:

```js
{
  "count": 13523,
  "_shards": {
    "total": 13,
    "successful": 13,
    "skipped": 0,
    "failed": 0
  }
}
```

#### 添加索引数据: PUT

```js
PUT /megacorp/employee/1
{
  "first_name": "John",
  "last_name": "Smith",
  "age": 25,
  "about": "I love to go rock climbing",
  "interests": ["sports", "music"]
}
```

路径 /megacorp/employee/1 包含了三部分的信息

- megacorp: 索引名称
- employee: 类型名称
- 1: 特定的ID

#### 检索文档: GET

```js
GET /megacorp/employee/1
```

结果

```js
{
  "_index": "megacorp",     // 索引名称
  "_type": "employee",      // 类型名称
  "_id": "1",               // 特定id
  "_version": 3,
  "found": true,
  "_source": {
    "first_name": "John",
    "last_name": "Smith",
    "age": 25,
    "about": "I love to go rock climbing",
    "interests": [
      "sports",
      "music"
    ]
  }
}
```

#### 轻量搜索: _search

查询 last_name 为 Smith 的雇员

```js
GET /megacorp/employee/_search?q=last_name:Smith
```

结果

```js
{
  "took": 3,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 0.2876821,
    "hits": [
      {
        "_index": "megacorp",
        "_type": "employee",
        "_id": "2",
        "_score": 0.2876821,
        "_source": {
          "first_name": "Jane",
          "last_name": "Smith",
          "age": 32,
          "about": "I like to collect rock albums",
          "interests": [
            "music"
          ]
        }
      },
      {
        "_index": "megacorp",
        "_type": "employee",
        "_id": "1",
        "_score": 0.2876821,
        "_source": {
          "first_name": "John",
          "last_name": "Smith",
          "age": 25,
          "about": "I love to go rock climbing",
          "interests": [
            "sports",
            "music"
          ]
        }
      }
    ]
  }
}
```

#### 使用查询表达式搜索

Elasticsearch 提供一个丰富灵活的查询语言叫做查询表达式，它支持构建更加复杂和健壮的查询。领域特定语言（DSL），指定了使用一个 JSON 请求。

查询 last_name 为 Smith 的雇员

```js
GET /megacorp/employee/_search
{
  "query": {
    "match": {
      "last_name": "Smith"
    }
  }
}
```

查询结果与轻量搜索查询的结果是一样的。

#### 过滤查询

查询 last_name 为 Smith 且 age 大于 30 的雇员

```js
GET /megacorp/employee/_search
{
"query": {
  "bool": {
    "must": {
      "match" : {
        "last_name" : "smith"
      }
    },
    "filter": {
      "range" : {
        "age" : { "gt" : 30 }
        }
      }
    }
  }
}
```

#### 全文搜索

查询 about 取值是 "rock climbing" 的雇员

```js
GET /megacorp/employee/_search
{
  "query" : {
    "match" : {
      "about" : "rock climbing"
    }
  }
}
```

查询结果得到两个匹配的文档，按照相关性排序。

```js
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 0.5753642,
    "hits": [
      {
        "_index": "megacorp",
        "_type": "employee",
        "_id": "1",
        "_score": 0.5753642,    // 相关性得分
        "_source": {
          "first_name": "John",
          "last_name": "Smith",
          "age": 25,
          "about": "I love to go rock climbing",
          "interests": [
            "sports",
            "music"
          ]
        }
      },
      {
        "_index": "megacorp",
        "_type": "employee",
        "_id": "2",
        "_score": 0.2876821,
        "_source": {
          "first_name": "Jane",
          "last_name": "Smith",
          "age": 32,
          "about": "I like to collect rock albums",
          "interests": [
            "music"
          ]
        }
      }
    ]
  }
}
```

#### 短语搜索

如果想要执行这样的一个查询，要同时匹配搜索条件，可以使用 match_phrase 的查询。查询同时包含 "rock" 和 "climbing"，并且已短语 rock climbing 的形式出现。

```js
GET /megacorp/employee/_search
{
  "query" : {
    "match_phrase" : {
      "about" : "rock climbing"
    }
  }
}
```

结果

```js
{
  ...,
  "hits": {
    "total": 1,
    "max_score": 0.5753642,
    "hits": [
      {
        "_index": "megacorp",
        "_type": "employee",
        "_id": "1",
        "_score": 0.5753642,
        "_source": {
          "first_name": "John",
          "last_name": "Smith",
          "age": 25,
          "about": "I love to go rock climbing",
          "interests": [
            "sports",
            "music"
          ]
        }
      }
    ]
  }
}
```
