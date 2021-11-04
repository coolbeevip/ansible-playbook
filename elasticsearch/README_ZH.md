# Ansible Playbook 安装 Elasticsearch 集群 | [English](README.md)

请先在目标服务器创建 `elasticsearch` 用户，脚本会在此用户下安装三节点集群，因为 7.X 版本后会自带 use the bundled JDK JDK，所以我们不需要提前安装 Java 环境。

## 下载安装包和 Playbook 脚本

在客户机上创建 playbook 脚本存放目录

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


## 开始安装

启动 ansible 容器工具连接目标服务器，并将 `~/my-docker-volume/ansible-playbook` 目录挂在到容器中，**注意这里 ANSIBLE_SSH_USERS，ANSIBLE_SSH_PASSS 配置为 elasticsearch 用户名密码**

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

执行安装脚本

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/elasticsearch/main.yml
```

至此，已经安装完毕，您可以在每个目标服务的安装目录下看到已经配置完毕的 elasticsearch 节点。

您可以使用以下脚本删除不在使用的安装包

```shell
bash-5.0# ansible all -m shell -a "rm -rf /opt/elasticsearch/elasticsearch-7.13.3-linux-x86_64.tar.gz"
```

## 启动 & 停止

### 启动集群

```shell
bash-5.0# ansible all -m shell -a 'sh /opt/elasticsearch/elasticsearch-7.13.3/bin/es.sh start'
```

### 检查状态

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

### 停止集群

```shell
bash-5.0# ansible all -m shell -a 'sh /opt/elasticsearch/elasticsearch-7.13.3/bin/es.sh stop'
```
