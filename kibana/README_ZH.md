# Ansible Playbook 自动化安装 Kibana & Metricbeat | [English](README.md)

本脚本将自动在目标服务器安装 Metricbeat 和 Kibana 服务实现对以下组件的可视化监控

* Elasticsearch
* Redis
* MySQL
* Kafka

因为 Metricbeat 采集服务的指标信息要存储在 Elasticsearch 中并通过 Kibana 展示，所以你需要先安装一个 Elasticsearch 集群

## 安装计划

服务器规划

| IP地址 | SSH 端口 | SSH 用户名 | SSH 密码 | ROOT 密码 | OS |
| ---- | ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | 22022 | kibana | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.181 | 22022 | kibana | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.182 | 22022 | kibana | 123456 | root123 | CentOS Linux release 7.9.2009 |

**提示：** 可以参考[批量自动化创建用户](https://github.com/coolbeevip/ansible-playbook/blob/main/README_ZH.md#%E5%88%9B%E5%BB%BA%E7%94%A8%E6%88%B7%E5%92%8C%E7%BB%84)

集群节点规划

| IP | Metricbeat | Kibana |
| ---- | ---- | ---- |
| 10.1.207.180 | ✓ | ✓ |
| 10.1.207.181 | ✓ | |
| 10.1.207.182 | ✓ | |

节点安装路径

| 路径 | 描述 |
| ---- | ---- |
| /opt/kibana | 程序安装路径 |
| ~/kibana_uninstall.sh | 集群卸载脚本 |
| ~/kibana.sh | 集群启停脚本 |
| /data01/kibana/logs | PID 文件以及日志文件 |


## 下载安装包和 Playbook 脚本

在你的笔记本上创建 playbook 脚本存放目录

```shell
mkdir -p ~/my-docker-volume/ansible-playbook
```

下载 playbook 脚本

```shell
cd ~/my-docker-volume/ansible-playbook
git clone https://github.com/coolbeevip/ansible-playbook.git
```

下载 Kibana 和 Metricbeat 安装包到 `~/my-docker-volume/ansible-playbook/packages` 目录

```shell
wget -P ~/my-docker-volume/ansible-playbook/packages https://artifacts.elastic.co/downloads/kibana/kibana-7.16.1-linux-x86_64.tar.gz --no-check-certificate
wget -P ~/my-docker-volume/ansible-playbook/packages https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.16.1-linux-x86_64.tar.gz --no-check-certificate
```

## 配置安装脚本

#### main.yml

这个文件中主要定义了目标服务器的地址，登录用户名以及每个 elasticsearch 节点的名称

```yaml
- hosts: 10.1.207.180
  user: kibana
  ...
- hosts: 10.1.207.181
  user: kibana
  ...
- hosts: 10.1.207.182
  user: kibana
  ...
```

#### vars_kibana.yml



操作系统 Limits

```shell
limits_hard_nproc: '65535'
limits_soft_nproc: '65535'
limits_hard_nofile: '278528'
limits_soft_nofile: '278528'
limits_hard_stack: 'unlimited'
limits_soft_stack: 'unlimited'
```

安装用的用户名、用户组

```shell
kibana_user: "kibana"
kibana_group: "kibana"
```

## 开始安装

启动 Ansible 工具连接到目标服务器，并将 `~/my-docker-volume/ansible-playbook` 目录挂在到容器中。

**提示：** ANSIBLE_SSH_USERS，ANSIBLE_SSH_PASSS 配置成您之前在目标服务器上创建的用户名 `kibana` 和密码 `123456`

**提示：** ANSIBLE_SU_PASSS 为 root 用户的密码

```shell
docker run --name ansible --rm -it \
  -e ANSIBLE_SSH_HOSTS=10.1.207.180,10.1.207.181,10.1.207.182 \
  -e ANSIBLE_SSH_PORTS=22022,22022,22022 \
  -e ANSIBLE_SSH_USERS=kibana,kibana,kibana \
  -e ANSIBLE_SSH_PASSS=kibana123,kibana123,kibana123 \
  -e ANSIBLE_SU_PASSS=root123,root123,root123 \
  -v ~/my-docker-volume/ansible-playbook:/ansible-playbook \
  coolbeevip/ansible:2.8.11-alpine \
  /bin/bash
```

#### 安装 Kibana 和 Metricbeat

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/kibana/main.yml
```

如果你看到如下信息，说明安装完成(必须 failed=0)

```shell
PLAY RECAP *****************************************************************************************************************************************************************************************************************************************************
10.1.207.180               : ok=43   changed=15   unreachable=0    failed=0    skipped=5    rescued=0    ignored=0
10.1.207.181               : ok=37   changed=9    unreachable=0    failed=0    skipped=11   rescued=0    ignored=0
10.1.207.182               : ok=37   changed=9    unreachable=0    failed=0    skipped=11   rescued=0    ignored=0s
```

**提示:** 因为第一次执行脚本时，会上传 Elasticsearch 安装包到所有服务器（约 327MB），所以执行时间较长（取决于你的客户端和服务器之间的网络速度）。 你也可以在执行以上脚本前手动将安装包上传到服务器的安装路径 `/opt/elasticsearch` 下。在我本地环境首次安装大概耗时 5 分钟（上传安装包大概 2 分钟，安装集群大概 3 分钟）

**提示:** 此脚本只适合初始化安装，重复执行此命令可能会收到 `Elasticsearch has been installed, please uninstall and then reinstall` 提示，此时需要先要使用 `ansible all -m shell -a '~/elasticsearch_uninstall.sh'` 命令卸载之前的安装。

#### 验证 Kibana 服务

查看进程ID

```shell
bash-5.0# ansible all -m shell -a '~/elasticsearch.sh status'
10.1.207.181 | CHANGED | rc=0 >>
Elasticsearch is Running as PID: 19219
19281

10.1.207.180 | CHANGED | rc=0 >>
Elasticsearch is Running as PID: 17353
17386

10.1.207.182 | CHANGED | rc=0 >>
Elasticsearch is Running as PID: 30660
```

查看目标服 elasticsearch 服务，可以看到每个节点服务都已经启动

```shell
bash-5.0# ansible all -m shell -a 'curl http://0.0.0.0:39200/?pretty'
10.1.207.182 | CHANGED | rc=0 >>
{
  "name" : "node-182",
  "cluster_name" : "nc-elasticsearch",
  "cluster_uuid" : "_na_",
  "version" : {
    "number" : "7.16.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "5d21bea28db1e89ecc1f66311ebdec9dc3aa7d64",
    "build_date" : "2021-07-02T12:06:10.804015202Z",
    "build_snapshot" : false,
    "lucene_version" : "8.8.2",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   522  100   522    0     0   3279      0 --:--:-- --:--:-- --:--:--  3303

10.1.207.181 | CHANGED | rc=0 >>
{
  "name" : "node-181",
  "cluster_name" : "nc-elasticsearch",
  "cluster_uuid" : "_na_",
  "version" : {
    "number" : "7.16.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "5d21bea28db1e89ecc1f66311ebdec9dc3aa7d64",
    "build_date" : "2021-07-02T12:06:10.804015202Z",
    "build_snapshot" : false,
    "lucene_version" : "8.8.2",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   522  100   522    0     0   3195      0 --:--:-- --:--:-- --:--:--  3222

10.1.207.180 | CHANGED | rc=0 >>
{
  "name" : "node-180",
  "cluster_name" : "nc-elasticsearch",
  "cluster_uuid" : "_na_",
  "version" : {
    "number" : "7.16.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "5d21bea28db1e89ecc1f66311ebdec9dc3aa7d64",
    "build_date" : "2021-07-02T12:06:10.804015202Z",
    "build_snapshot" : false,
    "lucene_version" : "8.8.2",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   522  100   522    0     0   3139      0 --:--:-- --:--:-- --:--:--  3163
```

查看集群状态，可以看到三节点集群已经建立

```shell
bash-5.0# ansible all -m shell -a 'curl http://0.0.0.0:39200/_cat/nodes?v'
10.1.207.182 | CHANGED | rc=0 >>
ip           heap.percent ram.percent cpu load_1m load_5m load_15m node.role   master name
10.1.207.181            5          73  17    0.77    0.47     0.37 cdfhilmrstw -      node-181
10.1.207.182            5          35  15    0.64    0.29     0.25 cdfhilmrstw -      node-182
10.1.207.180            5          99  30    3.47    1.92     1.76 cdfhilmrstw *      node-180  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   376  100   376    0     0   3079      0 --:--:-- --:--:-- --:--:--  3107

10.1.207.181 | CHANGED | rc=0 >>
ip           heap.percent ram.percent cpu load_1m load_5m load_15m node.role   master name
10.1.207.181            5          73  17    0.77    0.47     0.37 cdfhilmrstw -      node-181
10.1.207.180            5          99  30    3.47    1.92     1.76 cdfhilmrstw *      node-180
10.1.207.182            5          35  15    0.64    0.29     0.25 cdfhilmrstw -      node-182  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   376  100   376    0     0   3719      0 --:--:-- --:--:-- --:--:--  3722

10.1.207.180 | CHANGED | rc=0 >>
ip           heap.percent ram.percent cpu load_1m load_5m load_15m node.role   master name
10.1.207.181            5          73  17    0.77    0.47     0.37 cdfhilmrstw -      node-181
10.1.207.180            5          99  30    3.47    1.92     1.76 cdfhilmrstw *      node-180
10.1.207.182            5          35  15    0.64    0.29     0.25 cdfhilmrstw -      node-182  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   376  100   376    0     0   9153      0 --:--:-- --:--:-- --:--:--  9400
```

## 常用运维命令

启动服务

```shell
bash-5.0# ansible all -m shell -a '~/elasticsearch.sh start'
```

停止服务

```shell
bash-5.0# ansible all -m shell -a '~/elasticsearch.sh stop'
```

查看服务进程ID

```shell
bash-5.0# ansible all -m shell -a '~/elasticsearch.sh status'
```

## Q & A

#### 如何彻底删除 Elasticsearch 集群

A: `~/elasticsearch_uninstall.sh` 脚本将 **kill Elasticsearch 进程，删除程序文件和所有数据文件**

```shell
bash-5.0# ansible all -m shell -a '~/elasticsearch_uninstall.sh'
10.1.207.182 | CHANGED | rc=0 >>
Kill Elasticsearch process
Delete Elasticsearch data
Delete Elasticsearch programs
Delete elasticsearch_uninstall.sh

10.1.207.180 | CHANGED | rc=0 >>
Kill Elasticsearch process
Delete Elasticsearch data
Delete Elasticsearch programs
Delete elasticsearch_uninstall.sh

10.1.207.181 | CHANGED | rc=0 >>
Kill Elasticsearch process
Delete Elasticsearch data
Delete Elasticsearch programs
Delete elasticsearch_uninstall.sh
```


./metricbeat modules enable elasticsearch-xpack


/opt/elasticsearch/metricbeat-7.16.1-linux-x86_64/modules.d/elasticsearch-xpack.yml
[elasticsearch@oss-irms-181 modules.d]$ cat elasticsearch-xpack.yml
# Module: elasticsearch
# Docs: https://www.elastic.co/guide/en/beats/metricbeat/7.x/metricbeat-module-elasticsearch.html

- module: elasticsearch
  xpack.enabled: true
  period: 10s
  hosts: ["http://localhost:39200"]
  #username: "user"
  #password: "secret"


/opt/elasticsearch/metricbeat-7.16.1-linux-x86_64/metricbeat.yml  
# ---------------------------- Elasticsearch Output ----------------------------
output.elasticsearch:
  # Array of hosts to connect to.
  hosts: ["localhost:39200"]

  # Protocol - either `http` (default) or `https`.
  #protocol: "https"

  # Authentication credentials - either API key or username/password.
  #api_key: "id:api_key"
  #username: "elastic"
  #password: "changeme"



./metricbeat modules enable kafka
[kibana@oss-irms-180 metricbeat-7.16.1-linux-x86_64]$ cat modules.d/kafka.yml
# Module: kafka
# Docs: https://www.elastic.co/guide/en/beats/metricbeat/7.x/metricbeat-module-kafka.html

# Kafka metrics collected using the Kafka protocol
- module: kafka
  #metricsets:
  #  - partition
  #  - consumergroup
  period: 10s
  hosts: ["localhost:9092"]

  #client_id: metricbeat
  #retries: 3
  #backoff: 250ms

  # List of Topics to query metadata for. If empty, all topics will be queried.
  #topics: []

  # Optional SSL. By default is off.
  # List of root certificates for HTTPS server verifications
  #ssl.certificate_authorities: ["/etc/pki/root/ca.pem"]

  # Certificate for SSL client authentication
  #ssl.certificate: "/etc/pki/client/cert.pem"

  # Client Certificate Key
  #ssl.key: "/etc/pki/client/cert.key"

  # Client Certificate Passphrase (in case your Client Certificate Key is encrypted)
  #ssl.key_passphrase: "yourKeyPassphrase"

  # SASL authentication
  #username: ""
  #password: ""

  # SASL authentication mechanism used. Can be one of PLAIN, SCRAM-SHA-256 or SCRAM-SHA-512.
  # Defaults to PLAIN when `username` and `password` are configured.
  #sasl.mechanism: ''

# Metrics collected from a Kafka broker using Jolokia
#- module: kafka
#  metricsets:
#    - broker
#  period: 10s
#  hosts: ["localhost:8779"]

# Metrics collected from a Java Kafka consumer using Jolokia
#- module: kafka
#  metricsets:
#    - consumer
#  period: 10s
#  hosts: ["localhost:8774"]

# Metrics collected from a Java Kafka producer using Jolokia
#- module: kafka
#  metricsets:
#    - producer
#  period: 10s
#  hosts: ["localhost:8775"]



./metricbeat modules enable linux
[kibana@oss-irms-180 metricbeat-7.16.1-linux-x86_64]$ cat modules.d/linux.yml
# Module: linux
# Docs: https://www.elastic.co/guide/en/beats/metricbeat/7.x/metricbeat-module-linux.html

- module: linux
  period: 10s
  metricsets:
    - "pageinfo"
    - "memory"
    # - ksm
    # - conntrack
    # - iostat
    # - pressure
  enabled: true
  #hostfs: /hostfs
./metricbeat -e  
