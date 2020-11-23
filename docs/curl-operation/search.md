# 前言

使用 Elasticsearch 的初衷就是因为它强大的数据查找功能。前面说了文档数据的添加、替换、修改、删除以及获取文档数据基础 API，而这些基础 API 都是为了Elasticsearch 的 `_search` API 做铺垫。

本节所有的数据查找操作都以 [数据批处理](./batch.md) 中在最后批量导入 bank 索引的数据为前提。如果还没有导入 `accounts.json` 文件中的数据需要先进行导入。

# _search API

Elasticsearch 的 `_search` 有使用方式有两种风格：`REST request URI` 和 `REST request body`。

`REST request URI` 方式就是将查找的数据条件直接拼接在请求 `url` 中，而 `REST request body` 则是将数据放在请求体中，格式为 `JSON`。

`_search` API 请求格式一般如下：

```
GET /<Index>/_search
```

现在，先看下 `REST request URI` 形式的示例：

```bash
$ curl -XGET "localhost:9200/bank/_search?q=*&sort=account_number:asc&pretty"
```

在平时使用搜索引擎查找数据时注意看请求地址栏的数据信息，对这个查找应该不陌生。现在就来说下具体含义：

- `q=*`：查找 bank 索引下的所有文档数据。
- `sort=account_number:asc`：查找 bank 索引下所有的文档数据，并按照文档数据的 `account_number` 字段进行倒叙排序。这就类似于关系型数据库 MySQL 的 `ORDER BY FIELD ASC`。

现在来看下查询结果：

```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : null,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "0",
        "_score" : null,
        "_source" : {
          "account_number" : 0,
          "balance" : 16623,
          "firstname" : "Bradshaw",
          "lastname" : "Mckenzie",
          "age" : 29,
          "gender" : "F",
          "address" : "244 Columbus Place",
          "employer" : "Euron",
          "email" : "bradshawmckenzie@euron.com",
          "city" : "Hobucken",
          "state" : "CO"
        },
        "sort" : [0]
      },
      ...
    ]
  }
}
```

下面来说下返回的具体信息含义：

* `took`：Elastic 查找数据耗时，单位毫秒。
* `timed_out`：告诉查找数据是否执行超时。
* `_shards`：该信息主要告诉我们 Elastic 查找的数据分片，如这里 `_shards.successful` 值为 5 就表示分片查找成功数。
* `hits`：查找数据命中结果。
  + `total`：总数据匹配数
  + `hits`：数据返回集，默认返回10条，可以在查询是使用 `size` 参数进行指定。
  + `sort`：排序序列，类似于索引（从 0 开始）

可以看到，直接使用 `REST request URI` 的形式进行查找数据不利于阅读。现在再来看下使用 `REST request body` 的写法，将上面相同的查询条件直接是用 `body` 形式替换如下：

```bash
$ curl -XGET "localhost:9200/bank/_search?pretty" -H "Content-Type: application/json" -d'
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ]
}'
```

这里 `query` 就是指定的查询条件。`match_all` 指定查询 `bank` 索引下所有文档数据。`sort` 为排序，可以看到，接受的是一个数组，所以可以指定多个字段进行排序。

# 每页查询数量

在上面我们在查询时指定的查询条件是查询所有文档数据 `match_all`，并且按照 `account_number` 字段进行倒叙排序。响应结果虽然命中了 1000 条，实际返回了 10 条数据。这是因为 Elasticsearch 在查询条件时如果不指定查询数量将默认查询 10 条，现在我想查询 20 条：

```bash
$ curl -XGET "localhost:9200/bank/_search?pretty" -H "Content-Type: application/json" -d'
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ],
  "size": 20
}'
```

这样，在查询时就会返回 20 条数据。仔细看数据你会看到第一条数据的 `sort` 值为 0，最后一条是 19。

# 分页

Elasticsearch 既然有 `size` 参数，同样肯定会有一个 `from` 参数。`size` 指定了每页查询数量，`from` 则指定从第一条开始查找。看下下面的示例，从第 10 条开始查询，向后查询10条：

```bash
$ curl -XGET "localhost:9200/bank/_search?pretty" -H "Content-Type: application/json" -d'
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ],
  "size": 10,
  "from": 10
}'
```

查询后可以比较下结果，你会看到查询的最后一条数据是 19，其实就是在上面查找的 20 条数据的最后一条数据。

# 排序

在上面排序一直使用的是 `sort` 数组形式，其实 Elastic 还有一种 `JSON` 形式写法：

```bash
$ curl -XGET "localhost:9200/bank/_search?pretty" -H "Content-Type: application/json" -d'
{
  "query": { "match_all": {}},
  "sort": {
    "account_number": {"order": "asc"}
  },
  "size": 20
}'
```

输出后可以比较下是不是与之前一样的结果。

# 返回指定字段

通常，一个文档数据包含众多字段。在一次查询中我们一般都仅仅需要其中几个字段。如果直接返回全部字段一方面会降低系统响应速度（尽管微乎其微），另一方面则会占用网络带宽。

如关系型数据库查询语句：

```mysql
select name, age from user
```

