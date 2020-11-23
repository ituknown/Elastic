Elasticsearch 服务对外提供了基于 Rest 风格的 API，本章节主要介绍 Elasticsearch 相关 Rest API。

在 Elasticsearch 中，所有的获取操作都使用 `GET`，所有的删除操作都使用 `DELETE`，所有的新增或修改操作可以使用 `PUT` 或 `POST`。

比如请求：

```bash
PUT /<Indices>(?v&pretty)
```

请求类型为 `PUT`，后面的 `<Indices>` 指的是索引，对于该请求没有显示指定请求 IP 意思就是根据自己的机器决定。

比如是你本地的 Elasticsearch 服务器，那么该请求就是：

```
PUT localhost:9200/<Indeices>(?v&pretty)
```

如果是远程服务，就将对应的 IP 替换为远程机器 IP 即可。后面的 `?v&pretty` 指的是可选参数，`v` 值得是 `verbose` 的意思，对于获取数据（`GET` 请求）返回类似于 Table 形式的数据建议使用 `v` 。而对于使用 JSON 形式的请求（无论查询还是修改）就使用 `pretty` 参数。

另外，在请求时可能会遇到使用 localhost 或 127.0.0.1 能行但是局域网 IP 却不行的问题。原因是没有在配置文件  `elasticsearch.yml` 中显示的设置属性 `network.host`。该属性默认是注释的，即默认使用 localhost 或 127.0.0.1。如果想使用局域网 IP 的形式可以考虑将值设置为对应局域网 IP 即可，比如：

```
network.host: 192.168.0.128
```

重启后再次尝试使用基于 IP 的形式应该就没问题了。
