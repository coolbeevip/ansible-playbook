# Ansible Playbook 安装 AntDB 集群 | [English](README.md)

AntDB分布式数据库自动化安装脚本，此脚本仅供测试环境搭建使用

* MGR 集群的管理
* GTM 全局事务管理
* Coordinator 协调员管理用户会话
* Data Node 数据节点

## 安装计划

服务器规划

| IP | SSH 端口 | SSH 用户名 | SSH 密码 | ROOT 密码 |
| ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | 22022 | antdb | 123456 | root123 |
| 10.1.207.181 | 22022 | antdb | 123456 | root123 |
| 10.1.207.182 | 22022 | antdb | 123456 | root123 |

集群节点规划

| IP | MGR | GTM | Coordinator | DataNode |
| ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | Master | Master | Coordinator_1 | DataNode_Master_1, DataNode_Slave_3 |
| 10.1.207.181 | Slave_1 | | Coordinator_2 | DataNode_Master_2, DataNode_Slave_1 |
| 10.1.207.182 | | Slave_1 | | DataNode_Master_3, DataNode_Slave_2 |

节点安装路径

| 路径 | 描述 |
| ---- | ---- |
| /opt/antdb | rpm 安装介质存放路径 |
| ~/antdb_uninstall.sh | 集群卸载脚本 |
| /data01/antdb/app | PG 程序文件 |
| /data01/antdb/mgr | MGR 程序文件 |
| /data01/antdb/data | 数据文件目录 |
| /data01/antdb/tools | 工具脚本存放路径 |
| /data01/antdb/core | Core Dump 文件存放路径 |

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

下载 AntDB 安装包 `antdb.cluster-5.0.009be78c-centos7.9.rpm` 到 `~/my-docker-volume/ansible-playbook/packages` 目录

## 配置安装脚本

> 您可以编辑以下配置文件，修改默认参数


#### main.yml

安装 AntDB 集群的所有服务器 IP 地址，以及安装用系统用户名

```shell
- hosts: 10.1.207.180
  user: antdb

- hosts: 10.1.207.181
  user: antdb

- hosts: 10.1.207.182
  user: antdb
```

操作系统内核参数

```shell
limits_hard_nproc: '65535'
limits_soft_nproc: '65535'
limits_hard_nofile: '278528'
limits_soft_nofile: '278528'
limits_soft_stack: 'unlimited'
limits_soft_core: 'unlimited'
limits_hard_core: 'unlimited'
limits_soft_memlock: '250000000'
limits_hard_memlock: '250000000'
```

安装用系统用户名和密码

```shell
antdb_user: 'antdb'
antdb_group: 'antdb'
antdb_password: '123456'
```

AntDB rpm 包名称

```shell
antdb_tar: 'antdb.cluster-5.0.009be78c-centos7.9.rpm'
```

AntDB 安装路径

```shell
antdb_home_dir: '/opt/antdb'
antdb_mgr_dir: '/data01/antdb/mgr'
antdb_data_dir: '/data01/antdb/data'
antdb_tools_dir: '/data01/antdb/tools'
antdb_app_dir: '/data01/antdb/app'
antdb_core_dir: '/data01/antdb/core'
```

postgresql.conf 扩展参数

```shell
antdb_conf_listen_addresses: '*'
antdb_conf_port: 16432
antdb_conf_log_destination: 'csvlog'
antdb_conf_logging_collector: on
antdb_conf_log_directory: 'pg_log'
antdb_conf_log_rotation_size: 100MB
antdb_conf_log_min_messages: error
antdb_conf_log_statement: ddl
```

集群所有节点名称(此名称不是主机名，是集群管理用的节点名称，不能包含中划线)

```shell
# 集群节点名称（注意这不是主机名)
node_names:
  10.1.207.180:
    name: antdb180
  10.1.207.181:
    name: antdb181
  10.1.207.182:
    name: antdb182
```    

