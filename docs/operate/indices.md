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

# 扩展说明

Elastic 不要求必须要有索引才能存放数据。事实上，在添加文档时如果索引不存在会自行创建索引。下面来验证下：

先列出本机所有索引：

```bash
$ curl -XGET "localhost:9200/_cat/indices?pretty&pretty"
```

```
green open .watcher-history-7-2019.10.25 d4zSGgcKT1CDV6LGbTar8Q 1 0    90   0  126.7kb  126.7kb
green open .watches                      Bptdm6_ASx69bYYJ495nvw 1 0     6   0   83.5kb   83.5kb
green open .monitoring-es-6-2019.10.25   3-Km-OaRQZ20NVJN8tDxqA 1 0  1231 104 1011.3kb 1011.3kb
green open .monitoring-alerts-6          Qb7JbFCNTy-l-lAyBiMADg 1 0     1   0    6.4kb    6.4kb
green open .watcher-history-7-2019.10.21 rC7xOKLQTXaDLDLV_cQ05Q 1 0  2758   0    3.4mb    3.4mb
green open .kibana                       G6D_RxaGR5CGeHKdGp_GKw 1 0     1   0      4kb      4kb
green open .triggered_watches            9H35HOwtTUCGcjrH2lI3bQ 1 0     0   0  123.2kb  123.2kb
green open .monitoring-es-6-2019.10.23   L_QsqDKFS6GibnIkYa2IKg 1 0  1548  18  893.9kb  893.9kb
green open .watcher-history-7-2019.10.24 -708MKfjSvqvi6Q5Dj6cKQ 1 0   234   0  348.1kb  348.1kb
green open .monitoring-es-6-2019.10.22   iygh5IibSbaFDxTE9RyQOA 1 0 14941  35    5.9mb    5.9mb
green open .monitoring-es-6-2019.10.24   gh3afcsRTZmAnQTkUz8N3A 1 0  2823  77    1.7mb    1.7mb
green open .watcher-history-7-2019.10.22 zfWVREXaT8GWMnKPZkQDbw 1 0  1590   0    1.7mb    1.7mb
green open .watcher-history-7-2019.10.23 RSGz4iW1TR6wKfgYUmVXfw 1 0   192   0  325.2kb  325.2kb
```

本机器当前已有的索引属于 Elastic 内部索引（使用 `kibana` 以及 `logstack` 创建文档数据后就会 Elastic 主动创建这些索引），现在我们要创建的索引
名称为 `theme`，不过我们不会调用 `POST /<Index` API，而是以添加文档数据的形式：

```bash
$ curl -XPOST "localhost:9200/theme/_doc?pretty&pretty" -H "Content-Type: application/json" -d'
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
  "_id" : "tlfYAW4BcNwnFQPz_yBb",
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

从返回的信息中可以看到：索引为 `theme`，类型为：`_doc`。并且 Elastic 主动创建了文档数据的 `id`：`tlfYAW4BcNwnFQPz_yBb`。

先不看数据是否创建成功，来看下索引是否成功创建：

```bash
$ curl -XGET "localhost:9200/_cat/indices?pretty&pretty"
```

```
green  open .watcher-history-7-2019.10.25 d4zSGgcKT1CDV6LGbTar8Q 1 0   154   0 285.1kb 285.1kb
green  open .watches                      Bptdm6_ASx69bYYJ495nvw 1 0     6   0  75.2kb  75.2kb
green  open .monitoring-es-6-2019.10.25   3-Km-OaRQZ20NVJN8tDxqA 1 0  2328 138   1.4mb   1.4mb
green  open .monitoring-alerts-6          Qb7JbFCNTy-l-lAyBiMADg 1 0     1   0  12.3kb  12.3kb
green  open .watcher-history-7-2019.10.21 rC7xOKLQTXaDLDLV_cQ05Q 1 0  2758   0   3.4mb   3.4mb
green  open .kibana                       G6D_RxaGR5CGeHKdGp_GKw 1 0     1   0     4kb     4kb
green  open .triggered_watches            9H35HOwtTUCGcjrH2lI3bQ 1 0     0   0 123.2kb 123.2kb
yellow open theme                         XFijZlW2RfmdWkJ53oDI0A 5 1     1   0   4.8kb   4.8kb
green  open .monitoring-es-6-2019.10.23   L_QsqDKFS6GibnIkYa2IKg 1 0  1548  18 893.9kb 893.9kb
green  open .watcher-history-7-2019.10.24 -708MKfjSvqvi6Q5Dj6cKQ 1 0   234   0 348.1kb 348.1kb
green  open .monitoring-es-6-2019.10.22   iygh5IibSbaFDxTE9RyQOA 1 0 14941  35   5.9mb   5.9mb
green  open .monitoring-es-6-2019.10.24   gh3afcsRTZmAnQTkUz8N3A 1 0  2823  77   1.7mb   1.7mb
green  open .watcher-history-7-2019.10.22 zfWVREXaT8GWMnKPZkQDbw 1 0  1590   0   1.7mb   1.7mb
green  open .watcher-history-7-2019.10.23 RSGz4iW1TR6wKfgYUmVXfw 1 0   192   0 325.2kb 325.2kb
```

在输出的索引列表中可以看到 `theme`，说明在向文档中插入数据时 Elastic 并不强制要求指定索引一定存在，这点与 `MongoDB` 类似。