# 安装 Elasticsearch 集群

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

## gc.options

自定义 gc 日志输出到 /opt/elasticsearch/logs/gc.log
