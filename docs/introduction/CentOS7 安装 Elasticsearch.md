# 前言

Elasticsearch 分为解压版（`tar.gz`）以及包管理安装版（二进制安装版）两个安装版本，本篇会分别进行介绍，不过在实际使用中我更喜欢使用解压版。

Elasticsearch 更新周期很快，而且每个 Release 版本都会增加一些特性，所以安装时建议安装最新 Release 版本，可以点击下面的链接查看所有 Release 版本列表：

Release 版本文档列表：https://www.elastic.co/guide/en/elastic-stack/index.html

每个 Release 版本都对应 Elasticsearch、Kibana、Logstash、Beats、APM Server 以及 Elasticsearch Hadoop 等产品。

本文我们要安装的产品是 Elasticsearch，选择对应的产品后进入安装介绍页面即可。在这个页面就会看到对应各操作系统以及各自的安装方式，比如 Linux 就提供了基于解压版本的 `tar.gz` 文件以及各种发行 Linux 的包管理安装方式。

现在来一一介绍。

# 解压版安装

确定好版本后我们进入对应的解压版安装方式界面，比如 7.5 版本：https://www.elastic.co/guide/en/elasticsearch/reference/7.5/targz.html

这个页面提供了基于解压版的安装方式以及命令，我们直接拷贝就可以运行。

## 下载解压文件：

确定好安装目录，这里将我要安装的目录是 `/opt/elastic`，进入目录后执行如下命令进行下载压缩文件。

```bash
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.5.2-linux-x86_64.tar.gz
```

| 注意                                                           |
|:-------------------------------------------------------------|
| 由于 Elasticsearch 是国外产品，所以直接使用命令行下载可能会很慢。建议使用迅雷等下载工具将解压文件下载到本地之后再上传到服务器。 |

## 文件签名校验

将解压文件下载到服务器之后继续下载校验文件，其实直接从官网下载可以方式使用。但是为了防止在网络传输过程中被植入木马我们还是小心为妙，使用如下命令进行下载校验文件。

```bash
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.5.2-linux-x86_64.tar.gz.sha512
```

之后就继续使用命令进行校验 SHA，保证我们下载的文件是纯天然无公害不含任何防腐剂的环保的官方产品：

```bash
shasum -a 512 -c elasticsearch-7.5.2-linux-x86_64.tar.gz.sha512 
```

如果输出类似如下信息即表示文件正常：

```
elasticsearch-{version}-linux-x86_64.tar.gz: OK
```

**执行该命令可能会遇到如下问题：**

| shasum: command not found                                    |
|:------------------------------------------------------------ |
| 该问题原因是 `shasum` 在 CentOS 上被称为 `sha1sum`。该文件在 `/bin` 目录下，我们直接使用软连接进行命名为 `shasum` 即可：`sudo ln -s /bin/sha1sum /bin/shasum` |

| shasum: invalid option -- 'a'                                |
|:------------------------------------------------------------ |
| 该问题原因是缺少软件包 `perl-Digest-SHA`，使用 `YUM` 安装即可：`sudo yum install -y perl-Digest-SHA` |

## 安装与用户配置

一切准备就绪之后就可以进行加压安装了：

```bash
$ pwd
/opt/elastic

tar -xzvf elasticsearch-7.5.2-linux-x86_64.tar.gz
```

之后建立一个软连接（可选操作，我是为了方便才创建软连接的）：

```bash
sudo ln -s elasticsearch-7.5.2 es
```

Elasticsearch 与所有软件一样，可执行文件都在 bin 目录下，配置文件在 config 目录下。

基本上到这里 Elasticsearch 就安装好了，不过我们还可以继续设置一下环境变量：

```bash
sudo vim /etc/profile

export ES_HOME=/opt/elastic/es
export PATH=$PATH:/$ES_HOME/bin

source /etc/profile
```

需要说明的是，Elasticsearch 并不支持 root 用户登录，所以我们需要创建一个新用户（也可以直接使用一个已有的非 root 用户）：

```bash
sudo groupadd es
sudo useradd -g es es
```

然后将整个顶级 elastic 文件都归属到该用户下：

```bash
$ pwd
/opt/elastic

sudo chown -R es:es /opt/elastic
sudo chmod 774 /opt/elastic
```

## 系统参数与配置文件修改