集群各个组件端口

```shell
# agent
andb_agent_port: 18432
# GTM
antdb_gtm_port: 16655
# Coordinator
antdb_coordinator_port: 15432
# DataNode
antdb_datanode_master_port: 14332
antdb_datanode_slave_port: 14333
```

MGR 节点配置

```shell
mgr_nodes:
  master:
    ip: 10.1.207.180
  slave:
    ips: [10.1.207.182]
```

GTM 节点配置

```shell
gtm_nodes:
  master:
    name: gtm_master
    node: antdb180
  slaves:
    - name: gtm_slave_1
      node: antdb181
```

Coordinator 节点配置

```shell
coordinator_nodes:
  - name: coordinator_1
    node: antdb181
  - name: coordinator_2
    node: antdb182
```

DataNodes 节点配置

```shell
data_nodes:
  - master:
      name: dn_master_1
      node: antdb180
    slave:
      name: dn_slave_1
      node: antdb181
  - master:
      name: dn_master_2
      node: antdb181
    slave:
      name: dn_slave_2
      node: antdb182
  - master:
      name: dn_master_3
      node: antdb182
    slave:
      name: dn_slave_3
      node: antdb180
```

## 开始安装

启动 ansible 容器工具连接目标服务器，并将 `~/my-docker-volume/ansible-playbook` 目录挂载到容器中。

**提示：** ANSIBLE_SSH_USERS，ANSIBLE_SSH_PASSS 配置成您之前在目标服务器上创建的用户名 `antdb` 和密码 `123456`

**提示：** ANSIBLE_SU_PASSS 为 root 用户的密码

```shell
docker run --name ansible --rm -it \
  -e ANSIBLE_SSH_HOSTS=10.1.207.180,10.1.207.181,10.1.207.182 \
  -e ANSIBLE_SSH_PORTS=22022,22022,22022 \
  -e ANSIBLE_SSH_USERS=antdb,antdb,antdb \
  -e ANSIBLE_SSH_PASSS=123456,123456,123456 \
  -e ANSIBLE_SU_PASSS=root123,root123,root123 \
  -v /Users/zhanglei/mydocker/volume/ansible-playbook:/ansible-playbook \
  coolbeevip/ansible:2.8.11-alpine \
  /bin/bash  
```

## 安装 Antdb 集群

安装前初始化，设置内核参数，关闭防火墙，创建安装目录，上传安装介质

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/antdb/main-os-init.yml
```

**提示：** 此脚本在我的环境下执行耗时大约 2 分钟

安装并启动集群

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/antdb/main-cluster-install.yml
```

**提示：** 此脚本在我的环境下执行耗时大约 2 分钟

安装完毕，连接 MGR 主节点 **10.1.207.180** 检查集群状态，可以看到所有节点都已经 `running`

```shell
bash-5.0# ansible 10.1.207.180 -m shell -a 'psql -p 16432 -d postgres -c "monitor all;"'
10.1.207.180 | CHANGED | rc=0 >>
   nodename    |      nodetype      | status | description |     host     | port  | recovery |           boot time           | nodezone
---------------+--------------------+--------+-------------+--------------+-------+----------+-------------------------------+----------
 gtm_master    | gtmcoord master    | t      | running     | 10.1.207.180 | 16655 | false    | 2021-11-29 15:35:37.239812+08 | local
 gtm_slave_1   | gtmcoord slave     | t      | running     | 10.1.207.181 | 16655 | true     | 2021-11-29 15:31:55.145147+08 | local
 coordinator_1 | coordinator master | t      | running     | 10.1.207.181 | 15432 | false    | 2021-11-29 15:32:02.253647+08 | local
 coordinator_2 | coordinator master | t      | running     | 10.1.207.182 | 15432 | false    | 2021-11-29 15:31:09.045575+08 | local
 dn_master_1   | datanode master    | t      | running     | 10.1.207.180 | 14332 | false    | 2021-11-29 15:35:57.878057+08 | local
 dn_master_2   | datanode master    | t      | running     | 10.1.207.181 | 14332 | false    | 2021-11-29 15:32:11.965045+08 | local
 dn_master_3   | datanode master    | t      | running     | 10.1.207.182 | 14332 | false    | 2021-11-29 15:31:18.718835+08 | local
 dn_slave_1    | datanode slave     | t      | running     | 10.1.207.180 | 14333 | true     | 2021-11-29 15:36:01.301736+08 | local
 dn_slave_2    | datanode slave     | t      | running     | 10.1.207.181 | 14333 | true     | 2021-11-29 15:32:18.017186+08 | local
 dn_slave_3    | datanode slave     | t      | running     | 10.1.207.182 | 14333 | true     | 2021-11-29 15:31:27.937421+08 | local
(10 rows)
```

