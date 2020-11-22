# 前言

Elasticsearch 文档数据的唯一性是通过二个方式来设定的：索引 + 类型。索引可以理解为关系型数据库的库名，而类型则可以理解为关系型数据库库中的数据表。

Elasticsearch RestFul API 对文档数据访问遵循如下模式：

```
<Rest Verb> /<Index>/<Type>/<ID>(?v&pretty)
```

`Rest Verb` 指的是请求类型，，即 `GET` 、`PUT`、 `POST` 和 `DELETE`。

`Index` 指的是索引，等同于关系型数据库的库名称。

`Type` 指的是文档类型，等同于关系型数据库的数据表。

`ID` 指的是文档具体数据的 ID，可以通过该值定义文档数据的 ID。另外如果不指定将默认使用 Elasticsearch 的自增 ID。

# 查询文档数据

查询文档数据可以理解为查询关系型数据库中的数据。查询格式如下所示：

```
GET /Index/Type/ID(?v&pretty)
```

比如查询 twitter 索引下类型为 User 的文档用户的 ID 为1的数据：

```bash
GET /twitter/user/1?pretty
```

查询示例：

```bash
$ curl -XGET "localhost:9200/twitter/user/1?pretty"
```

查询结果如下：

```json
{
  "_index" : "twitter",
  "_type" : "user",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 0,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "username" : "Bon"
  }
}
```

在返回的 `JSON` 数据中有几个 key，先分别做一下说明：

| KEY        | 释义                                                         |
| ---------- | ------------------------------------------------------------ |
| `_index`   | 查询的索引                                                   |
| `_type`    | 查询的文档类型                                               |
| `_id`      | 查询的文档数据 ID                                            |
| `_version` | 数据版本号，在每次修改后都会自增1                            |
| `found`    | `true` 表示检索到数据，如果没有查询到数据该值为 `false`，并且 `source` 字段将不返回 |
| `_source`  | 查询的数据集，为 JSON 格式。如果 `found` 值为 false，该字段不会返回 |

| Note                                                         |
| :----------------------------------------------------------- |
| 在查询时返回的数据都是 JSON 格式，所以在实际使用中建议可选参数使用 `pretty` 。 |

# 添加(替换)文档数据

添加或替换数据都可以使用 `PUT` 和 `POST`。区别是，使用 `PUT` 时必须显示的指定一个 ID，如果不指定将会提示错误。而在使用
`POST` 时，如果不指定 ID 则表示使用 Elasticsearch 内部自增 ID 来作为文档数据的 ID，如果指定 ID 则表示新增（如果指定的 `ID` 在数据库中不存在）或替换原有数据：

```
PUT /Index/Type/ID(?v&pretty)
POST /Index/Type</ID>(?v&pretty)
```

现在分别使用这两种方式添加一条数据。

## 自定义文档ID

现在使用 `PUT` 方式向 twitter 索引中添加一条数据，用户名称为 David，指定的 ID 为 2：

```bash
curl -XPUT "localhost:9200/twitter/user/2?pretty" -H "Content-Type: application/json" -d'
{
  "name": "David"
}'
```

执行结果如下：

