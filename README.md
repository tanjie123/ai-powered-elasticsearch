## AI-Powered Elasticsearch

## 概述

本项目是参照AI-Powered Search[^1]书籍和Elasticsearch Learning to Rank[^2]指南构建的示例应用，以期望实现智能化搜索。


## 功能
- [ ] Signal Boosting
- [ ] Auto Learning to Rank
- [ ] Query Suggestion
- [ ] 知识图谱 

## 信号数据
不同类型的信号有不同的属性需要记录。例如，对于“查询”信号，我们要记录用户的关键字。对于“点击”信号，我们想要记录点击了哪个文档，以及哪个查询导致了点击。为了后续分析，我们还需要记录用户在查询后返回并可能查看了哪些文档。为了使事情更具可扩展性并避免为每种新信号类型编写自定义代码，我们采用一种通用格式来表示信号。

信号的数据结构：

- query_id：发起此信号的查询信号的唯一ID
- user:  代表特定用户的标识符
- type:  什么样的信号（查询、点击、购买等）
- target：信号在此signal_time适用的内容
- signal_time:  信号发生的日期和时间



例如，假设用户执行了以下操作序列：

1. 发起一次"ipad"查询，并返回了三个文档(doc1，doc2，doc3)
2. 点击doc1
3. 返回搜索结果列表并点击doc3
4. 将加入doc3购物车
5. 回去搜索框，发起"ipad cover"查询，并返回了两个文档(doc4，doc5)
6. 点击doc4
7. 将加入doc4购物车
8. 购买了购物车中(doc3, doc4)商品

用户在与系统的交互中，将产生以下信号数据：

| query_id | user | type        | target           | signal_time         |
| -------- | ---- | ----------- | ---------------- | ------------------- |
| 1        | u123 | query       | ipad             | 2020-05-01-09:00:00 |
| 1        | u123 | result      | doc1、doc2、doc3 | 2020-05-01-09:00:00 |
| 1        | u123 | click       | doc1             | 2020-05-01-09:00:10 |
| 1        | u123 | click       | doc3             | 2020-05-01-09:00:29 |
| 1        | u123 | add_to_cart | doc1             | 2020-05-01-09:03:40 |
| 2        | u123 | query       | ipad cover       | 2020-05-01-09:04:00 |
| 2        | u123 | result      | doc4、doc5       | 2020-05-01-09:04:00 |
| 2        | u123 | click       | doc4             | 2020-05-01-09:04:40 |
| 2        | u123 | add_to_cart | doc4             | 2020-05-01-09:05:50 |
| 1        | u123 | purchase    | doc3             | 2020-05-01-09:07:15 |
| 2        | u123 | purchase    | doc4             | 2020-05-01-09:07:15 |



## Signal Boosting

我们演示了`ipad`在我们的'products'数据集中对查询进行的开箱即用的搜索。

```
query = "ipad"

collection = "products"
request = {
    "query": query,
    "fields": ["upc", "name", "manufacturer", "score"],
    "limit": 5,
    "params": {
      "qf": "name manufacturer longDescription",
      "defType": "edismax"
    }
}

search_results = requests.post(solr_url + collection + "/select", json=request).json()["response"]["docs"]
display(HTML(render_search_results(query, search_results)))
```



## Todo

- [x] Docker & Docker Compose
- [ ] 定义前端埋点数据格式
- [ ] 基于Click Model生成SDBN数据 
- [ ] 构建Judgement List
- [ ] 基于 RankLib 训练LTR模型
- [ ] 


[^1]: AI-Powered Search，https://www.manning.com/books/ai-powered-search
[^2]: Elasticsearch Leaning to Rank, https://elasticsearch-learning-to-rank.readthedocs.io/en/latest/