通过 `Coordinator` 节点端口，测试连接集群是否正常

```shell
bash-5.0# ansible 10.1.207.180 -m shell -a 'psql -h 10.1.207.181 -p 15432 -c "SELECT datname FROM pg_database;"'
10.1.207.180 | CHANGED | rc=0 >>
  datname
-----------
 postgres
 antdb
 template1
 template0
(4 rows)
```

## 常用运维命令

连接 MGR 主节点，查看集群主机列表

```shell
bash-5.0# ansible 10.1.207.180 -m shell -a 'psql -p 16432 -d postgres -c "list host;"'
10.1.207.180 | CHANGED | rc=0 >>
   name   | user  | port  | protocol | agentport |   address    |      adbhome
----------+-------+-------+----------+-----------+--------------+-------------------
 antdb180 | antdb | 22022 | ssh      |     18432 | 10.1.207.180 | /data01/antdb/app
 antdb181 | antdb | 22022 | ssh      |     18432 | 10.1.207.181 | /data01/antdb/app
 antdb182 | antdb | 22022 | ssh      |     18432 | 10.1.207.182 | /data01/antdb/app
(3 rows)
```

连接 MGR 主节点，查看集群节点列表

```shell
bash-5.0# ansible 10.1.207.180 -m shell -a 'psql -p 16432 -d postgres -c "list node;"'
10.1.207.180 | CHANGED | rc=0 >>
     name      |   host   |        type        | mastername  | port  | sync_state |               path               | initialized | incluster | zone
---------------+----------+--------------------+-------------+-------+------------+----------------------------------+-------------+-----------+-------
 gtm_master    | antdb180 | gtmcoord master    |             | 16655 |            | /data01/antdb/data/gtm_master    | t           | t         | local
 gtm_slave_1   | antdb181 | gtmcoord slave     | gtm_master  | 16655 | sync       | /data01/antdb/data/gtm_slave_1   | t           | t         | local
 coordinator_1 | antdb181 | coordinator master |             | 15432 |            | /data01/antdb/data/coordinator_1 | t           | t         | local
 coordinator_2 | antdb182 | coordinator master |             | 15432 |            | /data01/antdb/data/coordinator_2 | t           | t         | local
 dn_master_1   | antdb180 | datanode master    |             | 14332 |            | /data01/antdb/data/dn_master_1   | t           | t         | local
 dn_master_2   | antdb181 | datanode master    |             | 14332 |            | /data01/antdb/data/dn_master_2   | t           | t         | local
 dn_master_3   | antdb182 | datanode master    |             | 14332 |            | /data01/antdb/data/dn_master_3   | t           | t         | local
 dn_slave_1    | antdb180 | datanode slave     | dn_master_1 | 14333 | sync       | /data01/antdb/data/dn_slave_1    | t           | t         | local
 dn_slave_2    | antdb181 | datanode slave     | dn_master_2 | 14333 | sync       | /data01/antdb/data/dn_slave_2    | t           | t         | local
 dn_slave_3    | antdb182 | datanode slave     | dn_master_3 | 14333 | sync       | /data01/antdb/data/dn_slave_3    | t           | t         | local
(10 rows)
```

