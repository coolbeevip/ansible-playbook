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

## 基准测试

```
|                                                         Metric |                           Task |       Value |    Unit |
|---------------------------------------------------------------:|-------------------------------:|------------:|--------:|
|                     Cumulative indexing time of primary shards |                                |     19.4123 |     min |
|             Min cumulative indexing time across primary shards |                                | 0.000633333 |     min |
|          Median cumulative indexing time across primary shards |                                |     3.88232 |     min |
|             Max cumulative indexing time across primary shards |                                |      3.9451 |     min |
|            Cumulative indexing throttle time of primary shards |                                |           0 |     min |
|    Min cumulative indexing throttle time across primary shards |                                |           0 |     min |
| Median cumulative indexing throttle time across primary shards |                                |           0 |     min |
|    Max cumulative indexing throttle time across primary shards |                                |           0 |     min |
|                        Cumulative merge time of primary shards |                                |           0 |     min |
|                       Cumulative merge count of primary shards |                                |           0 |         |
|                Min cumulative merge time across primary shards |                                |           0 |     min |
|             Median cumulative merge time across primary shards |                                |           0 |     min |
|                Max cumulative merge time across primary shards |                                |           0 |     min |
|               Cumulative merge throttle time of primary shards |                                |           0 |     min |
|       Min cumulative merge throttle time across primary shards |                                |           0 |     min |
|    Median cumulative merge throttle time across primary shards |                                |           0 |     min |
|       Max cumulative merge throttle time across primary shards |                                |           0 |     min |
|                      Cumulative refresh time of primary shards |                                |      1.8997 |     min |
|                     Cumulative refresh count of primary shards |                                |          66 |         |
|              Min cumulative refresh time across primary shards |                                |     0.00145 |     min |
|           Median cumulative refresh time across primary shards |                                |    0.424667 |     min |
|              Max cumulative refresh time across primary shards |                                |     0.45795 |     min |
|                        Cumulative flush time of primary shards |                                |      0.2294 |     min |
|                       Cumulative flush count of primary shards |                                |          12 |         |
|                Min cumulative flush time across primary shards |                                |           0 |     min |
|             Median cumulative flush time across primary shards |                                |   0.0162333 |     min |
|                Max cumulative flush time across primary shards |                                |   0.0695667 |     min |
|                                        Total Young Gen GC time |                                |       83.36 |       s |
|                                       Total Young Gen GC count |                                |        4630 |         |
|                                          Total Old Gen GC time |                                |       3.065 |       s |
|                                         Total Old Gen GC count |                                |          38 |         |
|                                                     Store size |                                |      2.8123 |      GB |
|                                                  Translog size |                                | 4.09782e-07 |      GB |
|                                         Heap used for segments |                                |    0.481937 |      MB |
|                                       Heap used for doc values |                                |   0.0282707 |      MB |
|                                            Heap used for terms |                                |    0.362503 |      MB |
|                                            Heap used for norms |                                |   0.0452271 |      MB |
|                                           Heap used for points |                                |           0 |      MB |
|                                    Heap used for stored fields |                                |   0.0459366 |      MB |
|                                                  Segment count |                                |          63 |         |
|                                                 Min Throughput |                   index-append |     1798.52 |  docs/s |
|                                                Mean Throughput |                   index-append |     2085.82 |  docs/s |
|                                              Median Throughput |                   index-append |     2107.73 |  docs/s |
|                                                 Max Throughput |                   index-append |     2143.55 |  docs/s |
|                                        50th percentile latency |                   index-append |       18779 |      ms |
|                                        90th percentile latency |                   index-append |     21099.8 |      ms |
|                                        99th percentile latency |                   index-append |     24426.1 |      ms |
|                                      99.9th percentile latency |                   index-append |     29546.7 |      ms |
|                                       100th percentile latency |                   index-append |     34127.2 |      ms |
|                                   50th percentile service time |                   index-append |       18779 |      ms |
|                                   90th percentile service time |                   index-append |     21099.8 |      ms |
|                                   99th percentile service time |                   index-append |     24426.1 |      ms |
|                                 99.9th percentile service time |                   index-append |     29546.7 |      ms |
|                                  100th percentile service time |                   index-append |     34127.2 |      ms |
|                                                     error rate |                   index-append |           0 |       % |
|                                                 Min Throughput |                    index-stats |       33.55 |   ops/s |
|                                                Mean Throughput |                    index-stats |        33.7 |   ops/s |
|                                              Median Throughput |                    index-stats |       33.71 |   ops/s |
|                                                 Max Throughput |                    index-stats |       33.83 |   ops/s |
|                                        50th percentile latency |                    index-stats |     18716.2 |      ms |
|                                        90th percentile latency |                    index-stats |     25988.5 |      ms |
|                                        99th percentile latency |                    index-stats |     27658.1 |      ms |
|                                      99.9th percentile latency |                    index-stats |     27824.7 |      ms |
|                                       100th percentile latency |                    index-stats |     27844.7 |      ms |
|                                   50th percentile service time |                    index-stats |      27.657 |      ms |
|                                   90th percentile service time |                    index-stats |     28.7906 |      ms |
|                                   99th percentile service time |                    index-stats |     37.8251 |      ms |
|                                 99.9th percentile service time |                    index-stats |     48.9266 |      ms |
|                                  100th percentile service time |                    index-stats |     51.1575 |      ms |
|                                                     error rate |                    index-stats |           0 |       % |
|                                                 Min Throughput |                     node-stats |       30.27 |   ops/s |
|                                                Mean Throughput |                     node-stats |       30.89 |   ops/s |
|                                              Median Throughput |                     node-stats |       30.87 |   ops/s |
|                                                 Max Throughput |                     node-stats |       31.44 |   ops/s |
|                                        50th percentile latency |                     node-stats |       12437 |      ms |
|                                        90th percentile latency |                     node-stats |     21604.1 |      ms |
|                                        99th percentile latency |                     node-stats |     23379.5 |      ms |
|                                      99.9th percentile latency |                     node-stats |     23553.1 |      ms |
|                                       100th percentile latency |                     node-stats |     23573.4 |      ms |
|                                   50th percentile service time |                     node-stats |      29.344 |      ms |
|                                   90th percentile service time |                     node-stats |     33.1467 |      ms |
|                                   99th percentile service time |                     node-stats |      40.011 |      ms |
|                                 99.9th percentile service time |                     node-stats |     64.5643 |      ms |
|                                  100th percentile service time |                     node-stats |     347.235 |      ms |
|                                                     error rate |                     node-stats |           0 |       % |
|                                                 Min Throughput |                        default |       30.95 |   ops/s |
|                                                Mean Throughput |                        default |       31.71 |   ops/s |
|                                              Median Throughput |                        default |        31.4 |   ops/s |
|                                                 Max Throughput |                        default |       33.11 |   ops/s |
|                                        50th percentile latency |                        default |       11554 |      ms |
|                                        90th percentile latency |                        default |     16849.9 |      ms |
|                                        99th percentile latency |                        default |     18268.2 |      ms |
|                                      99.9th percentile latency |                        default |     18392.1 |      ms |
|                                       100th percentile latency |                        default |     18404.1 |      ms |
|                                   50th percentile service time |                        default |     31.5603 |      ms |
|                                   90th percentile service time |                        default |      34.001 |      ms |
|                                   99th percentile service time |                        default |     39.7544 |      ms |
|                                 99.9th percentile service time |                        default |     284.394 |      ms |
|                                  100th percentile service time |                        default |     867.281 |      ms |
|                                                     error rate |                        default |           0 |       % |
|                                                 Min Throughput |                           term |       30.42 |   ops/s |
|                                                Mean Throughput |                           term |       31.49 |   ops/s |
|                                              Median Throughput |                           term |       31.57 |   ops/s |
|                                                 Max Throughput |                           term |       32.11 |   ops/s |
|                                        50th percentile latency |                           term |     21646.8 |      ms |
|                                        90th percentile latency |                           term |     29660.9 |      ms |
|                                        99th percentile latency |                           term |     31447.3 |      ms |
|                                      99.9th percentile latency |                           term |     31644.3 |      ms |
|                                       100th percentile latency |                           term |     31663.8 |      ms |
|                                   50th percentile service time |                           term |     28.8622 |      ms |
|                                   90th percentile service time |                           term |      30.424 |      ms |
|                                   99th percentile service time |                           term |     39.5943 |      ms |
|                                 99.9th percentile service time |                           term |     55.1886 |      ms |
|                                  100th percentile service time |                           term |     87.9936 |      ms |
|                                                     error rate |                           term |           0 |       % |
|                                                 Min Throughput |                         phrase |       31.86 |   ops/s |
|                                                Mean Throughput |                         phrase |       32.29 |   ops/s |
|                                              Median Throughput |                         phrase |       32.31 |   ops/s |
|                                                 Max Throughput |                         phrase |       32.62 |   ops/s |
|                                        50th percentile latency |                         phrase |     21800.8 |      ms |
|                                        90th percentile latency |                         phrase |     30262.7 |      ms |
|                                        99th percentile latency |                         phrase |     32123.6 |      ms |
|                                      99.9th percentile latency |                         phrase |     32319.9 |      ms |
|                                       100th percentile latency |                         phrase |     32342.3 |      ms |
|                                   50th percentile service time |                         phrase |     28.8735 |      ms |
|                                   90th percentile service time |                         phrase |      30.205 |      ms |
|                                   99th percentile service time |                         phrase |      41.354 |      ms |
|                                 99.9th percentile service time |                         phrase |     55.5922 |      ms |
|                                  100th percentile service time |                         phrase |     62.7629 |      ms |
|                                                     error rate |                         phrase |           0 |       % |
|                                                 Min Throughput |           country_agg_uncached |        2.98 |   ops/s |
|                                                Mean Throughput |           country_agg_uncached |        2.98 |   ops/s |
|                                              Median Throughput |           country_agg_uncached |        2.98 |   ops/s |
|                                                 Max Throughput |           country_agg_uncached |        2.98 |   ops/s |
|                                        50th percentile latency |           country_agg_uncached |     251.689 |      ms |
|                                        90th percentile latency |           country_agg_uncached |     284.403 |      ms |
|                                        99th percentile latency |           country_agg_uncached |     315.142 |      ms |
|                                       100th percentile latency |           country_agg_uncached |     328.091 |      ms |
|                                   50th percentile service time |           country_agg_uncached |     248.278 |      ms |
|                                   90th percentile service time |           country_agg_uncached |      280.26 |      ms |
|                                   99th percentile service time |           country_agg_uncached |     311.528 |      ms |
|                                  100th percentile service time |           country_agg_uncached |     324.968 |      ms |
|                                                     error rate |           country_agg_uncached |           0 |       % |
|                                                 Min Throughput |             country_agg_cached |       30.04 |   ops/s |
|                                                Mean Throughput |             country_agg_cached |       30.51 |   ops/s |
|                                              Median Throughput |             country_agg_cached |       30.56 |   ops/s |
|                                                 Max Throughput |             country_agg_cached |          31 |   ops/s |
|                                        50th percentile latency |             country_agg_cached |     34657.5 |      ms |
|                                        90th percentile latency |             country_agg_cached |     42362.4 |      ms |
|                                        99th percentile latency |             country_agg_cached |     44130.1 |      ms |
|                                      99.9th percentile latency |             country_agg_cached |       44302 |      ms |
|                                       100th percentile latency |             country_agg_cached |     44320.4 |      ms |
|                                   50th percentile service time |             country_agg_cached |     29.7696 |      ms |
|                                   90th percentile service time |             country_agg_cached |     33.6731 |      ms |
|                                   99th percentile service time |             country_agg_cached |     45.8928 |      ms |
|                                 99.9th percentile service time |             country_agg_cached |     71.1675 |      ms |
|                                  100th percentile service time |             country_agg_cached |     733.809 |      ms |
|                                                     error rate |             country_agg_cached |           0 |       % |
|                                                 Min Throughput |                         scroll |       12.44 | pages/s |
|                                                Mean Throughput |                         scroll |       12.63 | pages/s |
|                                              Median Throughput |                         scroll |       12.64 | pages/s |
|                                                 Max Throughput |                         scroll |        12.8 | pages/s |
|                                        50th percentile latency |                         scroll |      183223 |      ms |
|                                        90th percentile latency |                         scroll |      206279 |      ms |
|                                        99th percentile latency |                         scroll |      211496 |      ms |
|                                       100th percentile latency |                         scroll |      212076 |      ms |
|                                   50th percentile service time |                         scroll |     1832.23 |      ms |
|                                   90th percentile service time |                         scroll |     1862.78 |      ms |
|                                   99th percentile service time |                         scroll |     1967.52 |      ms |
|                                  100th percentile service time |                         scroll |     2140.04 |      ms |
|                                                     error rate |                         scroll |           0 |       % |
|                                                 Min Throughput |                     expression |        0.56 |   ops/s |
|                                                Mean Throughput |                     expression |        0.63 |   ops/s |
|                                              Median Throughput |                     expression |        0.63 |   ops/s |
|                                                 Max Throughput |                     expression |        0.69 |   ops/s |
|                                        50th percentile latency |                     expression |       69918 |      ms |
|                                        90th percentile latency |                     expression |     72267.2 |      ms |
|                                        99th percentile latency |                     expression |     72949.4 |      ms |
|                                       100th percentile latency |                     expression |     73016.1 |      ms |
|                                   50th percentile service time |                     expression |      748.47 |      ms |
|                                   90th percentile service time |                     expression |     798.708 |      ms |
|                                   99th percentile service time |                     expression |     1004.06 |      ms |
|                                  100th percentile service time |                     expression |     1005.58 |      ms |
|                                                     error rate |                     expression |           0 |       % |
|                                                 Min Throughput |                painless_static |        1.31 |   ops/s |
|                                                Mean Throughput |                painless_static |        1.32 |   ops/s |
|                                              Median Throughput |                painless_static |        1.32 |   ops/s |
|                                                 Max Throughput |                painless_static |        1.33 |   ops/s |
|                                        50th percentile latency |                painless_static |     10784.5 |      ms |
|                                        90th percentile latency |                painless_static |     11284.1 |      ms |
|                                        99th percentile latency |                painless_static |     11872.2 |      ms |
|                                       100th percentile latency |                painless_static |     11920.6 |      ms |
|                                   50th percentile service time |                painless_static |     765.696 |      ms |
|                                   90th percentile service time |                painless_static |     838.967 |      ms |
|                                   99th percentile service time |                painless_static |      930.78 |      ms |
|                                  100th percentile service time |                painless_static |     941.878 |      ms |
|                                                     error rate |                painless_static |           0 |       % |
|                                                 Min Throughput |               painless_dynamic |        1.27 |   ops/s |
|                                                Mean Throughput |               painless_dynamic |        1.27 |   ops/s |
|                                              Median Throughput |               painless_dynamic |        1.27 |   ops/s |
|                                                 Max Throughput |               painless_dynamic |        1.28 |   ops/s |
|                                        50th percentile latency |               painless_dynamic |       18127 |      ms |
|                                        90th percentile latency |               painless_dynamic |     21051.4 |      ms |
|                                        99th percentile latency |               painless_dynamic |     21597.6 |      ms |
|                                       100th percentile latency |               painless_dynamic |     21615.1 |      ms |
|                                   50th percentile service time |               painless_dynamic |      777.39 |      ms |
|                                   90th percentile service time |               painless_dynamic |     869.521 |      ms |
|                                   99th percentile service time |               painless_dynamic |      972.99 |      ms |
|                                  100th percentile service time |               painless_dynamic |     993.168 |      ms |
|                                                     error rate |               painless_dynamic |           0 |       % |
|                                                 Min Throughput | decay_geo_gauss_function_score |           1 |   ops/s |
|                                                Mean Throughput | decay_geo_gauss_function_score |           1 |   ops/s |
|                                              Median Throughput | decay_geo_gauss_function_score |           1 |   ops/s |
|                                                 Max Throughput | decay_geo_gauss_function_score |           1 |   ops/s |
|                                        50th percentile latency | decay_geo_gauss_function_score |     920.782 |      ms |
|                                        90th percentile latency | decay_geo_gauss_function_score |     998.869 |      ms |
|                                        99th percentile latency | decay_geo_gauss_function_score |     1033.89 |      ms |
|                                       100th percentile latency | decay_geo_gauss_function_score |     1045.86 |      ms |
|                                   50th percentile service time | decay_geo_gauss_function_score |     915.596 |      ms |
|                                   90th percentile service time | decay_geo_gauss_function_score |         991 |      ms |
|                                   99th percentile service time | decay_geo_gauss_function_score |     1032.86 |      ms |
|                                  100th percentile service time | decay_geo_gauss_function_score |        1044 |      ms |
|                                                     error rate | decay_geo_gauss_function_score |           0 |       % |
|                                                 Min Throughput |   decay_geo_gauss_script_score |           1 |   ops/s |
|                                                Mean Throughput |   decay_geo_gauss_script_score |           1 |   ops/s |
|                                              Median Throughput |   decay_geo_gauss_script_score |           1 |   ops/s |
|                                                 Max Throughput |   decay_geo_gauss_script_score |           1 |   ops/s |
|                                        50th percentile latency |   decay_geo_gauss_script_score |      936.05 |      ms |
|                                        90th percentile latency |   decay_geo_gauss_script_score |     973.677 |      ms |
|                                        99th percentile latency |   decay_geo_gauss_script_score |     1019.91 |      ms |
|                                       100th percentile latency |   decay_geo_gauss_script_score |     1024.38 |      ms |
|                                   50th percentile service time |   decay_geo_gauss_script_score |     931.501 |      ms |
|                                   90th percentile service time |   decay_geo_gauss_script_score |     971.601 |      ms |
|                                   99th percentile service time |   decay_geo_gauss_script_score |     1017.19 |      ms |
|                                  100th percentile service time |   decay_geo_gauss_script_score |     1020.53 |      ms |
|                                                     error rate |   decay_geo_gauss_script_score |           0 |       % |
|                                                 Min Throughput |     field_value_function_score |         1.5 |   ops/s |
|                                                Mean Throughput |     field_value_function_score |         1.5 |   ops/s |
|                                              Median Throughput |     field_value_function_score |         1.5 |   ops/s |
|                                                 Max Throughput |     field_value_function_score |         1.5 |   ops/s |
|                                        50th percentile latency |     field_value_function_score |     330.127 |      ms |
|                                        90th percentile latency |     field_value_function_score |     347.117 |      ms |
|                                        99th percentile latency |     field_value_function_score |      355.27 |      ms |
|                                       100th percentile latency |     field_value_function_score |     364.989 |      ms |
|                                   50th percentile service time |     field_value_function_score |     327.673 |      ms |
|                                   90th percentile service time |     field_value_function_score |     343.949 |      ms |
|                                   99th percentile service time |     field_value_function_score |     352.053 |      ms |
|                                  100th percentile service time |     field_value_function_score |     361.956 |      ms |
|                                                     error rate |     field_value_function_score |           0 |       % |
|                                                 Min Throughput |       field_value_script_score |         1.5 |   ops/s |
|                                                Mean Throughput |       field_value_script_score |         1.5 |   ops/s |
|                                              Median Throughput |       field_value_script_score |         1.5 |   ops/s |
|                                                 Max Throughput |       field_value_script_score |         1.5 |   ops/s |
|                                        50th percentile latency |       field_value_script_score |     376.666 |      ms |
|                                        90th percentile latency |       field_value_script_score |     398.657 |      ms |
|                                        99th percentile latency |       field_value_script_score |     454.444 |      ms |
|                                       100th percentile latency |       field_value_script_score |     458.427 |      ms |
|                                   50th percentile service time |       field_value_script_score |      374.48 |      ms |
|                                   90th percentile service time |       field_value_script_score |     396.584 |      ms |
|                                   99th percentile service time |       field_value_script_score |     452.781 |      ms |
|                                  100th percentile service time |       field_value_script_score |     457.078 |      ms |
|                                                     error rate |       field_value_script_score |           0 |       % |
|                                                 Min Throughput |                    large_terms |        0.56 |   ops/s |
|                                                Mean Throughput |                    large_terms |        0.56 |   ops/s |
|                                              Median Throughput |                    large_terms |        0.56 |   ops/s |
|                                                 Max Throughput |                    large_terms |        0.56 |   ops/s |
|                                        50th percentile latency |                    large_terms |      218790 |      ms |
|                                        90th percentile latency |                    large_terms |      253257 |      ms |
|                                        99th percentile latency |                    large_terms |      261114 |      ms |
|                                       100th percentile latency |                    large_terms |      261914 |      ms |
|                                   50th percentile service time |                    large_terms |     1747.94 |      ms |
|                                   90th percentile service time |                    large_terms |      1815.5 |      ms |
|                                   99th percentile service time |                    large_terms |     1881.91 |      ms |
|                                  100th percentile service time |                    large_terms |      2162.5 |      ms |
|                                                     error rate |                    large_terms |           0 |       % |
|                                                 Min Throughput |           large_filtered_terms |        0.52 |   ops/s |
|                                                Mean Throughput |           large_filtered_terms |        0.53 |   ops/s |
|                                              Median Throughput |           large_filtered_terms |        0.53 |   ops/s |
|                                                 Max Throughput |           large_filtered_terms |        0.53 |   ops/s |
|                                        50th percentile latency |           large_filtered_terms |      245779 |      ms |
|                                        90th percentile latency |           large_filtered_terms |      282469 |      ms |
|                                        99th percentile latency |           large_filtered_terms |      290437 |      ms |
|                                       100th percentile latency |           large_filtered_terms |      291324 |      ms |
|                                   50th percentile service time |           large_filtered_terms |     1756.56 |      ms |
|                                   90th percentile service time |           large_filtered_terms |     1824.54 |      ms |
|                                   99th percentile service time |           large_filtered_terms |     2229.18 |      ms |
|                                  100th percentile service time |           large_filtered_terms |     2809.47 |      ms |
|                                                     error rate |           large_filtered_terms |           0 |       % |
|                                                 Min Throughput |         large_prohibited_terms |        0.56 |   ops/s |
|                                                Mean Throughput |         large_prohibited_terms |        0.56 |   ops/s |
|                                              Median Throughput |         large_prohibited_terms |        0.56 |   ops/s |
|                                                 Max Throughput |         large_prohibited_terms |        0.56 |   ops/s |
|                                        50th percentile latency |         large_prohibited_terms |      223830 |      ms |
|                                        90th percentile latency |         large_prohibited_terms |      257684 |      ms |
|                                        99th percentile latency |         large_prohibited_terms |      265355 |      ms |
|                                       100th percentile latency |         large_prohibited_terms |      266173 |      ms |
|                                   50th percentile service time |         large_prohibited_terms |     1747.03 |      ms |
|                                   90th percentile service time |         large_prohibited_terms |     1817.48 |      ms |
|                                   99th percentile service time |         large_prohibited_terms |     2197.49 |      ms |
|                                  100th percentile service time |         large_prohibited_terms |     2283.87 |      ms |
|                                                     error rate |         large_prohibited_terms |           0 |       % |
|                                                 Min Throughput |           desc_sort_population |         1.5 |   ops/s |
|                                                Mean Throughput |           desc_sort_population |         1.5 |   ops/s |
|                                              Median Throughput |           desc_sort_population |         1.5 |   ops/s |
|                                                 Max Throughput |           desc_sort_population |        1.51 |   ops/s |
|                                        50th percentile latency |           desc_sort_population |     158.239 |      ms |
|                                        90th percentile latency |           desc_sort_population |     163.699 |      ms |
|                                        99th percentile latency |           desc_sort_population |     172.395 |      ms |
|                                       100th percentile latency |           desc_sort_population |     176.986 |      ms |
|                                   50th percentile service time |           desc_sort_population |     155.332 |      ms |
|                                   90th percentile service time |           desc_sort_population |     160.523 |      ms |
|                                   99th percentile service time |           desc_sort_population |      169.21 |      ms |
|                                  100th percentile service time |           desc_sort_population |     171.228 |      ms |
|                                                     error rate |           desc_sort_population |           0 |       % |
|                                                 Min Throughput |            asc_sort_population |        1.51 |   ops/s |
|                                                Mean Throughput |            asc_sort_population |        1.51 |   ops/s |
|                                              Median Throughput |            asc_sort_population |        1.51 |   ops/s |
|                                                 Max Throughput |            asc_sort_population |        1.51 |   ops/s |
|                                        50th percentile latency |            asc_sort_population |     155.815 |      ms |
|                                        90th percentile latency |            asc_sort_population |     165.969 |      ms |
|                                        99th percentile latency |            asc_sort_population |     192.864 |      ms |
|                                       100th percentile latency |            asc_sort_population |     256.663 |      ms |
|                                   50th percentile service time |            asc_sort_population |     153.301 |      ms |
|                                   90th percentile service time |            asc_sort_population |     162.615 |      ms |
|                                   99th percentile service time |            asc_sort_population |     190.702 |      ms |
|                                  100th percentile service time |            asc_sort_population |     254.019 |      ms |
|                                                     error rate |            asc_sort_population |           0 |       % |
|                                                 Min Throughput | asc_sort_with_after_population |         1.5 |   ops/s |
|                                                Mean Throughput | asc_sort_with_after_population |        1.51 |   ops/s |
|                                              Median Throughput | asc_sort_with_after_population |         1.5 |   ops/s |
|                                                 Max Throughput | asc_sort_with_after_population |        1.51 |   ops/s |
|                                        50th percentile latency | asc_sort_with_after_population |     197.385 |      ms |
|                                        90th percentile latency | asc_sort_with_after_population |     206.593 |      ms |
|                                        99th percentile latency | asc_sort_with_after_population |     217.753 |      ms |
|                                       100th percentile latency | asc_sort_with_after_population |     221.226 |      ms |
|                                   50th percentile service time | asc_sort_with_after_population |      194.56 |      ms |
|                                   90th percentile service time | asc_sort_with_after_population |     203.304 |      ms |
|                                   99th percentile service time | asc_sort_with_after_population |     210.905 |      ms |
|                                  100th percentile service time | asc_sort_with_after_population |     219.705 |      ms |
|                                                     error rate | asc_sort_with_after_population |           0 |       % |
|                                                 Min Throughput |            desc_sort_geonameid |           6 |   ops/s |
|                                                Mean Throughput |            desc_sort_geonameid |           6 |   ops/s |
|                                              Median Throughput |            desc_sort_geonameid |           6 |   ops/s |
|                                                 Max Throughput |            desc_sort_geonameid |           6 |   ops/s |
|                                        50th percentile latency |            desc_sort_geonameid |     44.2396 |      ms |
|                                        90th percentile latency |            desc_sort_geonameid |     51.0027 |      ms |
|                                        99th percentile latency |            desc_sort_geonameid |     63.9771 |      ms |
|                                       100th percentile latency |            desc_sort_geonameid |     68.5273 |      ms |
|                                   50th percentile service time |            desc_sort_geonameid |      41.552 |      ms |
|                                   90th percentile service time |            desc_sort_geonameid |     47.4226 |      ms |
|                                   99th percentile service time |            desc_sort_geonameid |     60.8173 |      ms |
|                                  100th percentile service time |            desc_sort_geonameid |     64.2967 |      ms |
|                                                     error rate |            desc_sort_geonameid |           0 |       % |
|                                                 Min Throughput | desc_sort_with_after_geonameid |        5.98 |   ops/s |
|                                                Mean Throughput | desc_sort_with_after_geonameid |        5.99 |   ops/s |
|                                              Median Throughput | desc_sort_with_after_geonameid |        5.99 |   ops/s |
|                                                 Max Throughput | desc_sort_with_after_geonameid |        5.99 |   ops/s |
|                                        50th percentile latency | desc_sort_with_after_geonameid |     159.465 |      ms |
|                                        90th percentile latency | desc_sort_with_after_geonameid |     197.729 |      ms |
|                                        99th percentile latency | desc_sort_with_after_geonameid |     208.561 |      ms |
|                                       100th percentile latency | desc_sort_with_after_geonameid |     222.923 |      ms |
|                                   50th percentile service time | desc_sort_with_after_geonameid |     140.805 |      ms |
|                                   90th percentile service time | desc_sort_with_after_geonameid |     191.285 |      ms |
|                                   99th percentile service time | desc_sort_with_after_geonameid |     201.323 |      ms |
|                                  100th percentile service time | desc_sort_with_after_geonameid |     202.252 |      ms |
|                                                     error rate | desc_sort_with_after_geonameid |           0 |       % |
|                                                 Min Throughput |             asc_sort_geonameid |        6.01 |   ops/s |
|                                                Mean Throughput |             asc_sort_geonameid |        6.02 |   ops/s |
|                                              Median Throughput |             asc_sort_geonameid |        6.02 |   ops/s |
|                                                 Max Throughput |             asc_sort_geonameid |        6.02 |   ops/s |
|                                        50th percentile latency |             asc_sort_geonameid |      45.974 |      ms |
|                                        90th percentile latency |             asc_sort_geonameid |     49.5045 |      ms |
|                                        99th percentile latency |             asc_sort_geonameid |     62.0141 |      ms |
|                                       100th percentile latency |             asc_sort_geonameid |     91.7896 |      ms |
|                                   50th percentile service time |             asc_sort_geonameid |     43.5626 |      ms |
|                                   90th percentile service time |             asc_sort_geonameid |     45.7498 |      ms |
|                                   99th percentile service time |             asc_sort_geonameid |     57.3795 |      ms |
|                                  100th percentile service time |             asc_sort_geonameid |     88.6317 |      ms |
|                                                     error rate |             asc_sort_geonameid |           0 |       % |
|                                                 Min Throughput |  asc_sort_with_after_geonameid |           6 |   ops/s |
|                                                Mean Throughput |  asc_sort_with_after_geonameid |           6 |   ops/s |
|                                              Median Throughput |  asc_sort_with_after_geonameid |           6 |   ops/s |
|                                                 Max Throughput |  asc_sort_with_after_geonameid |           6 |   ops/s |
|                                        50th percentile latency |  asc_sort_with_after_geonameid |     120.704 |      ms |
|                                        90th percentile latency |  asc_sort_with_after_geonameid |     160.245 |      ms |
|                                        99th percentile latency |  asc_sort_with_after_geonameid |     170.389 |      ms |
|                                       100th percentile latency |  asc_sort_with_after_geonameid |         172 |      ms |
|                                   50th percentile service time |  asc_sort_with_after_geonameid |     117.667 |      ms |
|                                   90th percentile service time |  asc_sort_with_after_geonameid |     156.771 |      ms |
|                                   99th percentile service time |  asc_sort_with_after_geonameid |     167.555 |      ms |
|                                  100th percentile service time |  asc_sort_with_after_geonameid |      169.41 |      ms |
|                                                     error rate |  asc_sort_with_after_geonameid |           0 |       % |



-----------------------------------
[INFO] SUCCESS (took 10944 seconds)
-----------------------------------
```