在之前的查询中，如果你仔细观察返回的数据格式，你会发现文档数据都是包含在 `_source` 字段下。所以，想要返回指定字段，我们仅仅需要将需要返回的字段在
`_source` 中指定即可，`_source` 接受的是数组形式。如下所示：

```bash
$ curl -XGET "localhost:9200/bank/_search?pretty" -H "Content-Type: application/json" -d'
{
  "_source": ["account_number", "balance"],
  "query": { "match_all": {}},
  "size": 1
}'
```

查询结果如下：

```json
{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1000,
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "bank",
        "_type" : "_doc",
        "_id" : "25",
        "_score" : 1.0,
        "_source" : {
          "account_number" : 25,
          "balance" : 40540
        }
      }
    ]
  }
}
```

可以看到，在 `hits._source` 中仅仅返回了 `account_number` 和 `balance` 字段。

# 进阶 _search API

在前面，我们在查询文档数据时使用的都是如下形式：

```json
{
  "query": {"match_all": {}} 
}
```

这个 `query` 就是我们的查询条件。但是在指定条件时一直使用的是 `match_all`，该 API 含义是查询指定索引下的所有文档数据，并没有指定具体条件。

是不是很容易想到如何指定条件 －－ `match`。

看下下面的示例：

```bash
$ curl -XGET "localhost:9200/bank/_search?pretty" -H "Content-Type: application/json" -d'
{
  "query": {
    "match": { "account_number": 20}
  }
}'
```

这个查询语句就是指定的查询字段 `account_number` 值为 20 的数据。

如何实现模糊查询呢？在 MySQL 等关系型数据库进行模糊匹配是都会使用 `LIKE` 语法，并且指定 `%`。而在 Elastic 中则不需要，看下下面的示例：

- 查询地址包含 `mill` 的所有数据：

```bash
$ curl -XGET "localhost:9200/bank/_search?pretty" -H "Content-Type: application/json" -d'
{
  "query": {
    "match": { "address": "mill" }
  }
}'
```

- 查询地址包含包含 `mill` 或 `lane` 的所有数据：

```bash
$ curl -XGET "localhost:9200/bank/_search?pretty" -H "Content-Type: application/json" -d'
{
  "query": {
    "match": { "address": "mill lane" }
  }
}'
```

- 查询地址包含 `mill` 并且包含 `lane` 的所有数据：

```bash
$ curl -XGET "localhost:9200/bank/_search?pretty" -H "Content-Type: application/json" -d'
{
  "query": {
    "match_phrase": { "address": "mill lane" }
  }
}'
```

# 高级 _search API `bool`

现在来继续看下 `bool` API，该 API 与之前说的 API 相同，不过有这更加强大的条件查询能力。

- 返回所有 `address` 包含 `mill` 并且包含 `lane` 的数据：

```bash
$ curl -XGET "localhost:9200/bank/_search?pretty" -H "Content-Type: application/json" -d'
{
  "query": {
    "bool": {
      "must": [
        {"match": { "address": "mill"}},
        {"match": { "address": "lane"}},
      ]
    }
  }
}'
```

- 返回所有 `address` 包含 `mill` 或者包含 `lane` 的数据：

```bash
$ curl -XGET "localhost:9200/bank/_search?pretty" -H "Content-Type: application/json" -d'
{
  "query": {
    "bool": {
      "should": [
        {"match": { "address": "mill"}},
        {"match": { "address": "lane"}}
      ]
    }
  }
}'
```

- 返回所有 `address` 不包含 `mill` 并且不包含 `lane` 的数据：

```bash
$ curl -XGET "localhost:9200/bank/_search?pretty" -H "Content-Type: application/json" -d'
{
  "query": {
    "bool": {
      "must_not": [
        {"match": { "address": "mill"}},
        {"match": { "address": "lane"}}
      ]
    }
  }
}'
```

- 返回所有 `age` 为 40 的用户，并且要求 `state` 不为 `ID`:

```bash
$ curl -XGET "localhost:9200/bank/_search?pretty" -H "Content-Type: application/json" -d'
{
  "query": {
    "bool": {
      "must": [
        {"match": { "age": "40"}}
      ],
      "must_not": [
        {"match": { "state": "ID"}}
      ]
    }
  }
}'
```

# 过滤及范围查找

如 MySQL 一样，我想要查询年龄在 18 ~ 22 岁之间的用户我可以使用如下语句：

```mysql
select * from uesr where age between 18 and 22;
```

Elasticsearch 想要实现范围查找需要借助 `filter` 条件过滤（`filter` 下需要使用 `range` ）。

现在，我要查询所有文档数据中 `balance` 值在 20000 ~ 30000 之间的数据（包含 20000 和 30000）可以使用如下语句：

```bash
$ curl -XGET "localhost:9200/bank/_search?pretty" -H "Content-Type: application/json" -d'
{
  "query": {
    "bool": {
      "must": { "match_all": {}},
      "filter": {
        "range": {
          "balance": { "gte": 20000, "lte": 30000 }
        }
      }
    }
  }
}'
```

- `lt`：小于
- `lte`：小于等于
- `gt`：大于
- `gte`：大于等于

**总结：** `bool` 相比较前面的查询语句表达的更加具体化，更便于阅读，在实际使用中应以 `bool` API 为主。