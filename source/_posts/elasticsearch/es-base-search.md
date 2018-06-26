---
title: Elasticsearch 基础查询语法
categories: elasticsearch
tags: #文章標籤 可以省略
  - es
---

### 模糊匹配: match

- 查询条件: entry_key 等于 app.scan.go_where

```js
GET _search
{
  "query": {
    "match": {
      "entry_key.keyword": "app.scan.go_where"
    }
  }
}
```

### 过滤搜索: filter

- 查询条件: 1.entry_key 等于 app.scan.go_where，2.new_entry_detail 包含 barcode 或者 qrCode，同时满足
- 过滤条件: app_version 在 6010041 和 6010042 之间

```js
GET /fe_app_log-2018.04.30/fe_app_log/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "entry_key.keyword": "app.scan.go_where"
          }
        },
        {
          "match_phrase": {
            "new_entry_detail": "barcode qrCode"
          }
        }
      ],
      "filter": {
        "range": {
          "app_version": {
            "gte":  ,
            "lte": 6010042
          }
        }
      }
    }
  }
}
```

### 短语搜索: match_phrase

```js
GET /fe_app_log-2018.04.30/fe_app_log/_search
{
  "query": {
    "match_phrase": {
      "new_entry_detail": "barcode=12"
    }
  }
}
```

短语搜索和检索的区别是:

- 短语搜索是`精确匹配`一系列单词或者短语
- 检索会默认拆词、根据相关性得分降序返回结果

#### 分组: aggs

```js
GET /fe_app_log-2018.04.30/fe_app_log/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "entry_key.keyword": "app.scan.go_where"
          }
        },
        {
          "match": {
            "new_entry_detail": "barcode"
          }
        }
      ],
      "filter": {
        "range": {
          "app_version": {
            "gte": 6010041,
            "lte": 6010042
          }
        }
      }
    }
  },
  "aggs": {
    "userId": {
      "terms": {
        "field": "user_id.keyword"
      },
      "aggs": {
        "version": {
          "terms": {
            "field": "app_version.keyword"
          }
        }
      }
    }
  }
}
```
检索结果按 user_id.keyword 和 app_version 进行分组，支持分级汇总