查看所有服务器上 AntDB 集群相关的进程

```shell
bash-5.0# ansible all -m shell -a 'ps -ef | grep /data01/antdb/app/bin'
10.1.207.181 | CHANGED | rc=0 >>
antdb    20804     1  0 10:44 ?        00:00:00 /data01/antdb/app/bin/agent -b -P 18432
antdb    20840     1  0 10:44 ?        00:00:00 /data01/antdb/app/bin/postgres --gtm_coord -D /data01/antdb/data/gtm_slave_1 -i
antdb    20899     1  0 10:44 ?        00:00:00 /data01/antdb/app/bin/postgres --coordinator -D /data01/antdb/data/coordinator_1 -i
antdb    20994     1  0 10:44 ?        00:00:00 /data01/antdb/app/bin/postgres --datanode -D /data01/antdb/data/dn_master_2 -i
antdb    21047     1  0 10:44 ?        00:00:00 /data01/antdb/app/bin/postgres --datanode -D /data01/antdb/data/dn_slave_2 -i
antdb    31323 31322  0 11:35 pts/3    00:00:00 /bin/sh -c ps -ef | grep /data01/antdb/app/bin
antdb    31325 31323  0 11:35 pts/3    00:00:00 grep /data01/antdb/app/bin

10.1.207.180 | CHANGED | rc=0 >>
antdb    16992     1  0 10:46 ?        00:00:00 /data01/antdb/app/bin/adbmgrd -D /data01/antdb/mgr
antdb    18118     1  0 10:47 ?        00:00:00 /data01/antdb/app/bin/agent -b -P 18432
antdb    18253     1  0 10:47 ?        00:00:00 /data01/antdb/app/bin/postgres --gtm_coord -D /data01/antdb/data/gtm_master -i
antdb    18426     1  0 10:48 ?        00:00:00 /data01/antdb/app/bin/postgres --datanode -D /data01/antdb/data/dn_master_1 -i
antdb    18479     1  0 10:48 ?        00:00:00 /data01/antdb/app/bin/postgres --datanode -D /data01/antdb/data/dn_slave_1 -i
antdb    28185 28184  0 11:39 pts/2    00:00:00 /bin/sh -c ps -ef | grep /data01/antdb/app/bin
antdb    28187 28185  0 11:39 pts/2    00:00:00 grep /data01/antdb/app/bin

10.1.207.182 | CHANGED | rc=0 >>
antdb    20644     1  0 10:43 ?        00:00:00 /data01/antdb/app/bin/agent -b -P 18432
antdb    20695     1  0 10:43 ?        00:00:00 /data01/antdb/app/bin/postgres --coordinator -D /data01/antdb/data/coordinator_2 -i
antdb    20751     1  0 10:43 ?        00:00:00 /data01/antdb/app/bin/postgres --datanode -D /data01/antdb/data/dn_master_3 -i
antdb    20837     1  0 10:43 ?        00:00:00 /data01/antdb/app/bin/postgres --datanode -D /data01/antdb/data/dn_slave_3 -i
antdb    28825 28824  0 11:34 pts/3    00:00:00 /bin/sh -c ps -ef | grep /data01/antdb/app/bin
antdb    28827 28825  0 11:34 pts/3    00:00:00 grep /data01/antdb/app/bin
```

## Q & A

#### 多服务器如何规划节点类型

> 生产环境请以官方建议为准

3 服务器推荐规划

| IP | MGR | GTM | Coordinator | DataNode |
| ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | Master | Master | Coordinator_1 | DataNode_Master_1, DataNode_Slave_3 |
| 10.1.207.181 | Slave_1 | | Coordinator_2 | DataNode_Master_2, DataNode_Slave_1 |
| 10.1.207.182 | | Slave_1 | | DataNode_Master_3, DataNode_Slave_2 |


