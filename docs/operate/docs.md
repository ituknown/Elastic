# 前言

Elastic 文档数据的唯一性是通过二个方式来设定的：索引、类型。索引可以理解为关系型数据库的库名，而类型则可以理解为关系型数据库库中的数据表。

Elastic RestFul API 对文档数据访问遵循如下模`式：

```
<Rest Verb> /<Index>/<Type>/<ID>(?pretty&pretty)
```

- `Rest Verb`：请求类型，即 `GET` `PUT` `POST` 和 `DELETE`
- `Index`    ：索引，等同于关系型数据库的库名称
- `Type`     ：文档类型，等同于关系型数据库的数据表。
- `ID`       ：文档id，可以通过该值定义文档数据的id，另外也可以不指定使用 Elastic 的自增id。

**注意：** 在命令模式中有一个可选 `pretty`，该选项表示是否已更友好的凡方式展示数据，下问查询示例中会做比较说明。

# 查询文档数据

查询文档数据可以理解为查询关系型数据库中的数据。查询格式如下所示：

```
GET /Index/Type/ID(?pretty&pretty)
```

先来查询一条数据：

```bash
GET /customer/_doc/1?pretty&pretty
```

该命令的含义是查询的索引是 `customer`，类型是 `_doc`，查询的数据的 `id` 是 `1`。另外，增加了 `pretty` 参数是为了以更友好的方式展示。

现在来使用 `curl` 来测试：

```bash
$ curl -XGET "localhost:9200/customer/_doc/1?pretty&pretty"
```

查询结果如下：

```json
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "_source" : {
    "name" : "John Doe"
  }
}
```

在返回的 `JSON` 数据中有几个 key，先分别做一下说明：

- `_index`：查询的索引
- `_type`：查询的文档类型
- `_id`：查询的文档数据id
- `_version`：版本号
- `found`：`true` 表示检索到数据，如果没有查询到数据该值为 `false`。并且 `source` 字段将不返回
- `_source`：查询的数据集

现在来将 `pretty` 查询参数去掉，看下查询的数据格式：

```
$ curl -XGET "localhost:9200/customer/_doc/1"
```

格式如下：

```json
{"_index":"customer","_type":"_doc","_id":"1","_version":1,"found":true,"_source":
{
  "name": "John Doe"
}}
```

可以看到使用与不使用 `pretty` 的差异。

# 添加(替换)文档数据

在 Elastic 中，添加或替换数据都可以使用 `PUT` 和 `POST`。区别是，在使用 `PUT` 时必须指定或自定义一个 `ID`，如果不指定将会提示错误。而在使用
`POST` 时，如果不指定 `ID` 则表示使用 Elastic 内部自增 `ID` 来做为文档数据的 `ID`，如果指定 `ID` 则表示新增（如果指定的 `ID` 在数据库中不
存在）或替换原有数据：

```
PUT /Index/Type/ID(?pretty&pretty)
POST /Index/Type(/ID)(?pretty&pretty)
```

现在分别使用这两种方式添加一条数据。

## 自定义文档ID

现在使用 `PUT` 方式添加一条数据，用户名称为 `David`，指定的id为 `100`。

```bash
$ curl -XPUT "localhost:9200/customer/_doc/100?pretty&pretty" -H "Content-Type: application/json" -d'
{
  "name": "David"
}'
```

执行结果如下：

```
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "100",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}
```

**注意：** 在使用 `PUT` 方式增加文档数据时一定要指定文档ID，如果不指定会抛出异常（即执行失败）。

## 使用自增文档ID

使用 `POST` 方式添加一条自增 `id` 数据，用户名称为 `Alicia`。

```bash
$ curl -XPOST "localhost:9200/customer/_doc?pretty&pretty" -H "Content-Type: application/json" -d'
{
  "name": "Alicia"
}'
```

执行结果如下：

```
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "11Z-820BcNwnFQPzIuAp",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 2,
  "_primary_term" : 1
}
```

从上面的示例可以看到，两条数据都添加成功，并且使用 `POST` 方式添加的id为：`11Z-820BcNwnFQPzIuAp`。现在可以使用 `GET` 命令分别查询来验证是
否成功插入，查询一下自增id看看查询结果：

```bash
$ curl -XGET "localhost:9200/customer/_doc/11Z-820BcNwnFQPzIuAp?pretty"
```

```
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "11Z-820BcNwnFQPzIuAp",
  "_version" : 1,
  "found" : true,
  "_source" : {
    "name" : "Alicia"
  }
}
```

## 替换文档数据

替换文档数据 `PUT` 和 `POST` 都是可以的，命令格式如下：

```
PUT/POST /Index/Type/ID(?pretty&pretty)
```

**注意：** 替换文档数据一定要指定文档 ID。

现在，我们先增加一条数据：

```bash
$ curl -XPUT "localhost:9200/customer/_doc/1000?pretty&pretty" -H "Content-Type: application/json" -d'
{
  "name": "Bob",
  "age" : 18
}'
```

该文档增加了一条数据，包含名称与年龄。执行后使用 `GET`输出如下：

```bash
$ curl -XGET "localhost:9200/customer/_doc/1000?pretty&pretty"
```

```json
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1000",
  "_version" : 1,
  "found" : true,
  "_source" : {
    "name" : "Bob",
    "age" : 18
  }
}
```

现在再来执行如下指令试试结果：
```bash
$ curl -XPUT "localhost:9200/customer/_doc/1000?pretty&pretty" -H "Content-Type: application/json" -d'
{
  "name": "Linda"
}'       
```

该命令将 `id` 为 1000 的数据替换掉，如果成功替换再次获取该数据时应该不会存在 `age` 字段。现在执行后 `GET` 结果，输出如下：

```json
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1000",
  "_version" : 2,
  "found" : true,
  "_source" : {
    "name" : "Linda"
  }
}
```

可以看到文档 `id` 为 1000 的数据被成功替换。另外，当每次修改文档数据时 `_version` 值都会自增 1。

> **总结：** 从上面的示例中可以明显的看出 `PUT` 与 `POST` 的区别，以及应用场景。总的来说，`POST` 功能更强大。

# 修改文档数据

修改文档数据格式如下：

```
POST /<Index>/<Type>/<ID>/_update?pretty
```

以上面的文档中 ID 为 1000 的数据为例，先将 `Linda` 替换为 `Bob`：

```bash
$ curl -XPOST "localhost:9200/customer/_doc/1000/_update?pretty&pretty" -H "Content-Type: application/json" -d'
{
  "doc": { "name": "Bob" }
}'
```

执行后再获取该文档数据：

```bash
$ curl -XGET "localhost:9200/customer/_doc/1000?pretty&pretty"
```

```json
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1000",
  "_version" : 2,
  "found" : true,
  "_source" : {
    "name" : "Bob"
  }
}
```

可以看到，数据 `Linda` 被修改为 `Bob`。等等，真的是修改吗？万一是替换呢？再来，我们在来在该数据上增加一个年龄字段：

```bash
$ curl -XPOST "localhost:9200/customer/_doc/1000/_update?pretty&pretty" -H "Content-Type: application/json" -d'
{
  "doc": { "age": 18 }
}'
```

如果确实是修改，执行该命令后获取该文档数据应该包含 `name` 和 `age`，现在执行后获取结果，如下所示：

```json
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1000",
  "_version" : 3,
  "found" : true,
  "_source" : {
    "name" : "Bob",
    "age" : 18
  }
}
```

这说明确实是修改，并且修改成功。现在，我还想修改一下年龄，我想给年龄增加 10 岁。看下下面的语句：

```bash
$ curl -XPOST "localhost:9200/customer/_doc/1000/_update?pretty&pretty" -H "Content-Type: application/json" -d'
{
  "script": "ctx._source.age += 10"
}'
```

先不管 `script` 和 `ctx` 什么意思，先执行并获取结果。看年龄是否确实增加了 10 岁，输出结果如下：

```json
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1000",
  "_version" : 4,
  "found" : true,
  "_source" : {
    "name" : "Bob",
    "age" : 28
  }
}
```

可以看到，年龄已经变成了 28，现在再来说下这条语句。

Elastic 支持一种名为脚本（`script`）的语法。这中语法在修改文档时尤为适用，`ctx` 含义是当前文档的上下文。如上面输出的 JSON 格式数据就是文档上
下文。`ctx._source` 意思是获取文档数据，也就是我们所插入的文档数据。该文档中有一个 `name` 和 `age` 字段。我们获取了 `age` 字段并自增 10。

这是 Elastic 的脚本语法。另外，Elastic 还有许多其他修改脚本语法。具体换官网：[Elastic 文档数据修改脚本语法](https://www.elastic.co/guide/en/elasticsearch/reference/6.3/docs-update-by-query.html)

# 删除文档数据

文档数据删除命令格式如下：

```
DELETE /Index/Type/ID(?pretty&pretty)
```

现在来删除索引 `customer` 文档类型为 `_doc` 文档 ID 为 100 的数据：

```bash
$ curl -XDELETE "localhost:9200/customer/_doc/100?pretty&pretty"
```

执行结果如下：

```json
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "100",
  "_version" : 4,
  "result" : "deleted",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 9,
  "_primary_term" : 1
}
```

`result` 值为 `deleted`，表示删除成功。