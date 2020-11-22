# 前言

Elasticsearch 分为解压版（`tar.gz`）以及包管理安装版两个安装版本，本篇会分别进行介绍。

Elasticsearch 更新周期很快，而且每个 Release 版本都会增加一些特性，所以安装时建议安装最新 Release 版本，可以点击链接查看所有 Release 版本列表：[Release 版本文档列表](https://www.elastic.co/guide/en/elastic-stack/index.html)

每个 Release 版本都对应 Elasticsearch、Kibana、Logstash、Beats、APM Server 以及 Elasticsearch Hadoop 等产品。

本文我们要安装的产品是 Elasticsearch，选择对应的产品后进入安装介绍页面即可。在这个页面就会看到对应各操作系统以及各自的安装方式，比如 Linux 就提供了基于解压版本的 `tar.gz` 文件以及各种发行 Linux 的包管理安装方式。

相比将而言，包管理安装版更加简单，现在来一一介绍。

# 解压版安装

确定好版本后我们进入对应的解压版安装方式界面，比如 7.5 版本：https://www.elastic.co/guide/en/elasticsearch/reference/7.5/targz.html

这个页面提供了基于解压版的安装方式以及命令，我们直接拷贝就可以运行。

## 下载解压文件：

确定好安装目录（这里我要安装的目录是 `/opt/elastic`）进入目录后执行 `wget` 命令进行下载压缩文件。

```bash
$ pwd
/opt/elastic

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

| shasum: command not found？                                  |
| :----------------------------------------------------------- |
| 该问题原因是 `shasum` 在 CentOS 上被称为 `sha1sum`。该文件在 `/bin` 目录下，我们直接使用软连接进行命名为 `shasum` 即可：`sudo ln -s /bin/sha1sum /bin/shasum` |

| shasum: invalid option -- 'a'？                              |
| :----------------------------------------------------------- |
| 该问题原因是缺少软件包 `perl-Digest-SHA`，使用 `YUM` 安装即可：`sudo yum install -y perl-Digest-SHA` |

## 安装与用户配置

一切准备就绪之后就可以进行加压安装了：

```bash
tar -xzvf elasticsearch-7.5.2-linux-x86_64.tar.gz
```

之后建立一个软连接（可选操作，我是为了方便指定不同版本才创建软连接的）：

```bash
sudo ln -s elasticsearch-7.5.2 es
```

Elasticsearch 与所有软件一样，可执行文件都在 bin 目录下，配置文件在 config 目录下。

```bash
$ cd es
$ ls
LICENSE.txt  NOTICE.txt  README.asciidoc  bin  config  jdk  lib  logs  modules  plugins
```

基本上到这里 Elasticsearch 就安装好了，不过我们还可以继续设置一下环境变量：

```bash
elasticsudo vim /etc/profile

export ES_HOME=/opt/elastic/es
export PATH=$PATH:/$ES_HOME/bin

source /etc/profile
```

需要说明的是，Elasticsearch 并不支持 root 用户登录，所以我们需要创建一个新用户（也可以直接使用一个已有的非 root 用户）：

```bash
elasticsudo groupadd es
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

Elasticsearch 的所有配置文件都在 config 目录下：

```bash
$ ls config/
elasticsearch.keystore  jvm.options        role_mapping.yml  users
elasticsearch.yml       log4j2.properties  roles.yml         users_roles
```

其中最重要的两个配置文件是 `jvm.options` 和 `elasticsearch.yml`。从配置文件中也可以看出来，Elasticsearch 依赖于 JVM。当然，Elasticsearch 安装包中内置的有推荐的 JVM 版本。如果不想使用内置的也可以自己在环境中进行安装一个，然后在环境变量中配置 `JAVA_HOME` 即可。

在对 Elasticsearch 不熟悉的情况下我们只需要知道 `jvm.options` 中默认配置的内存大小是1g：

```
-Xms1g
-Xmx1g
```

所以，如果当前机器没有足够的内存的话记得进行相应修改。修改时要注意，堆最小内存（`Xms`）与最大内存（`Xmx`）要保持一致，原因在后续文章中会进行说明。

另外，Elasticsearch 默认的 web 端口号是 9200。包括数据存储目录以及日志存储目录、集群名称都在 `elasticsearch.yml` 文件中进行配置。

这里我就只修改下如下两个配置：

```properties
path.data: /var/es/data
path.logs: /var/es/logs
```

这两个属性默认是注释的，`path.data` 指的是数据的存储目录，`path.logs` 指的是日志的输出目录。

这两个值得目录是随意的，可以指定到任意目录，注意使用绝对路径。（修改这两个配置原因是可以明确控制数据目录，之后如果进行升级也不用担心数据会丢失。当然，你也可以不修改使用默认的就好。）

在 `/var` 目录下创建一个 `es` 文件夹并将所属用户和读写执行权限（其他用户只有读权限）都设置给新创建的 `es` 用户：

```bash
sudo mkdir -p /var/es
sudo chown -R es:es /var/es
sudo chmod 774 /var/es
```

另外，Elasticsearch 有几个硬性要求。系统允许打开的线程数至少值为 4096，允许操作的文件句柄至少为 65535。

所以，我们需要进行设置一下。修改 `/etc/security/limits.conf` 文件

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

现在就可以使用进程查看命令查看是否启动成功，示例：

```bash
$ ps -ef | grep elastic | grep -v grep

es         8748      1  1 14:14 pts/0    00:02:11 //opt/elastic/es/jdk/bin/java -Des.networkaddress.cache.ttl=60 -Des.networkaddress.cache.negative.ttl=10 -XX:+AlwaysPreTouch -Xss1m -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Djna.nosys=true -XX:-OmitStackTraceInFastThrow -Dio.netty.noUnsafe=true -Dio.netty.noKeySetOptimization=true -Dio.netty.recycler.maxCapacityPerThread=0 -Dio.netty.allocator.numDirectArenas=0 -Dlog4j.shutdownHookEnabled=false -Dlog4j2.disable.jmx=true -Djava.locale.providers=COMPAT -Xms1g -Xmx1g -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -Djava.io.tmpdir=/tmp/elasticsearch-603325604387781353 -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=data -XX:ErrorFile=logs/hs_err_pid%p.log -Xlog:gc*,gc+age=trace,safepoint:file=logs/gc.log:utctime,pid,tags:filecount=32,filesize=64m -XX:MaxDirectMemorySize=536870912 -Des.path.home=//opt/elastic/es -Des.path.conf=//opt/elastic/es/config -Des.distribution.flavor=default -Des.distribution.type=tar -Des.bundled_jdk=true -cp //opt/elastic/es/lib/* org.elasticsearch.bootstrap.Elasticsearch -d
es         8764   8748  0 14:14 pts/0    00:00:00 /opt/elastic/es/modules/x-pack-ml/platform/linux-x86_64/bin/controller
```

现在来测试一下运行状态：

```bash
$ curl -X GET "localhost:9200/?pretty"
{
  "name" : "es-1",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "iCVXpDPITbe58j3-QUdGMA",
  "version" : {
    "number" : "7.5.2",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "8bec50e1e0ad29dad5653712cf3bb580cd1afcdf",
    "build_date" : "2020-01-15T12:11:52.313576Z",
    "build_snapshot" : false,
    "lucene_version" : "8.3.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

如果得到类似上面的输出即表示正常运行，到此基于解压版的安装方式就说完了。



# 包管理安装版

包管理安装版安装起来比解压版安装的更加简单。同样的，官网提供了基于包管理的安装介绍。比如 7.5 版本：

https://www.elastic.co/guide/en/elasticsearch/reference/7.5/rpm.html

## 下载签名key

首先我们需要使用 rpm 命令将签名key下载到本地：

```bash
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```

## 创建 YUM 源

在 `/etc/yum.repo.d` 目录下创建一个 Ealsticsearch 的源文件，文件名随意，比如 `Elasticsearch.repo`。内容如下：

```
[elasticsearch]
name=Elasticsearch repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=0
autorefresh=1
type=rpm-md
```

当然，上面是的 Elasticstack 官网提供的 repo。这个只能给那些对自己网络够自信的或有梯子的同学而言的。我不够自信，所以我选择清华镜像站：

```
[elasticsearch]
name=Elasticsearch repository for 7.x packages
baseurl=https://mirrors.tuna.tsinghua.edu.cn/elasticstack/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=0
autorefresh=1
type=rpm-md
```

| Note                                                         |
| :----------------------------------------------------------- |
| 上面的两个 repo 配置没有什么区别，功能一样。只是由于 Elastic 是国外产品，所以下载的会很慢。不过我们大中华地区有相应的镜像站，鼎鼎大名的可能就是 [清华大学镜像站](https://mirrors.tuna.tsinghua.edu.cn) 了，当然你也可以选择 [网易163](http://mirrors.163.com/) 和 [阿里镜像站](http://mirrors.aliyun.com/)。 |

## 安装

上面都设置好之后安装起来就很简单了，只需一条命令：

```bash
sudo yum install -y --enablerepo=elasticsearch elasticsearch
```

静静的等待安装，安装过程可能需要个几分钟，这个就要看个人网络了。


## 关于配置文件

安装成功后就来说下包管理安装的配置文件。这里基本上与解压版雷同，可以直接参考解压版的修改方式。区别主要是 Elasticsearch 的配置文件，使用包管理安装的配置文件都存储在 `/etc/elasticsearch` 目录下：

```bash
$ ls /etc/elasticsearch
elasticsearch.keystore  jvm.options    log4j2.properties  roles.yml  users_roles
elasticsearch.yml       jvm.options.d  role_mapping.yml   users
```

而且，除了 JVM 参数我们也不需要做额外的修改。甚至启动参数也根据需要做了修改，比如允许最大打开的文件句柄数等等。可以直接在启动服务文件中查看：

```bash
cat /usr/lib/systemd/system/elasticsearch.service
```

有关包管理的数据存储于配置可以参考 7.5 版本的官网说明：https://www.elastic.co/guide/en/elasticsearch/reference/7.5/rpm.html#rpm-layout。

## 系统管理

现在我们就可以直接使用系统管理命令来控制 Elasticsearch。先来看下你的系统使用的是 `init` 还是 `systemd`：

```bash
$ ps -p 1
   PID TTY          TIME CMD
     1 ?        00:00:03 systemd
```

看输出中的 CMD 信息就能得到自己系统的管理工具。下面的说明根据自己系统进行选择。

### 开机自启

Elasticsearch 默认是不启动的，我们可以选择设置为开机自启。

**`init` 用户：**

```bash
sudo chkconfig --add elasticsearch
```

**`systemd` 用户：**

```bash
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch.service
```

### 服务启动

**`init` 用户：**

```bash
sudo -i service elasticsearch start
```

**`systemd` 用户：**

```bash
sudo systemctl start elasticsearch.service
```

### 服务停止

**`init` 用户：**

```bash
sudo -i service elasticsearch stop
```

**`systemd` 用户：**

```bash
sudo systemctl stop elasticsearch.service
```

### 查看服务状态

**`init` 用户：**

```bash
sudo -i service elasticsearch status
```

**`systemd` 用户：**

```bash
sudo systemctl status elasticsearch.service
```

## 运行测试

服务启动后就来看下服务状态：

```bash
$ sudo systemctl status elasticsearch
● elasticsearch.service - Elasticsearch
   Loaded: loaded (/usr/lib/systemd/system/elasticsearch.service; enabled; vendor preset: disabled)
   Active: active (running) since Sun 2020-11-22 13:11:31 CST; 2min 4s ago
     Docs: https://www.elastic.co
 Main PID: 9920 (java)
   CGroup: /system.slice/elasticsearch.service
           ├─ 9920 /usr/share/elasticsearch/jdk/bin/java -Xshare:auto -Des.networkaddress.cache.ttl=60 -Des.networka...
           └─10109 /usr/share/elasticsearch/modules/x-pack-ml/platform/linux-x86_64/bin/controller

Nov 22 13:11:10 es-2 systemd[1]: Starting Elasticsearch...
Nov 22 13:11:31 es-2 systemd[1]: Started Elasticsearch.
```

看输出一切正常，再来使用 curl 请求测试一下：

```bash
$ curl -X GET "localhost:9200/?pretty"
{
  "name" : "es-2",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "4CHrkczxRySxs89XCyFGRg",
  "version" : {
    "number" : "7.10.0",
    "build_flavor" : "default",
    "build_type" : "rpm",
    "build_hash" : "51e9d6f22758d0374a0f3f5c6e8f3a7997850f96",
    "build_date" : "2020-11-09T21:30:33.964949Z",
    "build_snapshot" : false,
    "lucene_version" : "8.7.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

如果得到类似上面的输出即表示正常运行。