```json
{
  "_index" : "twitter",
  "_type" : "user",
  "_id" : "2",
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

**注意：** 在使用 `PUT` 方式增加文档数据时一定要指定文档 ID，如果不指定会抛出异常（即执行失败）。

## 使用自增文档ID

使用 `POST` 方式添加一条自增 ID 数据，用户名称为 Alicia。

```bash
curl -XPOST "localhost:9200/twitter/user?pretty" -H "Content-Type: application/json" -d'
{
  "name": "Alicia"
}'
```

执行结果如下：

```json
{
  "_index" : "twitter",
  "_type" : "user",
  "_id" : "5JBq73UB5Tq_xcodMhFf",
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

从上面的示例可以看到，数据添加成功了，并且使用 `POST` 方式添加的 ID 为：`5JBq73UB5Tq_xcodMhFf`。现在可以使用 `GET` 命令查询来验证是否成功插入，查询一下自增id看看查询结果：

```bash
curl -XGET "localhost:9200/twitter/user/5JBq73UB5Tq_xcodMhFf?pretty"
```

```json
{
  "_index" : "twitter",
  "_type" : "user",
  "_id" : "5JBq73UB5Tq_xcodMhFf",
  "_version" : 1,
  "_seq_no" : 2,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "Alicia"
  }
}
```

## 替换文档数据

替换文档数据与新增操作相同，区别是一定要指定一个 ID。命令格式如下：

```
PUT/POST /Index/Type/ID(?v&pretty)
```

现在，我们先增加一条数据：

```bash
curl -XPUT "localhost:9200/twitter/user/3?pretty" -H "Content-Type: application/json" -d'
{
  "name": "Bob",
  "age" : 18
}'
```

向文档中增加一条数据，包含名称与年龄。执行后使用 `GET`输出如下：

```bash
curl -XGET "localhost:9200/twitter/user/3?pretty"
```

```json
{
  "_index" : "twitter",
  "_type" : "user",
  "_id" : "3",
  "_version" : 1,
  "_seq_no" : 3,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "Bob",
    "age" : 18
  }
}
```

现在再来执行如下指令试试结果：
```bash
curl -XPUT "localhost:9200/twitter/user/3?pretty" -H "Content-Type: application/json" -d'
{
  "name": "Linda"
}'       
```

该命令表示将之前新增的数据替换掉，如果成功替换那么再次获取该数据时应该不会存在 `age` 字段。现在执行后 `GET` 结果，输出如下：

```json
{
  "_index" : "twitter",
  "_type" : "user",
  "_id" : "3",
  "_version" : 2,
  "_seq_no" : 4,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "Linda"
  }
}
```

可以看到文档 `id` 为 3 的数据被成功替换。另外，当每次修改文档数据时 `_version` 值都会自增 1。

| 总结                                                         |
| :----------------------------------------------------------- |
| 从上面的示例中可以明显的看出 `PUT` 与 `POST` 的区别，以及应用场景。总的来说，`POST` 功能更强大。 |

# 修改文档数据

修改文档数据格式如下：

```
POST /<Index>/<Type>/<ID>/_update?pretty
```

以上面的文档中 ID 为 3 的数据为例，先将 `Linda` 替换为 `Bob`：

```bash
curl -XPOST "localhost:9200/twitter/user/3/_update?pretty" -H "Content-Type: application/json" -d'
{
  "doc": { "name": "Bob" }
}'
```

执行后再获取该文档数据：

```bash
curl -XGET "localhost:9200/twitter/user/3?pretty"
```

```json
{
  "_index" : "twitter",
  "_type" : "user",
  "_id" : "3",
  "_version" : 3,
  "_seq_no" : 5,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "Bob"
  }
}
```

可以看到，数据 `Linda` 被修改为 `Bob`。等等，真的是修改吗？万一是替换呢？再来，我们在来在该数据上增加一个年龄字段：

```bash
curl -XPOST "localhost:9200/twitter/user/3/_update?pretty" -H "Content-Type: application/json" -d'
{
  "doc": { "age": 18 }
}'
```

如果确实是修改，执行该命令后获取该文档数据应该包含 `name` 和 `age`，现在执行后获取结果，如下所示：

```json
{
  "_index" : "twitter",
  "_type" : "user",
  "_id" : "3",
  "_version" : 4,
  "_seq_no" : 5,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "Bob",
    "age" : 18
  }
}
```

这说明确实是修改，并且修改成功。现在，我还想修改一下年龄，我想给年龄增加 10 岁。看下下面的语句：

```bash
curl -XPOST "localhost:9200/twitter/user/3/_update?pretty" -H "Content-Type: application/json" -d'
{
  "script": "ctx._source.age += 10"
}'
```

先不管 `script` 和 `ctx` 什么意思，先执行并获取结果。看年龄是否确实增加了 10 岁，输出结果如下：

```json
{
  "_index" : "twitter",
  "_type" : "user",
  "_id" : "3",
  "_version" : 5,
  "_seq_no" : 6,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "Bob",
    "age" : 28
  }
}
```

可以看到，年龄已经变成了 28，现在再来说下这条语句。

Elasticsearch 支持一种名为脚本（`script`）的语法。这中语法在修改文档时尤为适用，`ctx` 含义是当前文档的上下文。如上面输出的 JSON 格式数据就是文档上下文。`ctx._source` 意思是获取文档数据，也就是我们所插入的文档数据。该文档中有一个 `name` 和 `age` 字段。我们获取了 `age` 字段并自增 10。

这是 Elasticsearch 的脚本语法。另外，Elasticsearch 还有许多其他修改脚本语法。具体换官网：[Elastic 文档数据修改脚本语法](https://www.elastic.co/guide/en/elasticsearch/reference/6.3/docs-update-by-query.html)

# 删除文档数据

文档数据删除命令格式如下：

```
DELETE /Index/Type/ID(?v&pretty)
```

现在来删除索引 `twitter` 文档类型为 `user`  ID 为 3 的数据：

```bash
curl -XDELETE "localhost:9200/twitter/user/3?pretty"
```

执行结果如下：

```json
{
  "_index" : "twitter",
  "_type" : "user",
  "_id" : "3",
  "_version" : 10,
  "result" : "deleted",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 12,
  "_primary_term" : 1
}
```

`result` 值为 `deleted`，表示删除成功。

# 后记

虽然 Elasticsearch 的所有数据都存储在某个索引下的某个类型文档中，但是在 v7.5 中我测试发现无法在同一个索引中同时创建多个类型的文档。

比如我当前 twitter 索引下有个 user 类型的文档。我现在想要再创建一个 customer 类型的文档数据：

```bash
curl -XPUT "localhost:9200/twitter/customer/1?pretty" -H "Content-Type: application/json" -d'
{
  "name": "David"
}'
```

结果就提示异常：

```
Rejecting mapping update to [twitter] as the final mapping would have more than 1 type: [user, customer]
```

也就是说一个索引下只能创建一个类型文档，到这里你是不是就觉得奇怪了？这是因为在 v6.8 版本做了改变，只允许创建单类型文档，具体见官方文档说明：https://www.elastic.co/guide/en/elasticsearch/reference/6.8/mapping.html#mapping