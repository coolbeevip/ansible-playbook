# Ansible Playbook 自动化安装 AntDB 集群 | [English](README.md)

> 6 分钟自动化安装一个 3 节点 AntDB 分布式数据库集群，能够自动配置内核参数、创建目录、初始化集群配置并启动集群。此脚本仅供测试环境搭建使用，生产环境请参考官方安装文档。

包含组件

* MGR 集群的管理
* GTM 全局事务管理
* Coordinator 协调员管理用户会话
* Data Node 数据节点

## 安装计划

服务器规划

| IP | SSH 端口 | SSH 用户名 | SSH 密码 | ROOT 密码 | OS |
| ---- | ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | 22022 | antdb | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.181 | 22022 | antdb | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.182 | 22022 | antdb | 123456 | root123 | CentOS Linux release 7.9.2009 |

**提示：** 可以参考[批量自动化创建用户](https://github.com/coolbeevip/ansible-playbook/blob/main/README_ZH.md#%E5%88%9B%E5%BB%BA%E7%94%A8%E6%88%B7%E5%92%8C%E7%BB%84)

集群节点规划

| IP | MGR | GTM | Coordinator | DataNode |
| ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | Master | Master | | DataNode_Master_1, DataNode_Slave_3 |
| 10.1.207.181 | | Slave_1 | Coordinator_1 | DataNode_Master_2, DataNode_Slave_1 |
| 10.1.207.182 | Slave_1 | | Coordinator_2 | DataNode_Master_3, DataNode_Slave_2 |

节点安装路径

| 路径 | 描述 |
| ---- | ---- |
| /opt/antdb | rpm 安装介质存放路径 |
| ~/antdb_uninstall.sh | 强制集群卸载脚本 |
| ~/antdb_config.sql | 数据库参数脚本 |
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

#### maim-os-init.yml

配置安装 AntDB 集群的所有服务器 IP 地址，以及安装用系统用户名，此脚本用来设置内核参数，创建目录、上传安装介质

```yaml
- hosts: 10.1.207.180
  user: antdb

- hosts: 10.1.207.181
  user: antdb

- hosts: 10.1.207.182
  user: antdb
```

#### maim-cluster-install.yml

配置 AntDB MGR 主/备节点服务器 IP 地址，以及安装用系统用户名。备节点的名称 `mgr_slave_1`

```yaml
# MGR Master Node Initialize
- hosts: 10.1.207.180
  user: antdb

# MGR Slave Node Initialize
- hosts: 10.1.207.182
  user: antdb
  vars:
    mgr_slave_name: "mgr_slave_1"

# Restart AntDB All NOdes on MGR Master Node
- hosts: 10.1.207.180
  user: antdb
```

#### var_antdb.yml

操作系统内核参数

```yaml
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

```yaml
antdb_user: 'antdb'
antdb_group: 'antdb'
antdb_password: '123456'
```

AntDB rpm 包名称

```yaml
antdb_tar: 'antdb.cluster-5.0.009be78c-centos7.9.rpm'
```

AntDB 安装路径

```yaml
antdb_home_dir: '/opt/antdb'
antdb_mgr_dir: '/data01/antdb/mgr'
antdb_data_dir: '/data01/antdb/data'
antdb_tools_dir: '/data01/antdb/tools'
antdb_app_dir: '/data01/antdb/app'
antdb_core_dir: '/data01/antdb/core'
```

postgresql.conf 扩展参数

```yaml
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

```yaml
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

```yaml
# Agent
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

```yaml
mgr_nodes:
  master:
    ip: 10.1.207.180
  slave:
    ips: [10.1.207.182]
```

GTM 节点配置

```yaml
gtm_nodes:
  master:
    name: gtm_master
    node: antdb180
  slaves:
    - name: gtm_slave_1
      node: antdb181
```

Coordinator 节点配置

```yaml
coordinator_nodes:
  - name: coordinator_1
    node: antdb181
  - name: coordinator_2
    node: antdb182
```

DataNodes 节点配置

```yaml
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

数据库配置参数

```yaml
antdb_set:
  datanode:
    max_connections: 1000 # 自定义最大连接数
    max_prepared_transactions: 1000 # 等于最大连接数
    max_worker_processes: 108 # cpu*2，不能小于 108
    shared_buffers: 2GB # 物理内存 * 25% GB
    effective_cache_size: 3GB # 物理内存 * 75% GB
    max_wal_size: 1GB # 2 * shared_buffers GB
    random_page_cost: 4 # 如果是SSD磁盘，设置为1；如果是SATA磁盘，保持默认值4
  coordinator:
    max_connections: 1000 # 最大连接数
    max_prepared_transactions: 1000 # 等于最大连接数
    max_worker_processes: 108 # cpu*2，不能小于 108
  gtmcoord:
    max_connections: 1000 # 最大连接数
    max_prepared_transactions: 1000 # 等于最大连接数
    max_worker_processes: 108 # cpu*2，不能小于 108
    shared_buffers: 5GB # 物理内存 * 25% GB
```    

更多默认配置参见 `antdb_config.sql.j2` 文件

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

## 安装集群

系统初始化，设置内核参数，关闭防火墙，创建安装目录，上传安装介质，启动并初始化集群

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/antdb/main-os-init.yml /ansible-playbook/antdb/main-cluster-install.yml
```

如果你看到如下信息，说明安装完成

```shell
TASK [Install Succeed] ********************************************************************************************************************************************************************************************************
ok: [10.1.207.180] => {
    "msg": "Install Succeed!"
}
```

**提示：** 此脚本在我的环境下执行耗时大约 6 分钟

## 验证集群

安装完毕，登录到 MGR 主节点 **10.1.207.180** 检查集群状态，可以看到所有节点都已经 `running`

```shell
bash-5.0# ansible 10.1.207.180 -m shell -a 'psql -p 16432 -d postgres -c "monitor all;"'
10.1.207.180 | CHANGED | rc=0 >>
   nodename    |      nodetype      | status | description |     host     | port  | recovery |           boot time           | nodezone
---------------+--------------------+--------+-------------+--------------+-------+----------+-------------------------------+----------
 gtm_master    | gtmcoord master    | t      | running     | 10.1.207.180 | 16655 | false    | 2021-11-30 13:39:48.610397+08 | local
 gtm_slave_1   | gtmcoord slave     | t      | running     | 10.1.207.181 | 16655 | true     | 2021-11-30 13:36:06.056928+08 | local
 coordinator_1 | coordinator master | t      | running     | 10.1.207.181 | 15432 | false    | 2021-11-30 13:36:12.660815+08 | local
 coordinator_2 | coordinator master | t      | running     | 10.1.207.182 | 15432 | false    | 2021-11-30 13:35:19.633406+08 | local
 dn_master_1   | datanode master    | t      | running     | 10.1.207.180 | 14332 | false    | 2021-11-30 13:40:08.57693+08  | local
 dn_master_2   | datanode master    | t      | running     | 10.1.207.181 | 14332 | false    | 2021-11-30 13:36:22.518815+08 | local
 dn_master_3   | datanode master    | t      | running     | 10.1.207.182 | 14332 | false    | 2021-11-30 13:35:29.467795+08 | local
 dn_slave_1    | datanode slave     | t      | running     | 10.1.207.180 | 14333 | true     | 2021-11-30 13:40:12.453562+08 | local
 dn_slave_2    | datanode slave     | t      | running     | 10.1.207.181 | 14333 | true     | 2021-11-30 13:36:29.635405+08 | local
 dn_slave_3    | datanode slave     | t      | running     | 10.1.207.182 | 14333 | true     | 2021-11-30 13:35:39.245053+08 | local
(10 rows)
```

登录到 MGR 主节点 **10.1.207.180**，通过 `Coordinator` 节点端口，测试连接集群是否正常

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

登录到 MGR 备节点 **10.1.207.182** 检查集群状态，可以看到所有节点都已经 `running`，说明 MGR 备节点可以正常工作

```shell
bash-5.0# ansible 10.1.207.182 -m shell -a 'psql -p 16432 -d postgres -c "monitor all;"'
10.1.207.180 | CHANGED | rc=0 >>
   nodename    |      nodetype      | status | description |     host     | port  | recovery |           boot time           | nodezone
---------------+--------------------+--------+-------------+--------------+-------+----------+-------------------------------+----------
 gtm_master    | gtmcoord master    | t      | running     | 10.1.207.180 | 16655 | false    | 2021-11-30 13:39:48.610397+08 | local
 gtm_slave_1   | gtmcoord slave     | t      | running     | 10.1.207.181 | 16655 | true     | 2021-11-30 13:36:06.056928+08 | local
 coordinator_1 | coordinator master | t      | running     | 10.1.207.181 | 15432 | false    | 2021-11-30 13:36:12.660815+08 | local
 coordinator_2 | coordinator master | t      | running     | 10.1.207.182 | 15432 | false    | 2021-11-30 13:35:19.633406+08 | local
 dn_master_1   | datanode master    | t      | running     | 10.1.207.180 | 14332 | false    | 2021-11-30 13:40:08.57693+08  | local
 dn_master_2   | datanode master    | t      | running     | 10.1.207.181 | 14332 | false    | 2021-11-30 13:36:22.518815+08 | local
 dn_master_3   | datanode master    | t      | running     | 10.1.207.182 | 14332 | false    | 2021-11-30 13:35:29.467795+08 | local
 dn_slave_1    | datanode slave     | t      | running     | 10.1.207.180 | 14333 | true     | 2021-11-30 13:40:12.453562+08 | local
 dn_slave_2    | datanode slave     | t      | running     | 10.1.207.181 | 14333 | true     | 2021-11-30 13:36:29.635405+08 | local
 dn_slave_3    | datanode slave     | t      | running     | 10.1.207.182 | 14333 | true     | 2021-11-30 13:35:39.245053+08 | local
(10 rows)
```

## 添加客户端白名单

登录到 MGR 主节点 **10.1.207.180**，增加客户端白名单，否则你连接的时候将会收到 **[28000] FATAL: no pg_hba.conf entry for host "x.x.x.x", user "xxx", database "xxx"** 类似的错误。命令格式：`add hba coordinator all ("host <database> <user> <ip-address> <ip-mark> <auth-method>");`

例如：添加 C类地址段 10.4.16.0～10.4.16.255

```shell
bash-5.0# ansible 10.1.207.180 -m shell -a 'psql -p 16432 -d postgres -c "add hba coordinator all (\"host all all 10.4.16.0 24 md5\");"'
10.1.207.180 | CHANGED | rc=0 >>
 nodename | values
----------+---------
 *        | success
(1 row)
```

例如：添加 B类地址段 10.4.0.0～10.4.255.255

```shell
bash-5.0# ansible 10.1.207.180 -m shell -a 'psql -p 16432 -d postgres -c "add hba coordinator all (\"host all all 10.4.0.0 16 md5\");"'
10.1.207.180 | CHANGED | rc=0 >>
 nodename | values
----------+---------
 *        | success
(1 row)
```

## 创建数据库

SSH 登录到集群任一服务器，然后使用 `psql -h 10.1.207.181 -p 15432` 命令连接 `Coordinator` 主(备)节点

```sql
[antdb@oss-irms-180 ~]$ psql -h 10.1.207.181 -p 15432
psql (5.0.1 based on PG 11.10)
Type "help" for help.

antdb=# create database testdb;
CREATE DATABASE

antdb=# create user testdbuser with encrypted password 'testdbpass';
CREATE ROLE

antdb=# grant all privileges on database testdb to testdbuser;
GRANT
```

通过 Coordinator 主节 **10.1.207.181** 连接数据库

```shell
[antdb@oss-irms-180 ~]$ psql -h 10.1.207.181 -p 15432 -d testdb -U testdbuser -w testdbpass
psql: warning: extra command-line argument "testdbpass" ignored
psql (5.0.1 based on PG 11.10)
Type "help" for help.

testdb=>
```

通过 Coordinator 备节点 **10.1.207.182** 连接数据库

```shell
[antdb@oss-irms-180 ~]$ psql -h 10.1.207.182 -p 15432 -d testdb -U testdbuser -w testdbpass
psql: warning: extra command-line argument "testdbpass" ignored
psql (5.0.1 based on PG 11.10)
Type "help" for help.

testdb=>
```

## 常用运维命令

登录到 MGR 主节点 **10.1.207.180**，查看集群主机列表

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

登录到 MGR 主节点 **10.1.207.180**，查看集群节点列表

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
bash-5.0# ansible all -m shell -a 'ps -ef | grep [/]data01/antdb/app/bin'
10.1.207.181 | CHANGED | rc=0 >>
antdb    25907     1  0 13:35 ?        00:00:00 /data01/antdb/app/bin/agent -b -P 18432
antdb    25938     1  0 13:36 ?        00:00:00 /data01/antdb/app/bin/postgres --gtm_coord -D /data01/antdb/data/gtm_slave_1 -i
antdb    25987     1  0 13:36 ?        00:00:00 /data01/antdb/app/bin/postgres --coordinator -D /data01/antdb/data/coordinator_1 -i
antdb    26085     1  0 13:36 ?        00:00:00 /data01/antdb/app/bin/postgres --datanode -D /data01/antdb/data/dn_master_2 -i
antdb    26149     1  0 13:36 ?        00:00:00 /data01/antdb/app/bin/postgres --datanode -D /data01/antdb/data/dn_slave_2 -i

10.1.207.180 | CHANGED | rc=0 >>
antdb    13429     1  0 13:39 ?        00:00:00 /data01/antdb/app/bin/adbmgrd -D /data01/antdb/mgr
antdb    14465     1  0 13:39 ?        00:00:00 /data01/antdb/app/bin/agent -b -P 18432
antdb    14588     1  0 13:39 ?        00:00:00 /data01/antdb/app/bin/postgres --gtm_coord -D /data01/antdb/data/gtm_master -i
antdb    14723     1  0 13:40 ?        00:00:00 /data01/antdb/app/bin/postgres --datanode -D /data01/antdb/data/dn_master_1 -i
antdb    14779     1  0 13:40 ?        00:00:00 /data01/antdb/app/bin/postgres --datanode -D /data01/antdb/data/dn_slave_1 -i

10.1.207.182 | CHANGED | rc=0 >>
antdb    18597     1  0 13:35 ?        00:00:00 /data01/antdb/app/bin/agent -b -P 18432
antdb    18643     1  0 13:35 ?        00:00:00 /data01/antdb/app/bin/postgres --coordinator -D /data01/antdb/data/coordinator_2 -i
antdb    18722     1  0 13:35 ?        00:00:00 /data01/antdb/app/bin/postgres --datanode -D /data01/antdb/data/dn_master_3 -i
antdb    18775     1  0 13:35 ?        00:00:00 /data01/antdb/app/bin/postgres --datanode -D /data01/antdb/data/dn_slave_3 -i
```

停止所有节点

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

启动所有节点

```shell
bash-5.0# ansible 10.1.207.180 -m shell -a 'psql -p 16432 -d postgres -c "start all;"'
10.1.207.180 | CHANGED | rc=0 >>
      operation type      |   nodename    | status | description
--------------------------+---------------+--------+-------------
 start gtmcoord master    | gtm_master    | t      | success
 start gtmcoord slave     | gtm_slave_1   | t      | success
 start coordinator master | coordinator_1 | t      | success
 start coordinator master | coordinator_2 | t      | success
 start datanode master    | dn_master_1   | t      | success
 start datanode master    | dn_master_2   | t      | success
 start datanode master    | dn_master_3   | t      | success
 start datanode slave     | dn_slave_1    | t      | success
 start datanode slave     | dn_slave_2    | t      | success
 start datanode slave     | dn_slave_3    | t      | success
(10 rows)NOTICE:  [SUCCESS] host(10.1.207.180) cmd(START GTMCOORD MASTER BACKEND) params( start -D /data01/antdb/data/gtm_master -Z gtm_coord -o -i -c -W -l /data01/antdb/data/gtm_master/logfile).
NOTICE:  waiting max 90 seconds for gtmcoord master to start ...

NOTICE:  [SUCCESS] host(10.1.207.181) cmd(START GTMCOORD SLAVE BACKEND) params( start -D /data01/antdb/data/gtm_slave_1 -Z gtm_coord -o -i -c -W -l /data01/antdb/data/gtm_slave_1/logfile).
NOTICE:  waiting max 90 seconds for gtmcoord slave to start ...

NOTICE:  [SUCCESS] host(10.1.207.181) cmd(START COORD BACKEND) params( start -D /data01/antdb/data/coordinator_1 -Z coordinator -o -i -c -W -l /data01/antdb/data/coordinator_1/logfile).
NOTICE:  [SUCCESS] host(10.1.207.182) cmd(START COORD BACKEND) params( start -D /data01/antdb/data/coordinator_2 -Z coordinator -o -i -c -W -l /data01/antdb/data/coordinator_2/logfile).
NOTICE:  waiting max 90 seconds for coordinator master to start ...

NOTICE:  [SUCCESS] host(10.1.207.180) cmd(START DATANODE BACKEND) params( start -D /data01/antdb/data/dn_master_1 -Z datanode -o -i -c -W -l /data01/antdb/data/dn_master_1/logfile).
NOTICE:  [SUCCESS] host(10.1.207.181) cmd(START DATANODE BACKEND) params( start -D /data01/antdb/data/dn_master_2 -Z datanode -o -i -c -W -l /data01/antdb/data/dn_master_2/logfile).
NOTICE:  [SUCCESS] host(10.1.207.182) cmd(START DATANODE BACKEND) params( start -D /data01/antdb/data/dn_master_3 -Z datanode -o -i -c -W -l /data01/antdb/data/dn_master_3/logfile).
NOTICE:  waiting max 90 seconds for datanode master to start ...

NOTICE:  [SUCCESS] host(10.1.207.180) cmd(START DATANODE BACKEND) params( start -D /data01/antdb/data/dn_slave_1 -Z datanode -o -i -c -W -l /data01/antdb/data/dn_slave_1/logfile).
NOTICE:  [SUCCESS] host(10.1.207.181) cmd(START DATANODE BACKEND) params( start -D /data01/antdb/data/dn_slave_2 -Z datanode -o -i -c -W -l /data01/antdb/data/dn_slave_2/logfile).
NOTICE:  [SUCCESS] host(10.1.207.182) cmd(START DATANODE BACKEND) params( start -D /data01/antdb/data/dn_slave_3 -Z datanode -o -i -c -W -l /data01/antdb/data/dn_slave_3/logfile).
NOTICE:  waiting max 90 seconds for datanode slave to start ...
```

查看节点数据库配置

```shell
bash-5.0# ansible 10.1.207.180 -m shell -a 'psql -p 16432 -d postgres -c "show param dn_master_1 max;"'
10.1.207.180 | CHANGED | rc=0 >>
            type             | status |                             message
-----------------------------+--------+-----------------------------------------------------------------
 datanode master dn_master_1 | t      | autovacuum_freeze_max_age = 200000000                          +
                             |        | autovacuum_max_workers = 5                                     +
                             |        | autovacuum_multixact_freeze_max_age = 400000000                +
                             |        | bgwriter_lru_maxpages = 100                                    +
                             |        | max_cn_prealloc_xid_size = 0                                   +
                             |        | max_connections = 1000                                         +
                             |        | max_coordinators = 16                                          +
                             |        | max_datanodes = 16                                             +
                             |        | max_files_per_process = 1000                                   +
                             |        | max_function_args = 100                                        +
                             |        | max_identifier_length = 63                                     +
                             |        | max_index_keys = 32                                            +
                             |        | max_locks_per_transaction = 256                                +
                             |        | max_logical_replication_workers = 4                            +
                             |        | max_parallel_maintenance_workers = 2                           +
                             |        | max_parallel_workers = 8                                       +
                             |        | max_parallel_workers_per_gather = 2                            +
                             |        | max_pool_size = 100                                            +
                             |        | max_pred_locks_per_page = 2                                    +
                             |        | max_pred_locks_per_relation = -2                               +
                             |        | max_pred_locks_per_transaction = 64                            +
                             |        | max_prepared_transactions = 1000                               +
                             |        | max_replication_slots = 10                                     +
                             |        | max_stack_depth = 8MB                                          +
                             |        | max_standby_archive_delay = 30s                                +
                             |        | max_standby_streaming_delay = 30s                              +
                             |        | max_sync_workers_per_subscription = 2                          +
                             |        | max_wal_senders = 5                                            +
                             |        | max_wal_size = 1GB                                             +
                             |        | max_worker_processes = 32                                      +
                             |        | reduce_scan_max_buckets = 2048                                 +
                             |        | rep_max_avail_flag = off                                       +
                             |        | rep_max_avail_lsn_lag = 8192                                   +
                             |        | use_aux_max_times = 1
 datanode slave dn_slave_1   | f      | could not connect to server: Connection refused                +
                             |        |         Is the server running on host "127.0.0.1" and accepting+
                             |        |         TCP/IP connections on port 14333?                      +
                             |        |
(2 rows)
```

## Q & A

#### 多服务器如何规划节点类型

> 生产环境请以官方建议为准

3 服务器推荐规划

| IP | MGR | GTM | Coordinator | DataNode |
| ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | Master | Master | | DataNode_Master_1, DataNode_Slave_3 |
| 10.1.207.181 | | Slave_1 | Coordinator_1 | DataNode_Master_2, DataNode_Slave_1 |
| 10.1.207.182 | Slave_1 | | Coordinator_2 | DataNode_Master_3, DataNode_Slave_2 |


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

使用强制卸载脚本，此脚本将 kill 所有 AndDB 进程，并删除程序和数据目录

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

#### 从节点启动失败, 日志提示 **hot standby is not possible because xxx = xxx is a lower setting than on the master server**

使用 monitor all 命令，查看到从节点启动失败。

```shell

bash-5.0# ansible 10.1.207.180 -m shell -a 'psql -p 16432 -d postgres -c "monitor all;"'
10.1.207.180 | CHANGED | rc=0 >>
   nodename    |      nodetype      | status | description |     host     | port  | recovery |           boot time           | nodezone
---------------+--------------------+--------+-------------+--------------+-------+----------+-------------------------------+----------
 gtm_master    | gtmcoord master    | t      | running     | 10.1.207.180 | 16655 | false    | 2021-12-01 10:49:12.169417+08 | local
 gtm_slave_1   | gtmcoord slave     | f      | not running | 10.1.207.181 | 16655 | unknown  | unknown                       | local
 coordinator_2 | coordinator master | t      | running     | 10.1.207.182 | 15432 | false    | 2021-12-01 10:46:03.882011+08 | local
 coordinator_1 | coordinator master | t      | running     | 10.1.207.181 | 15432 | false    | 2021-12-01 10:46:56.865553+08 | local
 dn_master_2   | datanode master    | t      | running     | 10.1.207.181 | 14332 | false    | 2021-12-01 10:46:57.42246+08  | local
 dn_master_3   | datanode master    | t      | running     | 10.1.207.182 | 14332 | false    | 2021-12-01 10:46:04.535587+08 | local
 dn_master_1   | datanode master    | t      | running     | 10.1.207.180 | 14332 | false    | 2021-12-01 10:50:43.884491+08 | local
 dn_slave_1    | datanode slave     | f      | not running | 10.1.207.180 | 14333 | unknown  | unknown                       | local
 dn_slave_2    | datanode slave     | f      | not running | 10.1.207.181 | 14333 | unknown  | unknown                       | local
 dn_slave_3    | datanode slave     | f      | not running | 10.1.207.182 | 14333 | unknown  | unknown                       | local
(10 rows)WARNING:  gtmcoord slave gtm_slave_1 recovery status is unknown
WARNING:  datanode slave dn_slave_1 recovery status is unknown
WARNING:  datanode slave dn_slave_2 recovery status is unknown
WARNING:  datanode slave dn_slave_3 recovery status is unknown
```

例如： gtm_slave_1 这个从节点状态为 not running

在 `/data01/antdb/data/gtm_slave_1/pg_log` 目录下查看最新日志，发现从节点 `max_worker_processes` 参数值小于主节点对应参数值，导致启动失败。

```shell
[antdb@oss-irms-181 gtm_slave_1]$ cat pg_log/postgresql-2021-12-01_104526.csv
2021-12-01 10:45:26.926 CST,,,28219,,61a6e1c6.6e3b,3,,2021-12-01 10:45:26 CST,,0,FATAL,22023,"hot standby is not possible because max_worker_processes = 32 is a lower setting than on the master server (its value was 108)",,,,,,,,,""
2021-12-01 10:45:26.929 CST,,,28215,,61a6e1c6.6e37,2,,2021-12-01 10:45:26 CST,,0,LOG,00000,"startup process (PID 28219) exited with exit code 1",,,,,,,,,""
2021-12-01 10:45:26.929 CST,,,28215,,61a6e1c6.6e37,3,,2021-12-01 10:45:26 CST,,0,LOG,00000,"aborting startup due to startup process failure",,,,,,,,,""
2021-12-01 10:45:26.948 CST,,,28215,,61a6e1c6.6e37,4,,2021-12-01 10:45:26 CST,,0,LOG,00000,"database system is shut down",,,,,,,,,""
```

但是服务器上的主节点对应参数值也是 32，108 这个数值是注释掉的

```shell
[antdb@oss-irms-180 gtm_master]$ cat postgresql.conf | grep max_worker_processes
#max_worker_processes = 108		# (change requires restart)
#max_parallel_workers = 8		# maximum number of max_worker_processes that
#max_logical_replication_workers = 4	# taken from max_worker_processes
max_worker_processes = 32
```

使用命令查看主节点 max_worker_processes 值也是 32

```shell
bash-5.0# ansible 10.1.207.180 -m shell -a 'psql -p 16432 -d postgres -c "show param gtm_master max;"'
10.1.207.180 | CHANGED | rc=0 >>
            type            | status |                             message
----------------------------+--------+-----------------------------------------------------------------
 gtmcoord slave gtm_slave_1 | f      | could not connect to server: Connection refused                +
                            |        |         Is the server running on host "127.0.0.1" and accepting+
                            |        |         TCP/IP connections on port 16655?                      +
                            |        |
 gtmcoord master gtm_master | t      | autovacuum_freeze_max_age = 200000000                          +
                            |        | autovacuum_max_workers = 3                                     +
                            |        | autovacuum_multixact_freeze_max_age = 400000000                +
                            |        | bgwriter_lru_maxpages = 100                                    +
                            |        | max_cn_prealloc_xid_size = 0                                   +
                            |        | max_connections = 1000                                         +
                            |        | max_coordinators = 16                                          +
                            |        | max_datanodes = 16                                             +
                            |        | max_files_per_process = 1000                                   +
                            |        | max_function_args = 100                                        +
                            |        | max_identifier_length = 63                                     +
                            |        | max_index_keys = 32                                            +
                            |        | max_locks_per_transaction = 256                                +
                            |        | max_logical_replication_workers = 4                            +
                            |        | max_parallel_maintenance_workers = 2                           +
                            |        | max_parallel_workers = 8                                       +
                            |        | max_parallel_workers_per_gather = 2                            +
                            |        | max_pool_size = 100                                            +
                            |        | max_pred_locks_per_page = 2                                    +
                            |        | max_pred_locks_per_relation = -2                               +
                            |        | max_pred_locks_per_transaction = 64                            +
                            |        | max_prepared_transactions = 1000                               +
                            |        | max_replication_slots = 10                                     +
                            |        | max_stack_depth = 8MB                                          +
                            |        | max_standby_archive_delay = 30s                                +
                            |        | max_standby_streaming_delay = 30s                              +
                            |        | max_sync_workers_per_subscription = 2                          +
                            |        | max_wal_senders = 5                                            +
                            |        | max_wal_size = 1GB                                             +
                            |        | max_worker_processes = 32                                      +
                            |        | reduce_scan_max_buckets = 2048                                 +
                            |        | rep_max_avail_flag = off                                       +
                            |        | rep_max_avail_lsn_lag = 8192                                   +
                            |        | use_aux_max_times = 1
(2 rows)
```

因为默认值是 108，通过 PG 流复制安装从节点参数值只能往大改。所以要不就不设置这个参数，要不就修改为大于等于 108。
