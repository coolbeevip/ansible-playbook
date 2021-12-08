# Ansible Playbook 自动化安装 Elasticsearch 集群 | [English](README.md)

请先在目标服务器创建 `elasticsearch` 用户，密码 `123456`，脚本会在此用户下安装三节点集群，因为 7.X 版本后会自带 JDK，所以我们不需要提前安装 Java 环境。

## 安装计划

服务器规划

| IP地址 | SSH 端口 | SSH 用户名 | SSH 密码 | ROOT 密码 | OS |
| ---- | ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | 22022 | elasticsearch | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.181 | 22022 | elasticsearch | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.182 | 22022 | elasticsearch | 123456 | root123 | CentOS Linux release 7.9.2009 |

**提示：** 可以参考[批量自动化创建用户](https://github.com/coolbeevip/ansible-playbook/blob/main/README_ZH.md#%E5%88%9B%E5%BB%BA%E7%94%A8%E6%88%B7%E5%92%8C%E7%BB%84)

集群节点规划

| IP地址 | Elasticsearch |
| ---- | ---- |
| 10.1.207.180 | ✓ |
| 10.1.207.181 | ✓ |
| 10.1.207.182 | ✓ |

节点安装路径

| 路径 | 描述 |
| ---- | ---- |
| /opt/elasticsearch | 程序安装路径 |
| ~/elasticsearch_uninstall.sh | 集群卸载脚本 |
| ~/elasticsearch.sh | 集群启停脚本 |
| /data01/elasticsearch/logs | 日志文件 |
| /data01/elasticsearch/data | 数据文件 |
| /data01/elasticsearch/dump | DUMP 文件 |
| /data01/elasticsearch/temp | 临时文件 |

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

下载 elasticsearch 安装包到 `~/my-docker-volume/ansible-playbook/packages` 目录

```shell
wget -P ~/my-docker-volume/ansible-playbook/packages https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.13.3-linux-x86_64.tar.gz --no-check-certificate
```

## 配置安装脚本

您需要根据实际情况修改如下配置文件，例如 IP 地址，端口号等

#### main.yml

这个文件中主要定义了目标服务器的地址，登录用户名以及每个 elasticsearch 节点的名称

```yaml
- hosts: 10.1.207.180
  user: elasticsearch
  ...
- hosts: 10.1.207.181
  user: elasticsearch
  ...
- hosts: 10.1.207.182
  user: elasticsearch
  ...
```

#### vars_elasticsearch.yml

这个文件中定义了安装路径、数据路径、安装包文件名、端口、内存等变量信息，你可以根据实际情况修改这些变量，这些变量在并部署时会替换到以下文件中。

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
es_user: "elasticsearch"
es_group: "elasticsearch"
```

安装介质名称以及解压后的目录名

```shell
es_tar: "elasticsearch-7.13.3-linux-x86_64.tar.gz"
es_tar_unzip_dir: "elasticsearch-7.13.3"
```

安装路径

```shell
es_home_dir: "/opt/elasticsearch"
es_log_dir: "/data01/elasticsearch/logs"
es_data_dir: "/data01/elasticsearch/data"
es_heap_dump_path: "/data01/elasticsearch/dump"
es_temp_dir: "/data01/elasticsearch/temp"
```

服务配置

```shell
# 网路信息
es_network_host: "0.0.0.0"
es_http_port: 39200
es_transport_port: 39010
# 集群名称
es_cluster_name: "my-elasticsearch"
# 自动创建索引
es_action_auto_create_index: "true"
# 内存配置
es_heap_size: 8g
es_bootstrap_memory_lock: false
```

集群名称配置

```shell
# 集群节点名称（注意这不是主机名)
node_names:
  10.1.207.180:
    es_node_name: node-180
  10.1.207.181:
    es_node_name: node-181
  10.1.207.182:
    es_node_name: node-182
```

**提示：** 其他更多默认配置，请参考如下文件

* ansible-playbook/elasticsearch/config/elasticsearch.yml.j2
* ansible-playbook/elasticsearch/config/jvm.options.j2
* ansible-playbook/elasticsearch/config/jvm.options.d/gc.options.j2

## 开始安装

启动 Ansible 工具连接到目标服务器，并将 `~/my-docker-volume/ansible-playbook` 目录挂在到容器中。

**提示：** ANSIBLE_SSH_USERS，ANSIBLE_SSH_PASSS 配置成您之前在目标服务器上创建的用户名 `elasticsearch` 和密码 `123456`

**提示：** ANSIBLE_SU_PASSS 为 root 用户的密码

```shell
docker run --name ansible --rm -it \
  -e ANSIBLE_SSH_HOSTS=10.1.207.180,10.1.207.181,10.1.207.182 \
  -e ANSIBLE_SSH_PORTS=22022,22022,22022 \
  -e ANSIBLE_SSH_USERS=elasticsearch,elasticsearch,elasticsearch \
  -e ANSIBLE_SSH_PASSS=123456,123456,123456 \
  -e ANSIBLE_SU_PASSS=root123,root123,root123 \
  -v ~/my-docker-volume/ansible-playbook:/ansible-playbook \
  coolbeevip/ansible:2.8.11-alpine \
  /bin/bash
```

#### 安装 Elasticsearch 集群

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/elasticsearch/main.yml
```

如果你看到如下信息，说明安装完成

```shell
TASK [Install Succeed] ********************************************************************************************************************************************************************************************************
ok: [10.1.207.180] => {
    "msg": "Install Succeed!"
}
```

**提示:** 因为第一次执行脚本时，会上传 Elasticsearch 安装包到所有服务器（约 327MB），所以执行时间较长（取决于你的客户端和服务器之间的网络速度）。 你也可以在执行以上脚本前手动将安装包上传到服务器的安装路径 `/opt/elasticsearch` 下。在我本地环境首次安装大概耗时 5 分钟（上传安装包大概 2 分钟，安装集群大概 3 分钟）

**提示:** 此脚本只适合初始化安装，重复执行此命令可能会收到 `Elasticsearch has been installed, please uninstall and then reinstall` 提示，此时需要先要使用 `ansible all -m shell -a '~/elasticsearch_uninstall.sh'` 命令卸载之前的安装。

**提示:** 您可以使用以下脚本删除不在使用的安装包

```shell
bash-5.0# ansible all -m shell -a "rm -rf /opt/elasticsearch/elasticsearch-7.13.3-linux-x86_64.tar.gz"
```

#### 验证 Elasticsearch 集群

查看目标服务器上进程, 你可以看到每个服务上有两个进程

```shell
bash-5.0# ansible all -m shell -a 'ps aux | grep [/]opt/elasticsearch | wc -l'
10.1.207.182 | CHANGED | rc=0 >>
2

10.1.207.180 | CHANGED | rc=0 >>
2

10.1.207.181 | CHANGED | rc=0 >>
2
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
    "number" : "7.13.3",
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
    "number" : "7.13.3",
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
    "number" : "7.13.3",
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

启动集群

```shell
bash-5.0# ansible all -m shell -a '~/elasticsearch.sh start'
```

停止集群

```shell
bash-5.0# ansible all -m shell -a '~/elasticsearch.sh stop'
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
