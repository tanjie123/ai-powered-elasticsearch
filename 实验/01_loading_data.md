## 实验01
通过kibana向ES添加products索引
```
PUT products/_doc/1 
{
  "id": 1,
  "name": "苹果电脑",
  "desc": "这是苹果电脑哟..."
}

PUT products/_doc/2 
{
  "id": 2,
  "name": "苹果手表",
  "desc": "这是苹果手表哟..."
}

PUT products/_doc/3
{
  "id": 3,
  "name": "苹果充电器",
  "desc": "这是苹果充电器哟..."
}

PUT products/_doc/4
{
  "id": 4,
  "name": "苹果手机",
  "desc": "这是苹果手机器哟..."
}
```
## 实验内容 
一般而言，在一个商城中，苹果电脑和苹果手机的关注度会更高，而苹果充电器和苹果手表的关注度会更低。如果单纯地根据ES内置地BM25相关性算法进行文本统计，则苹果手机和苹果电脑会排在苹果手表地后面。

## 执行
原始查询：
```
GET products/_search
{
  "query": {
    "match": {
      "name": "苹果"
    }
  }
}
```

我们可以通过用户行为数据来优化查询，具体的逻辑是手机用户的行为数据作为信号用来调优相关性算法。
步骤1：收集信号,写入`signals`索引
```
PUT signals/_doc/1
{
  "query_id": 1,
  "user": "u123",
  "type": "query",
  "target": "苹果",
  "signal_time": "2019-07-31 08:49:07.3116"
}

PUT signals/_doc/2
{
  "query_id": 1,
  "user": "u123",
  "type": "results",
  "target": "1,2,3,4",
  "signal_time": "2019-07-31 08:50:07.3116"
}

PUT signals/_doc/3
{
  "query_id": 1,
  "user": "u123",
  "type": "click",
  "target": "1",
  "signal_time": "2019-07-31 08:51:07.3116"
}

PUT signals/_doc/4
{
  "query_id": 1,
  "user": "u123",
  "type": "click",
  "target": "4",
  "signal_time": "2019-07-31 08:52:07.3116"
}

PUT signals/_doc/5
{
  "query_id": 1,
  "user": "u123",
  "type": "add_to_cart",
  "target": "4",
  "signal_time": "2019-07-31 08:52:07.3116"
}

PUT signals/_doc/6
{
  "query_id": 2,
  "user": "u123",
  "type": "query",
  "target": "苹果手机",
  "signal_time": "2019-07-31 08:53:07.3116"
}

PUT signals/_doc/7
{
  "query_id": 2,
  "user": "u123",
  "type": "query",
  "target": "苹果手机",
  "signal_time": "2019-07-31 08:54:07.3116"
}

PUT signals/_doc/8
{
  "query_id": 2,
  "user": "u123",
  "type": "results",
  "target": "4",
  "signal_time": "2019-07-31 08:55:07.3116"
}


PUT signals/_doc/9
{
  "query_id": 2,
  "user": "u123",
  "type": "click",
  "target": "4",
  "signal_time": "2019-07-31 08:56:07.3116"
}

PUT signals/_doc/10
{
  "query_id": 2,
  "user": "u123",
  "type": "add_to_cart",
  "target": "4",
  "signal_time": "2019-07-31 08:57:07.3116"
}

PUT signals/_doc/11
{
  "query_id": 1,
  "user": "u123",
  "type": "purchase",
  "target": "1",
  "signal_time": "2019-07-31 08:58:07.3116"
}

PUT signals/_doc/12
{
  "query_id": 1,
  "user": "u123",
  "type": "purchase",
  "target": "4",
  "signal_time": "2019-07-31 08:59:07.3116"
}
```

或者写入数据库
```
INSERT INTO test.signals (query_id, user, type, target, signal_time) VALUES ('1', 'u123', 'query', '苹果', '2019-07-31 08:49:07.3116');
INSERT INTO test.signals (query_id, user, type, target, signal_time) VALUES ('1', 'u123', 'results', '1,2,3,4', '2019-07-31 08:50:07.3116');
INSERT INTO test.signals (query_id, user, type, target, signal_time) VALUES ('1', 'u123', 'click', '1', '2019-07-31 08:51:07.3116');
INSERT INTO test.signals (query_id, user, type, target, signal_time) VALUES ('1', 'u123', 'add_to_cart', '4', '2019-07-31 08:52:07.3116');
INSERT INTO test.signals (query_id, user, type, target, signal_time) VALUES ('2', 'u123', 'query', '苹果手机', '2019-07-31 08:53:07.3116');
INSERT INTO test.signals (query_id, user, type, target, signal_time) VALUES ('2', 'u123', 'results', '4', '2019-07-31 08:54:07.3116');
INSERT INTO test.signals (query_id, user, type, target, signal_time) VALUES ('2', 'u123', 'click', '4', '2019-07-31 08:55:07.3116');
INSERT INTO test.signals (query_id, user, type, target, signal_time) VALUES ('2', 'u123', 'add_to_cart', '4', '2019-07-31 08:56:07.3116');
INSERT INTO test.signals (query_id, user, type, target, signal_time) VALUES ('1', 'u123', 'purchase', '1', '2019-07-31 08:57:08.3116');
INSERT INTO test.signals (query_id, user, type, target, signal_time) VALUES ('2', 'u123', 'purchase', '4', '2019-07-31 08:58:07.3116');
```

步骤2：对signal数据进行统计, 并写入'signal_boosting'
```
signals_aggregation_query = """
select q.target as query, c.target as doc, count(c.target) as boost
  from signals c left join signals q on c.query_id = q.query_id
  where c.type = 'click' AND q.type = '苹果'
  group by query, doc
  order by boost desc
"""
```

使用信号改进查询算法
```
GET products/_search
{
  "query": {
    "function_score": {
      "query": {
        "bool": {
          "must": [
            {
              "bool": {
                "should": [
                  {
                    "multi_match": {
                      "query": "苹果",
                      "fields": [
                        "name^1",
                        "desc^5"
                      ]
                    }
                  }
                ]
              }
            }
          ]
        }
      },
      "functions": [
        {
          "filter": {
            "terms": {
              "id": [1,4]
            }
          },
          "weight": 2
        }
      ],
      "boost_mode": "sum"
    }
  }
}
```