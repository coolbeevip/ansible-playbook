# 系统设置

## 禁用交换分区

```shell
swapoff -a
sed -i "s/^.*swap/#&/g" /etc/fstab
```

## 设置 vm.swappiness 和虚拟内存

```shell
echo "vm.swappiness=1" >> /etc/sysctl.config
echo "vvm.max_map_count=262144" >> /etc/sysctl.config
sysctl -p
```

## 设置文件句柄

编辑 `/etc/security/limits.conf`

```shell
* soft nofile 65535
* hard nofile 65535
* soft nproc 65535
* hard nproc 65535
```

# ElasticSearch

## elasticsearch.yml

自定义数据和日志存储路径

```properties
path.data: /data01/elasticsearch/data
path.logs: /data01/elasticsearch/logs
```

写缓冲比例（堆外空间）

```properties
indices.memory.index_buffer_size: 50%
```

读缓冲比例（堆外空间）

```properties
indices.queries.cache.size: 30%
```

线程队列空间

```properties
thread_pool.search.queue_size: 10000
thread_pool.get.queue_size: 1000
thread_pool.write.queue_size: 10000
```

## jvm.options

* 不能大于物理内存的 50%

> JVM 配置 8G + 预留空闲 8G（空闲的 ES 也会使用）

```properties
-Xms8g
-Xmx8g
```

* 设置 dump 存储路径

```properties
-XX:HeapDumpPath=/data01/elasticsearch/dump
```

* 设置临时目录路径

```properties
-Djava.io.tmpdir=/data01/elasticsearch/temp
```

* 设置 JVM fatal error logs 路径

```properties
-XX:ErrorFile=/data01/elasticsearch/logs/hs_err_pid%p.log
```

* JDK9 GC 日志目录

```properties
9-:-Xlog:gc*,gc+age=trace,safepoint:file=/data01/elasticsearch/logs/gc.log:utctime,pid,tags:filecount=32,filesize=64m
```

## 索引配置

禁用集群自动均衡

```shell
curl -X PUT "10.1.207.180:9200/_cluster/settings?flat_settings=true&pretty" -H 'Content-Type: application/json' -d'
{
  "transient" : {
    "cluster.routing.rebalance.enable" : "none"
  }
}
'
```

禁用副本、刷新间隔、日志写入策略等（首次导入时配置），导入后恢复以下参数

```shell
curl -X PUT "10.1.207.180:9200/onu_zl_test/_settings?pretty" -H 'Content-Type: application/json' -d'
{
  "index": {
    "number_of_replicas": 0,
    "refresh_interval": "300s",
    "translog": {
      "durability": "async",
      "sync_interval": "120s",
      "flush_threshold_size": "2048mb"
    },
    "merge": {
      "scheduler": {
        "max_thread_count": 1
      }
    }
  }
}
'
```

## 基准测试

* 按照以上优化方式部署的 3 节点集群
* 批量写入 onu 表数据（984万+，7.93GB），耗时 4 分钟，吞吐率 30MB/s

```
==========================================
ElasticSearch Import CSV
----------------- params ------------------
index: onu_zl_test
id: 0
csv count: 9848742
csv size: 7.93 GB
batch: 10000
concurrency: 16
----------------- output ------------------
succeed : 9848742
fails : 0
total time: 04 min, 25 sec
parse cvs time: 44 min, 21 sec
average size per: 0.84 KB
throughput records: 37165 ops/s
throughput size: 30.65 MB ops/s
==========================================
refresh [onu_zl_test] ... status OK time 31897ms
```
