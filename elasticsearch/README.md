# 安装 Elasticsearch 集群

## 数据目录

> 我们将执行文件和数据文件分开，创建以下目录

* ES 根目录 `/opt/elasticsearch`
* ES 安装目录 `/opt/elasticsearch/elasticsearch-7.8.1`
* ES 配置文件 `/opt/elasticsearch/elasticsearch-7.8.1/config/elasticsearch.yml`
* ES 配置文件(jvm) `/opt/elasticsearch/elasticsearch-7.8.1/config/jvm.options`
* ES 配置文件(log) `/opt/elasticsearch/elasticsearch-7.8.1/config/log4j.properties`
* 日志文件路径 `/opt/elasticsearch/logs`
* 数据文件路径 `/opt/elasticsearch/data`
* DUMP 文件路径  `/opt/elasticsearch/dump`
* 临时文件目录 `/opt/elasticsearch/temp`

## elasticsearch.yml

自定义数据和日志存储路径

```properties
path.data: /opt/elasticsearch/data
path.logs: /opt/elasticsearch/logs
```

## jvm.options

* 不能大于物理内存的 50%

```properties
-Xms4g
-Xmx4g
```

* 设置 dump 存储路径

```properties
-XX:HeapDumpPath=/opt/elasticsearch/dump
```

* 设置临时目录路径

```properties
-Djava.io.tmpdir=/opt/elasticsearch/temp
```

* 设置 JVM fatal error logs 路径

```properties
-XX:ErrorFile=/opt/elasticsearch/logs/hs_err_pid%p.log
```

* JDK9 GC 日志目录

```properties
9-:-Xlog:gc*,gc+age=trace,safepoint:file=/opt/elasticsearch/logs/gc.log:utctime,pid,tags:filecount=32,filesize=64m
```
