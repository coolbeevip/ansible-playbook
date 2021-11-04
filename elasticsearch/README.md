# Ansible Playbook Install Elasticsearch Cluster | [中文](README_ZH.md)

Please create a `elasticsearch` user and defautl password is `123456` on the linux server before starting. This playbook script install three node clusters.
Since the 7.x version will include the JDK, there is no need to install the JDK in advance.

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
wget -P ~/my-docker-volume/ansible-playbook/packages https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.13.3-linux-x86_64.tar.gz --no-check-certificate
```

## Configuration

The following parameters need to be configured in the actual environment

#### vars_elasticsearch.yml

This file defines variable information such as installation path, data path, installation package file name, port, memory, etc. You can modify these variables according to the actual environment. These variables will be replaced in the following files when they are deployed.

* elasticsearch-7.13.3/config/elasticsearch.yml
* elasticsearch-7.13.3/config/jvm.options
* elasticsearch-7.13.3/config/jvm.options.d/gc.options

#### main.yml

This file mainly defines the address of the target server, the login user name, and the name of each Elasticsearch node

```yaml
- hosts: 10.1.207.180
  user: elasticsearch
  ...
  vars:
    es_node_name: "node-180"

- hosts: 10.1.207.181
  user: elasticsearch
  ...
  vars:
    es_node_name: "node-181"

- hosts: 10.1.207.182
  user: elasticsearch
  ...
  vars:
    es_node_name: "node-182"
```

## Install

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

Run Ansible playbook scripts to install

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/elasticsearch/main.yml
```

At this point, the installation is complete, and you can see the Elasticsearch node that has been configured in the installation directory of each target service.

You can use the following script to delete the temporary files generated during the installation

```shell
bash-5.0# ansible all -m shell -a "rm -rf /opt/elasticsearch/elasticsearch-7.13.3-linux-x86_64.tar.gz"
```

## Start & Stop

### Start Elasticsearch Cluster

```shell
bash-5.0# ansible all -m shell -a 'sh /opt/elasticsearch/elasticsearch-7.13.3/bin/es.sh start'
```

### Check Elasticsearch Service Info

Use API to view service status

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

Check Cluster Status

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

### Stop Elasticsearch Cluster

```shell
bash-5.0# ansible all -m shell -a 'sh /opt/elasticsearch/elasticsearch-7.13.3/bin/es.sh stop'
```