4 服务器推荐规划

| IP | MGR | GTM | Coordinator | DataNode |
| ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | Master | | | DataNode_Master_1, DataNode_Slave_3 |
| 10.1.207.181 | Slave_1 | | Coordinator_1 | DataNode_Master_2, DataNode_Slave_1 |
| 10.1.207.182 | | Master | | DataNode_Slave_2 |
| 10.1.207.183 | | Slave_1 | Coordinator_2 | DataNode_Master_3 |


5 服务器推荐规划

| IP | MGR | GTM | Coordinator | DataNode |
| ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | | Slave_1 | Coordinator_1 | DataNode_Master_1, DataNode_Slave_5 |
| 10.1.207.181 | | Slave_2 | Coordinator_2 | DataNode_Master_2, DataNode_Slave_1 |
| 10.1.207.182 | Slave_1 | Master | Coordinator_3 | DataNode_Master_3, DataNode_Slave_2 |
| 10.1.207.183 | Slave_2 | | Coordinator_4 | DataNode_Master_4, DataNode_Slave_3 |
| 10.1.207.184 | Master | | Coordinator_5 | DataNode_Master_5, DataNode_Slave_4 |

#### 如何强制卸载 AntDB 分布式集群

```shell
bash-5.0# ansible all -m shell -a '~/antdb_uninstall.sh'
```

#### 如何正常卸载 AntDB 分布式集群

连接 MGR 主节点，停止所有节点服务

```shell
bash-5.0# ansible 10.1.207.180 -m shell -a 'psql -p 16432 -d postgres -c "stop all mode f;"'
10.1.207.180 | CHANGED | rc=0 >>
     operation type      |   nodename    | status | description
-------------------------+---------------+--------+-------------
 stop datanode slave     | dn_slave_1    | t      | success
 stop datanode slave     | dn_slave_2    | t      | success
 stop datanode slave     | dn_slave_3    | t      | success
 stop datanode master    | dn_master_1   | t      | success
 stop datanode master    | dn_master_2   | t      | success
 stop datanode master    | dn_master_3   | t      | success
 stop coordinator master | coordinator_1 | t      | success
 stop coordinator master | coordinator_2 | t      | success
 stop gtmcoord slave     | gtm_slave_1   | t      | success
 stop gtmcoord master    | gtm_master    | t      | success
(10 rows)NOTICE:  [SUCCESS] host(10.1.207.180) cmd(STOP DATANODE BACKEND) params( stop -D /data01/antdb/data/dn_slave_1 -Z datanode -m fast -o -i -c -W).
NOTICE:  [SUCCESS] host(10.1.207.181) cmd(STOP DATANODE BACKEND) params( stop -D /data01/antdb/data/dn_slave_2 -Z datanode -m fast -o -i -c -W).
NOTICE:  [SUCCESS] host(10.1.207.182) cmd(STOP DATANODE BACKEND) params( stop -D /data01/antdb/data/dn_slave_3 -Z datanode -m fast -o -i -c -W).
NOTICE:  waiting max 90 seconds for datanode slave to stop ...

NOTICE:  [SUCCESS] host(10.1.207.180) cmd(STOP DATANODE BACKEND) params( stop -D /data01/antdb/data/dn_master_1 -Z datanode -m fast -o -i -c -W).
NOTICE:  [SUCCESS] host(10.1.207.181) cmd(STOP DATANODE BACKEND) params( stop -D /data01/antdb/data/dn_master_2 -Z datanode -m fast -o -i -c -W).
NOTICE:  [SUCCESS] host(10.1.207.182) cmd(STOP DATANODE BACKEND) params( stop -D /data01/antdb/data/dn_master_3 -Z datanode -m fast -o -i -c -W).
NOTICE:  waiting max 90 seconds for datanode master to stop ...

NOTICE:  [SUCCESS] host(10.1.207.181) cmd(STOP COORD BACKEND) params( stop -D /data01/antdb/data/coordinator_1 -Z coordinator -m fast -o -i -c -W).
NOTICE:  [SUCCESS] host(10.1.207.182) cmd(STOP COORD BACKEND) params( stop -D /data01/antdb/data/coordinator_2 -Z coordinator -m fast -o -i -c -W).
NOTICE:  waiting max 90 seconds for coordinator master to stop ...

NOTICE:  [SUCCESS] host(10.1.207.181) cmd(STOP GTMCOORD SLAVE BACKEND) params( stop -D /data01/antdb/data/gtm_slave_1 -Z gtm_coord -m fast -o -i -c -W).
NOTICE:  waiting max 90 seconds for gtmcoord slave to stop ...

NOTICE:  [SUCCESS] host(10.1.207.180) cmd(STOP COORD BACKEND) params( stop -D /data01/antdb/data/gtm_master -Z coordinator -m fast -o -i -c -W).
NOTICE:  waiting max 90 seconds for gtmcoord master to stop ...
```

