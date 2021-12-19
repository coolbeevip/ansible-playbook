# Automate Install Elasticsearch Cluster with Ansible Playbook | [中文](README_ZH.md)

Please create a `elasticsearch` user and defautl password is `123456` on the linux server before starting. This playbook script install three node clusters.
Since the 7.x version will use the bundled JDK, there is no need to install the Java in advance.

## Planning Your Installation

Planning for server

| IP | SSH PORT | SSH USER | SSH PASSWORD | ROOT PASSWORD | OS |
| ---- | ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | 22022 | elasticsearch | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.181 | 22022 | elasticsearch | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.182 | 22022 | elasticsearch | 123456 | root123 | CentOS Linux release 7.9.2009 |

**TIPS：** reference [Create User in Batch Automation](https://github.com/coolbeevip/ansible-playbook#create-user--group)

Planning for nodes

| IP | Elasticsearch |
| ---- | ---- |
| 10.1.207.180 | ✓ |
| 10.1.207.181 | ✓ |
| 10.1.207.182 | ✓ |

Planning for installation directory

| PATH | DESCRIPTION |
| /opt/elasticsearch | programs |
| ~/elasticsearch_uninstall.sh | uninstall script |
| ~/elasticsearch.sh | elasticsearch start & stop shell |
| /data01/elasticsearch/logs | log directory |
| /data01/elasticsearch/data | data directory |
| /data01/elasticsearch/dump | dump directory |
| /data01/elasticsearch/temp | temporary directory |

## Download Elasticsearch Tar & Ansible Playbook scripts

Create a directory for Ansible Playbook scripts

```shell
mkdir -p ~/my-docker-volume/ansible-playbook
```

Download Ansible playbook scripts

```shell
cd ~/my-docker-volume/ansible-playbook
git clone https://github.com/coolbeevip/ansible-playbook.git
```

Download Elasticsearch tar to directory `~/my-docker-volume/ansible-playbook/packages`

```shell
wget -P ~/my-docker-volume/ansible-playbook/packages https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.16.1-linux-x86_64.tar.gz --no-check-certificate
```

## Configuration

> You can edit the following configuration files to modify the default parameters

#### main.yml

This file mainly defines the address of the target server, the login user name

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

This file defines variable information such as installation path, data path, installation package file name, port, memory, etc. You can modify these variables according to the actual environment. These variables will be replaced in the following files when they are deployed.

Linux  Limits

```shell
limits_hard_nproc: '65535'
limits_soft_nproc: '65535'
limits_hard_nofile: '278528'
limits_soft_nofile: '278528'
limits_hard_stack: 'unlimited'
limits_soft_stack: 'unlimited'
```

Linux user & group

```shell
es_user: "elasticsearch"
es_group: "elasticsearch"
```

Tarball and unzipped directory name

```shell
es_tar: "elasticsearch-7.16.1-linux-x86_64.tar.gz"
es_tar_unzip_dir: "elasticsearch-7.16.1"
```

Installation path

```shell
es_home_dir: "/opt/elasticsearch"
es_log_dir: "/data01/elasticsearch/logs"
es_data_dir: "/data01/elasticsearch/data"
es_heap_dump_path: "/data01/elasticsearch/dump"
es_temp_dir: "/data01/elasticsearch/temp"
```

Elasticsearch configuration

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

Elasticsearch node name

```shell
es_hosts:
  10.1.207.180:
    es_node_name: node-180
  10.1.207.181:
    es_node_name: node-181
  10.1.207.182:
    es_node_name: node-182
```

XPack Security

```yaml
es_xpack_security_enabled: false
ex_xpack_security_users:
  - username: myadmin
    password: myadmin123
    roles: [superuser]
  - username: mykibana
    password: mykibana123
    roles: [kibana_admin]
```  

**TIPS：** For more default configuration, please refer to the following file

## Installation

Start the Ansible container tool to connect to the target server, And mount directory `~/my-docker-volume/ansible-playbook` in the container.

**NOTICE:** ANSIBLE_SSH_USERS，ANSIBLE_SSH_PASSS is linux user elasticsearch and password

**NOTICE:** ANSIBLE_SU_PASSS is user root password

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

#### Install Elasticsearch Cluster

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/elasticsearch/main.yml
```

If you see the following message, the installation is completed(must failed=0)

```shell
PLAY RECAP *****************************************************************************************************************************************************************************************************************************************************
10.1.207.180               : ok=43   changed=15   unreachable=0    failed=0    skipped=5    rescued=0    ignored=0
10.1.207.181               : ok=37   changed=9    unreachable=0    failed=0    skipped=11   rescued=0    ignored=0
10.1.207.182               : ok=37   changed=9    unreachable=0    failed=0    skipped=11   rescued=0    ignored=0
```

**TIPS:** Because the Elasticsearch installation package will be uploaded to all servers (about 327MB) when the script is executed for the first time, so take longer to execute. The first installation on my local machine takes < 5 minutes(upload package taske about 2 minutes, others take about 3 minutes.)

**TIPS:** This script is only used for initial installation. Repeated execution of this command may receive a prompt of `Elasticsearch has been installed, please uninstall and then reinstall`. At this time, you need to use `ansible all -m shell -a '~/elasticsearch_uninstall.sh'` uninstalls.

**TIPS:** You can use the following script to delete the temporary files generated during the installation

```shell
bash-5.0# ansible all -m shell -a "rm -rf /opt/elasticsearch/elasticsearch-7.16.1-linux-x86_64.tar.gz"
```

#### Verify Elasticsearch Cluster

Lookup Elasticsearch PID

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

View Elasticsearch service

**TIPS:** Add authentication parameters `--user myadmin:myadmin123` for RESTful API when enabled XPack Security, for example: `ansible all -m shell -a 'curl --user myadmin:myadmin123 http://0.0.0.0:39200/?pretty'`

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

View Elasticsearch cluster status

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

## Common Maintenance Commands

Start Elasticsearch

```shell
bash-5.0# ansible all -m shell -a '~/elasticsearch.sh start'
```

Stop Elasticsearch

```shell
bash-5.0# ansible all -m shell -a '~/elasticsearch.sh stop'
```

Elasticsearch PID

```shell
bash-5.0# ansible all -m shell -a '~/elasticsearch.sh status'
```

## Q & A

#### How to force uninstall

A: The `~/elasticsearch_uninstall.sh` script will **kill -9 Elasticsearch processes and delete programs and all data**

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