因为刚安装，所以我们需要简单的修改下配置文件，编辑 config 目录下的 `elasticsearch.yml`。修改如下两个配置属性：

```properties
path.data: /var/es/data
path.logs: /var/es/logs
```

这两个属性默认是注释的，`path.data` 指的是数据的存储目录，`path.logs` 指的是日志的输出目录。

这两个值得目录是随意的，可以指定到任意目录，注意使用绝对路径。

然后在 `/var` 目录下创建一个 `es` 文件夹并将所属用户和读写执行权限（其他用户只有读权限）都设置给新创建的 `es` 用户：

```bash
sudo mkdir -p /var/es
sudo chown -R es:es /var/es
sudo chmod 774 /var/es
```

另外，Elasticsearch 有几个硬性要求。系统允许打开的线程数至少值为 4096，允许操作的文件句柄至少为 65535。

所以，我们需要进行设置一下。修改 `etc/security/limits.conf` 文件

```bash
sudo vim /etc/security/limits.conf
```

在文件末尾添加或修改如下内容，除了已有的注释默认是空配置文件。如果有的话看这些值满足 Elasticsearch 硬性要求不，已满足就不必修改了：

```
* soft nproc 4096
* hard nproc 4096

* soft nofile 65535
* hard nofile 65535
```

其中 `*` 表示的是所有用户，如果指定到特定用户可以将 `*` 替换为具体用户名，比如 `es` 用户：

```
es soft nproc 4096
```

另外 `soft` 和 `hard` 分别指的是软件和硬件。

`nproc` 指的是线程，`nofile` 指的是文件句柄。至于后面的数值分别指最大允许打开的线程数和最大允许操作的文件句柄数。

## 运行 Elasticsearch

运行命令如下所示：

```bash
$ES_HOME/bin/elasticsearch -d -p <pid_file>
```

如果已经配置了环境变量可以直接使用如下命令：

```bash
elasticsearch -d -p <pid_file>
```

其中 `-d` 指的是允许在后台运行，`-p` 指的是将运行成功的进程 id 放到执行文件中，这两个参数都是可选的。

| 注意                                                         |
|:------------------------------------------------------------ |
| Elasticsearch 基于 JVM（默认使用内置，我们先暂时不管直接使用内置的即可），在启动时的默认堆大小为 1g。所以，如果操作系统内存无法满足默认设置修改 config 文件夹下的 `jvm.options` 文件。其他参数先不管，直接修改 `-Xms1g` 和 `-Xmx1g` 为合适的内存大小。堆内存最大值与最小值一定要相同，原因在之后的文章中会进行说明。 |

现在就可以使用进程查看命令查看是否启动成功，示例：

```bash
$ ps -ef | grep elastic | grep -v grep

es         8748      1  1 14:14 pts/0    00:02:11 //opt/elastic/es/jdk/bin/java -Des.networkaddress.cache.ttl=60 -Des.networkaddress.cache.negative.ttl=10 -XX:+AlwaysPreTouch -Xss1m -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Djna.nosys=true -XX:-OmitStackTraceInFastThrow -Dio.netty.noUnsafe=true -Dio.netty.noKeySetOptimization=true -Dio.netty.recycler.maxCapacityPerThread=0 -Dio.netty.allocator.numDirectArenas=0 -Dlog4j.shutdownHookEnabled=false -Dlog4j2.disable.jmx=true -Djava.locale.providers=COMPAT -Xms1g -Xmx1g -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -Djava.io.tmpdir=/tmp/elasticsearch-603325604387781353 -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=data -XX:ErrorFile=logs/hs_err_pid%p.log -Xlog:gc*,gc+age=trace,safepoint:file=logs/gc.log:utctime,pid,tags:filecount=32,filesize=64m -XX:MaxDirectMemorySize=536870912 -Des.path.home=//opt/elastic/es -Des.path.conf=//opt/elastic/es/config -Des.distribution.flavor=default -Des.distribution.type=tar -Des.bundled_jdk=true -cp //opt/elastic/es/lib/* org.elasticsearch.bootstrap.Elasticsearch -d
es         8764   8748  0 14:14 pts/0    00:00:00 /opt/elastic/es/modules/x-pack-ml/platform/linux-x86_64/bin/controller
```



# 包管理安装版（二进制安装版）

