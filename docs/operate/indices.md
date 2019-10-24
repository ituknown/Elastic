# 前言

Elastic 的所有数据操作都基于索引（`indices`）。在 Elastic 的概念中，索引可以理解为关系型数据库的库名。在索引下面又分类型（`Type`），可以将类
型理解为关系型数据库的数据表（`Table`）。

在 Elastic 中，所有的获取操作都使用 `GET`，所有的删除操作都使用 `DELETE`，所有的新增或修改操作可以使用 `PUT` 或 `POST`。

下面就来看下索引的基本操作。

# 新增索引

新增索引使用如下命令，`pretty` 选项是为了以更友好的方式展示数据：

```
PUT /<Index>(?pretty&pretty)
```

> **注意：** 在索引新增操作只能使用 `PUT` 请求。

现在来新增一个名称为 `customer` 的索引：

```bash
$ curl -XPUT "localhost:9200/customer?pretty&pretty"
```

执行结果如下：

```json
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "customer"
}
```

`acknowledged` 值为 `true` 表示操作成功，另外也可以看到 `index` 值为 `customer`。

> **注意：** 如果使用 `POST` 将会抛出如下错误：
```json
{
  "error" : "Incorrect HTTP method for uri [/customer?pretty&pretty] and method [POST], allowed: [GET, HEAD, DELETE, PUT]",
  "status" : 405
}
```

# 列出全部索引

查看索引可以使用 Elastic 的 `_cat` 指令。如下所示：

```
GET /_cat/indices?v&pretty
 或
GET /_cat/indices?pretty&pretty
```

两者区别是 `v` 指令展示的更详细，而 `pretty` 则展示的更直观：

```bash
curl -XGET "localhost:9200/_cat/indices?v&pretty"

health status index                         uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .monitoring-es-6-2019.10.22   iygh5IibSbaFDxTE9RyQOA   1   0      10135          119      4.3mb          4.3mb
green  open   .watches                      Bptdm6_ASx69bYYJ495nvw   1   0          6            0     75.2kb         75.2kb
green  open   .monitoring-alerts-6          Qb7JbFCNTy-l-lAyBiMADg   1   0          1            0      6.1kb          6.1kb
green  open   .watcher-history-7-2019.10.21 rC7xOKLQTXaDLDLV_cQ05Q   1   0       2758            0      3.4mb          3.4mb
green  open   .kibana                       G6D_RxaGR5CGeHKdGp_GKw   1   0          1            0        4kb            4kb
green  open   .triggered_watches            9H35HOwtTUCGcjrH2lI3bQ   1   0          0            0     67.3kb         67.3kb
green  open   .watcher-history-7-2019.10.22 zfWVREXaT8GWMnKPZkQDbw   1   0       1096            0      1.1mb          1.1mb
yellow open   customer                      PQydIt6OSvWzfQ1qGbN-Kw   5   1          0            0      1.2kb          1.2kb
```

> 由于之前已经存储过数据，所以在展示时可以看到除了新建的 `customer` 索引之外还有其他几个索引，这里不做说明。

从展示的数据详情中可以很直观的看出索引的健康状态，占用内存数量等。

再来看下 `pretty` 展示的结果：

```bash
curl -XGET "localhost:9200/_cat/indices?pretty&pretty"

green  open .monitoring-es-6-2019.10.22   iygh5IibSbaFDxTE9RyQOA 1 0 10327 85  4.3mb  4.3mb
green  open .watches                      Bptdm6_ASx69bYYJ495nvw 1 0     6  0 75.4kb 75.4kb
green  open .monitoring-alerts-6          Qb7JbFCNTy-l-lAyBiMADg 1 0     1  0  6.1kb  6.1kb
green  open .watcher-history-7-2019.10.21 rC7xOKLQTXaDLDLV_cQ05Q 1 0  2758  0  3.4mb  3.4mb
green  open .kibana                       G6D_RxaGR5CGeHKdGp_GKw 1 0     1  0    4kb    4kb
green  open .triggered_watches            9H35HOwtTUCGcjrH2lI3bQ 1 0     0  0 67.3kb 67.3kb
green  open .watcher-history-7-2019.10.22 zfWVREXaT8GWMnKPZkQDbw 1 0  1120  0  1.1mb  1.1mb
yellow open customer                      PQydIt6OSvWzfQ1qGbN-Kw 5 1     0  0  1.2kb  1.2kb
```

从展示的数据可以看出，`v` 和 `pretty` 的区别之处在于 `v` 展示了 title 信息，可以根据自己的喜好选择。

# 删除索引

索引删除使用 `DELETE`，格式如下：

```
DELETE /<Index>(?pretty&pretty)
```

现在来将索引 `customer` 删掉：

```bash
$ curl -XDELETE "localhost:9200/customer?pretty&pretty"
```

结果如下，表示删除成功：

```json
{
  "acknowledged" : true
}
```