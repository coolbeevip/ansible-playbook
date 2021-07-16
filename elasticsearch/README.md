# Install Elasticsearch Cluster

## Installation directory

You can define these variables in file `vars_elasticsearch.yml`, In production, we strongly recommend you set the `data` `logs` `dump` and `temp` directory

```
es_home_dir: "/opt/elasticsearch"
es_log_dir: "/data01/elasticsearch/logs"
es_data_dir: "/data01/elasticsearch/data"
es_heap_dump_path: "/data01/elasticsearch/dump"
es_temp_dir: "/data01/elasticsearch/temp"
```

## User & Group

```
es_user: "elasticsearch"
es_group: "elasticsearch"
```

## Clustr name

```
es_cluster_name: "nc-elasticsearch"
```

## VM options

```
es_heap_size: 8g
```

## Networks

```
es_network_host: "0.0.0.0"
es_http_port: 9200
es_transport_port: 9010
```

## Others

```
es_action_auto_create: "true"
es_bootstrap_memory_lock: false
```

## Monitoring and Management Elasticsearch

```yml
version: '3.2'
services:
  elasticsearch-hq:
    image: elastichq/elasticsearch-hq
    hostname: elasticsearch-hq
    container_name: elasticsearch-hq
    restart: always
    cap_add:
      - SYS_PTRACE
    security_opt:
      - apparmor:unconfined
    logging:
      driver: "json-file"
      options:
        max-size: "100m"
    environment:
      HQ_DEFAULT_URL: "http://10.1.207.180:9200"
    ports:
      - 5000:5000
```

## Benchmark

[elastic-benchmark](https://github.com/coolbeevip/elastic-benchmark) high write throughpu benchmark

```shell
==========================================
ElasticSearch Import CSV
----------------- params ------------------
index: myindex
id: 0
csv count: 9848742
csv size: 7.93 GB
batch: 10000
concurrency: 16
----------------- output ------------------
succeed : 9848742
fails : 0
total time: 04 min, 25 sec
average size per: 0.84 KB
throughput records: 37165 ops/s
throughput size: 30.65 MB ops/s
==========================================
refresh [myindex] ... status OK time 31897ms
```
