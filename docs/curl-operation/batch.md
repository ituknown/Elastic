# 前言

Elasticsearch 提供批处理（`batch processing`）功能，这对于大批量数据处理提供了更佳的处理方式，批处理的优点就不用说了。

如在前面我们在操作文档数据时都是一条一条增加，删除同样如此。Elasticsearch 的批处理使用的是 `_bulk` API，通常数据格式如下所示：

```
POST /<Index>/<Type>/_bulk?pretty
{
  { "<Action>": {"_id": "<ID>"} }
  { "<Key>": "<Value>"}
}
```

简单的说，JSON 格式的数据第一行跟的是操作的行为。

如果创建数据，那么 `Action` 就是 `index`，后面跟的是指定文档的 `ID`。如果想要向该文档数据插入数据，就在下一个 JSON 中设置属性与值。

如果是修改操作，`Action` 的值就是 `update`，后面同样跟随文档的 `ID`。下面的 JSON 就是要修改的文档数据。

如果是删除操作，`Action` 的值就是 `delete`，后面同样跟随文档的 `ID`。注意，后面就不再跟数据了。

看下下面的示例：

```bash
$ curl -X POST "localhost:9200/bank/customer/_bulk?pretty&pretty" -H 'Content-Type: application/json' -d'
{"index":{"_id":"1"}}
{"name": "John Doe" }
{"index":{"_id":"2"}}
{"name": "Jane Doe" }
{"update":{"_id":"1"}}
{"name": "John Doe's sister" }
{"delete":{"_id":"2"}}
'
```

这个语句的含义是：

在索引 bank 下，创建（如果 `ID` 已存在就是替换）文档数据 customer 并指定 `ID` 为 1，该文档数据的值为 `{"name": "John Doe" }`。
接着继续创建（如果 `ID` 已存在就是替换） `ID` 为 2 的文档数据。下面一个是 `update` 操作，该语句的意思是将文档 `ID` 为 1 的数据中的 `name` 的值修改为 `John Doe's sister`。最后是删除操作，将文档数据 `ID` 为 2 的数据删除。

下面来看下实例。

# 批量新增

在之前先将已存在的索引 `bank` 删掉，便于下面的说明。

```bash
$ curl -XDELETE "localhost:9200/bank?pretty"
```

现在开始批量增加数据（其实就两条，便于演示）：

```bash
$ curl -X POST "localhost:9200/bank/customer/_bulk?pretty" -H 'Content-Type: application/json' -d'
{"index":{"_id":"1"}}
{"name": "John Doe" }
{"index":{"_id":"2"}}
{"name": "Jane Doe" }
'
```

| 说明                                                         |
| :----------------------------------------------------------- |
| 虽然索引 bank 在第一步已经被删除，但是 Elasticsearch 并不强制要求新增文档数据时指定索引一定存在，Elasticsearch 会根据需要自行创建。 |

执行完成后来使用 `_search` 命令查看下数据：

```bash
$ curl -XGET "localhost:9200/bank/customer/_search?pretty"
```

输出结果如下：

```json
{
  "took" : 51,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [{
      "_index" : "bank",
      "_type" : "customer",
      "_id" : "1",
      "_score" : 1.0,
      "_source" : {
        "name" : "John Doe"
      }
    }, {
      "_index" : "bank",
      "_type" : "customer",
      "_id" : "2",
      "_score" : 1.0,
      "_source" : {
        "name" : "Jane Doe"
      }
    }]
  }
}
```

上面的数据先不做过多解释，只需要知道数据批量插入成功即可。

# 批量修改与删除

直接看示例：

```bash
$ curl -X POST "localhost:9200/bank/customer/_bulk?pretty" -H 'Content-Type: application/json' -d'
{"update":{"_id":"1"}}
{"doc": { "name": "John Doe becomes Jane Doe" } }
{"delete":{"_id":"2"}}
'
```

将索引 bank 下的 customer 文档中 ID 为 1 的数据进行一次修改。并删除 ID 为 2 的数据。执行后做一次查询：

```json
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : 1,
    "max_score" : 1.0,
    "hits" : [{
      "_index" : "bank",
      "_type" : "customer",
      "_id" : "1",
      "_score" : 1.0,
      "_source" : {
        "name" : "John Doe becomes Jane Doe"
      }
    }]
  }
}
```

可以看到，ID 为 1 的数据已经修改成功，并且 ID 为 2 的数据被删除。

# JSON 文件批量数据导入

现在我们来使用 JSON 文件批量导入数据。以银行账户信息数据为例，数据属性如下所示：

```json
{
    "account_number": 0,
    "balance": 16623,
    "firstname": "Bradshaw",
    "lastname": "Mckenzie",
    "age": 29,
    "gender": "F",
    "address": "244 Columbus Place",
    "employer": "Euron",
    "email": "bradshawmckenzie@euron.com",
    "city": "Hobucken",
    "state": "CO"
}
```

准备好一个 JSON 文件的数据：[accounts.json](./_file/accounts.json)（可以右键在新标签页预览保存）。

假设我将 `accounts.json` 文件放在 `_simple` 文件下，执行命令如下所示：

```bash
$ curl -XDELETE "localhost:9200/bank?pretty"
$ curl -XPOST "localhost:9200/bank/customer/_bulk?pretty&refresh" -H "Content-Type: application/json" --data-binary "@_file/accounts.json"
```

看下索引信息：

```bash
$ curl -XGET "localhost:9200/_cat/indices?v"

health status index   uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   bank    IBxsiHSLSXeblvdxM4KQIA   1   1       1000            0    414.2kb        414.2kb
yellow open   twitter VJJqMaxUT0qhW_HAbKlHBw   1   1          3            0      4.9kb          4.9kb
```

可以在 bank 索引下看到我们刚才新增的 1000 条客户数据。

