# 前言

Elastic 的所有数据操作都基于索引（`indices`）。在 Elastic 的概念中，索引可以理解为关系型数据库的库名。在索引下面又分类型（`Type`），可以将类型理解为关系型数据库的数据表（`Table`）。

下面就来看下索引的基本操作。

# 新增索引

新增索引使用如下命令，`pretty` 选项是为了以更友好的方式展示数据：

```
PUT /<Index>(?pretty)
```

| 说明                                |
| :---------------------------------- |
| 在索引新增操作只能使用 `PUT` 请求。 |

现在来新增一个名称为 `customer` 的索引：

```bash
$ curl -XPUT "localhost:9200/customer?pretty"
```

执行结果如下：

```json
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "customer"
}
```

acknowledged 为 true 表示删除成功，另外也可以看到 index 值为 customer。

| 索引不支持修改操作                                           |
| :----------------------------------------------------------- |
| 如果使用 POST 就会提示 405 错误：`Incorrect HTTP method for uri [/customer?pretty] and method [POST], allowed: [GET, HEAD, DELETE, PUT]`。 |

| 重复添加？                                                   |
| ------------------------------------------------------------ |
| 如果尝试新增一个已存在的索引就会提示 `resource_already_exists_exception` 错误。 |

# 列出全部索引

查看索引可以使用 Elastic 的 `_cat` 指令。如下所示：

```
GET /_cat/indices?v&pretty
```

先看下使用 `pretty` 可选参数：

```bash
$ curl -XGET "localhost:9200/_cat/indices?pretty"

yellow open customer XF00V_ncTF-RYX2JFbFQTA 1 1 0 0 283b 283b
```

再来看下 `v` 可选参数：

```bash
$ curl -XGET "localhost:9200/_cat/indices?v"

health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   customer XF00V_ncTF-RYX2JFbFQTA   1   1          0            0       283b           283b
```

很直观的就可以看出两个可选参数的区别，`v` 除了数据之外还列出了 Title 信息，这更加说明了可选参数 `v` 更适合类似 Table 格式的数据。

# 删除索引

索引删除使用 `DELETE`，格式如下：

```
DELETE /<Index>(?v&pretty)
```

现在来将索引 `customer` 删掉：

```bash
$ curl -XDELETE "localhost:9200/customer?pretty"
```

执行结果如下：

```json
{
  "acknowledged" : true
}
```

acknowledged 为 true 表示删除成功。

# 扩展说明

Elastic 不要求必须要有索引才能存放数据。事实上，在添加文档时如果索引不存在会自行创建索引。下面来验证下：

先列出本机所有索引：

```bash
$ curl -XGET "localhost:9200/_cat/indices?v"
$
```

可以看到当前机器上已没有索引，现在我们要创建的索引名称为 `theme`，不过我们不会调用 `POST /<Index` API，而是以添加文档数据的形式：

```bash
$ curl -XPOST "localhost:9200/theme/_doc?pretty" -H "Content-Type: application/json" -d'
{
  "name": "Bob",
  "country": "America"
}'
```

上面的语句意思是在索引 `theme` 创建一个名为 `_doc` 类型的文档，向该文档中新增一条 JSON 格式数据。数据包含用于名称与国家，现在执行。

输出结果如下：

```json
{
  "_index" : "theme",
  "_type" : "_doc",
  "_id" : "45BB73UB5Tq_xcodohEx",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

从返回的信息中可以看到：索引为 `theme`，类型为：`_doc`。并且 Elasticsearch 主动创建了文档数据的 `id`：`tlfYAW4BcNwnFQPz_yBb`。

先不看数据是否创建成功，来看下索引是否成功创建：

```bash
$ curl -XGET "localhost:9200/_cat/indices?v"

health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   theme SVCeDdrQQOuQA4ZFTXfTtw   1   1          1            0        4kb            4kb
```

在输出的索引列表中可以看到 `theme`，说明在向文档中插入数据时 Elasticsearch 并不强制要求指定索引一定存在，这点与 `MongoDB` 类似。