连接 MGR 主节点，清理数据

```shell
bash-5.0# ansible 10.1.207.180 -m shell -a 'psql -p 16432 -d postgres -c "clean all;"'
10.1.207.180 | CHANGED | rc=0 >>
   nodename    |      nodetype      | status | description
---------------+--------------------+--------+-------------
 dn_slave_1    | datanode slave     | t      | success
 dn_slave_2    | datanode slave     | t      | success
 dn_slave_3    | datanode slave     | t      | success
 dn_master_1   | datanode master    | t      | success
 dn_master_2   | datanode master    | t      | success
 dn_master_3   | datanode master    | t      | success
 coordinator_1 | coordinator master | t      | success
 coordinator_2 | coordinator master | t      | success
 gtm_slave_1   | gtm slave          | t      | success
 gtm_master    | gtm master         | t      | success
(10 rows)
```

连接 MGR 主节点，停止所有节点 Agent

```shell
bash-5.0# ansible 10.1.207.180 -m shell -a 'psql -p 16432 -d postgres -c "stop agent all;"'
10.1.207.180 | CHANGED | rc=0 >>
 nodename | status | description
----------+--------+-------------
 antdb180 | t      | success
 antdb181 | t      | success
 antdb182 | t      | success
(3 rows)
```

查询所有服务器上 AntDB 进程都已经停止

```shell
bash-5.0# ansible all -m shell -a 'ps -ef | grep /data01/antdb/app/bin'
10.1.207.180 | CHANGED | rc=0 >>
antdb    16992     1  0 10:46 ?        00:00:00 /data01/antdb/app/bin/adbmgrd -D /data01/antdb/mgr
antdb    29175 29174 27 11:42 pts/2    00:00:00 /bin/sh -c ps -ef | grep /data01/antdb/app/bin
antdb    29177 29175  0 11:42 pts/2    00:00:00 grep /data01/antdb/app/bin

10.1.207.181 | CHANGED | rc=0 >>
antdb    31995 31994  0 11:38 pts/3    00:00:00 /bin/sh -c ps -ef | grep /data01/antdb/app/bin
antdb    31998 31995  0 11:38 pts/3    00:00:00 grep /data01/antdb/app/bin

10.1.207.182 | CHANGED | rc=0 >>
antdb    29370 29369  0 11:37 pts/3    00:00:00 /bin/sh -c ps -ef | grep /data01/antdb/app/bin
antdb    29372 29370  0 11:37 pts/3    00:00:00 grep /data01/antdb/app/bin
```

查询所有服务器上的数据文件都已经删除

```shell
bash-5.0# ansible all -m shell -a 'ls /data01/antdb'
10.1.207.180 | CHANGED | rc=0 >>


10.1.207.181 | CHANGED | rc=0 >>


10.1.207.182 | CHANGED | rc=0 >>
```
