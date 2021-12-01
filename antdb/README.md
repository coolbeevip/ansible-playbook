# Automate Install AntDB Distributed Cluster with Ansible Playbook | [中文](README_ZH.md)

> Automate install 3-node AntDB distributed database cluster in 6 minutes, automatically configure kernel parameters, initialize directories, cluster configuration, and startup the cluster. This script is only for test environment setup. For a production environment, please refer to the official installation document。

AntDB Distributed Cluster is the Combination of,

* MGR - AntDB Manager
* GTM - Global Transaction Manager
* CN - Coordinator
* DN - Data Node

## Planning Your Installation

Planning for server

| IP | SSH PORT | SSH USER | SSH PASSWORD | ROOT PASSWORD | OS |
| ---- | ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | 22022 | antdb | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.181 | 22022 | antdb | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.182 | 22022 | antdb | 123456 | root123 | CentOS Linux release 7.9.2009 |

**TIPS：** reference [Create User in Batch Automation](https://github.com/coolbeevip/ansible-playbook#create-user--group)

Planning for nodes

| IP | MGR | GTM | Coordinator | DataNode |
| ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | Master | Master | | DataNode_Master_1, DataNode_Slave_3 |
| 10.1.207.181 | | Slave_1 | Coordinator_1 | DataNode_Master_2, DataNode_Slave_1 |
| 10.1.207.182 | Slave_1 | | Coordinator_2 | DataNode_Master_3, DataNode_Slave_2 |

Planning for installation directory

| PATH | DESCRIPTION |
| ---- | ---- |
| /opt/antdb | PRM package |
| ~/antdb_uninstall.sh | AntDB cluster uninstall script |
| ~/antdb_config.sql | Database parameter script |
| /data01/antdb/app | PG program directory |
| /data01/antdb/mgr | MGR program directory |
| /data01/antdb/data | Node data and configuration directory|
| /data01/antdb/tools | Tool scripts |
| /data01/antdb/core | core dump directory |

## Download MySQL Tar & Ansible Playbook Scripts

Create a directory for Ansible Playbook scripts

```shell
mkdir -p ~/my-docker-volume/ansible-playbook
```

Download Ansible playbook scripts

```shell
cd ~/my-docker-volume/ansible-playbook
git clone https://github.com/coolbeevip/ansible-playbook.git
```

Download AntDB RPM package `antdb.cluster-5.0.009be78c-centos7.9.rpm` to `~/my-docker-volume/ansible-playbook/packages`

## Configuration

> You can edit the following configuration files to modify the default parameters

#### maim-os-init.yml

Define all the server IP addresses of the AntDB cluster and the system user. This script automatically config the kernel parameters, creates a directory and uploads the RPM package

```yaml
- hosts: 10.1.207.180
  user: antdb

- hosts: 10.1.207.181
  user: antdb

- hosts: 10.1.207.182
  user: antdb
```

#### maim-cluster-install.yml

Define the IP address of the AntDB MGR master/slave node server and the system user. MGR slave node name `mgr_slave_1`

```yaml
# MGR Master Node Initialize
- hosts: 10.1.207.180
  user: antdb

# MGR Slave Node Initialize
- hosts: 10.1.207.182
  user: antdb
  vars:
    mgr_slave_name: "mgr_slave_1"

# Restart all node of AntDB cluster on MGR Master Node
- hosts: 10.1.207.180
  user: antdb
```

#### var_antdb.yml

Linux Limits

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

Linux User & Group

```yaml
antdb_user: 'antdb'
antdb_group: 'antdb'
antdb_password: '123456'
```

AntDB RPM package name

```yaml
antdb_tar: 'antdb.cluster-5.0.009be78c-centos7.9.rpm'
```

AntDB Install Directories

```yaml
antdb_home_dir: '/opt/antdb'
antdb_mgr_dir: '/data01/antdb/mgr'
antdb_data_dir: '/data01/antdb/data'
antdb_tools_dir: '/data01/antdb/tools'
antdb_app_dir: '/data01/antdb/app'
antdb_core_dir: '/data01/antdb/core'
```

AntDB postgresql.conf extended parameters

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

The names of all nodes in the cluster (this name is not a host name, it is a node name for cluster management, name can only contain letters, numbers, and underscores)

```yaml
node_names:
  10.1.207.180:
    name: antdb180
  10.1.207.181:
    name: antdb181
  10.1.207.182:
    name: antdb182
```    

Component ports

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

MGR Configuration

```yaml
mgr_nodes:
  master:
    ip: 10.1.207.180
  slave:
    ips: [10.1.207.182]
```

GTM Configuration

```yaml
gtm_nodes:
  master:
    name: gtm_master
    node: antdb180
  slaves:
    - name: gtm_slave_1
      node: antdb181
```

Coordinator Configuration

```yaml
coordinator_nodes:
  - name: coordinator_1
    node: antdb181
  - name: coordinator_2
    node: antdb182
```

DataNodes Configuration

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

PostgreSQL Server Configuration Parameters

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

For more default configuration, please refer to the `antdb_config.sql.j2` file

## Installation

Start the ansible container tool to connect to the target server, And mount directory `~/my-docker-volume/ansible-playbook` in the container.

**NOTICE:** ANSIBLE_SSH_USERS，ANSIBLE_SSH_PASSS is linux user redis and password

**NOTICE:** ANSIBLE_SU_PASSS is user root password

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

#### Install AntDB Distributed Cluster

Automatically configure system parameters, disable the firewall, create  installation directories, upload RPM package, initialize and startup the cluster

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/antdb/main-os-init.yml /ansible-playbook/antdb/main-cluster-install.yml
```

If you see the following message, the installation is complete

```shell
TASK [Install Succeed] ********************************************************************************************************************************************************************************************************
ok: [10.1.207.180] => {
    "msg": "Install Succeed!"
}
```

**TIPS：** This script takes about 6 minutes to execute in my local

## Verify the cluster

Check the cluster status on MGR master node **10.1.207.180**, you can see that all nodes have been `running`

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

Check the cluster status on MGR slave node **10.1.207.180**, you can see that all nodes have been `running`

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

Test the `Coordinator` connection on the MGR master node **10.1.207.180**

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

## Configure client authentication

When you receive a connection error `[28000] FATAL: no pg_hba.conf entry for host "x.x.x.x", user "xxx", database "xxx"`, You need add client authorization on MGR master node **10.1.207.180**.

Command：

```shell
add hba coordinator all ("host <database> <user> <ip-address> <ip-mark> <auth-method>");
```

For example: add class-c address 10.4.16.0～10.4.16.255

```shell
bash-5.0# ansible 10.1.207.180 -m shell -a 'psql -p 16432 -d postgres -c "add hba coordinator all (\"host all all 10.4.16.0 24 md5\");"'
10.1.207.180 | CHANGED | rc=0 >>
 nodename | values
----------+---------
 *        | success
(1 row)
```

For example: add class-b address 10.4.0.0～10.4.255.255

```shell
bash-5.0# ansible 10.1.207.180 -m shell -a 'psql -p 16432 -d postgres -c "add hba coordinator all (\"host all all 10.4.0.0 16 md5\");"'
10.1.207.180 | CHANGED | rc=0 >>
 nodename | values
----------+---------
 *        | success
(1 row)
```

## Create Database & User

Login to any server in the cluster using SSH, and use the `psql -h 10.1.207.181 -p 15432` command to connect to the any node of the `Coordinator`

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

Connect to the database through the Coordinator node **10.1.207.181**

```shell
[antdb@oss-irms-180 ~]$ psql -h 10.1.207.181 -p 15432 -d testdb -U testdbuser -w testdbpass
psql: warning: extra command-line argument "testdbpass" ignored
psql (5.0.1 based on PG 11.10)
Type "help" for help.

testdb=>
```

Connect to the database through the Coordinator node **10.1.207.182**

```shell
[antdb@oss-irms-180 ~]$ psql -h 10.1.207.182 -p 15432 -d testdb -U testdbuser -w testdbpass
psql: warning: extra command-line argument "testdbpass" ignored
psql (5.0.1 based on PG 11.10)
Type "help" for help.

testdb=>
```

## Common Maintenance Commands

List hosts

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

List nodes

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

List process of AntDB each host

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

Stop all nodes of the cluster

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

Start all nodes of the cluster

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

Show node database parameters

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

#### How to plan node types for multiple servers

> Please refer to official recommendations for production environment

Planning for 3 servers

| IP | MGR | GTM | Coordinator | DataNode |
| ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | Master | Master | | DataNode_Master_1, DataNode_Slave_3 |
| 10.1.207.181 | | Slave_1 | Coordinator_1 | DataNode_Master_2, DataNode_Slave_1 |
| 10.1.207.182 | Slave_1 | | Coordinator_2 | DataNode_Master_3, DataNode_Slave_2 |

Planning for 4 servers

| IP | MGR | GTM | Coordinator | DataNode |
| ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | Master | | | DataNode_Master_1, DataNode_Slave_3 |
| 10.1.207.181 | Slave_1 | | Coordinator_1 | DataNode_Master_2, DataNode_Slave_1 |
| 10.1.207.182 | | Master | | DataNode_Slave_2 |
| 10.1.207.183 | | Slave_1 | Coordinator_2 | DataNode_Master_3 |

Planning for 5 servers

| IP | MGR | GTM | Coordinator | DataNode |
| ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | | Slave_1 | Coordinator_1 | DataNode_Master_1, DataNode_Slave_5 |
| 10.1.207.181 | | Slave_2 | Coordinator_2 | DataNode_Master_2, DataNode_Slave_1 |
| 10.1.207.182 | Slave_1 | Master | Coordinator_3 | DataNode_Master_3, DataNode_Slave_2 |
| 10.1.207.183 | Slave_2 | | Coordinator_4 | DataNode_Master_4, DataNode_Slave_3 |
| 10.1.207.184 | Master | | Coordinator_5 | DataNode_Master_5, DataNode_Slave_4 |

#### How to force uninstall AntDB cluster

This script will kill -9 all AndDB processes and delete programs and data directories

```shell
bash-5.0# ansible all -m shell -a '~/antdb_uninstall.sh'
```

#### How to gracefully uninstall AntDB distributed cluster

Stop all node for AntDB cluster

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

Clean all datas;

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

Stop all Agent of AntDB cluster

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

Check that the AntDB process on all servers has stopped

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

Check that all data files on the server have been deleted

```shell
bash-5.0# ansible all -m shell -a 'ls /data01/antdb'
10.1.207.180 | CHANGED | rc=0 >>


10.1.207.181 | CHANGED | rc=0 >>


10.1.207.182 | CHANGED | rc=0 >>
```

#### The slave node fails to start, the log prompts **hot standby is not possible because xxx = xxx is a lower setting than on the master server**

Use the `monitor all;` to query the startup status of the node

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

For example: gtm_slave_1 The status of this slave node is not running

Check the log file in the directory of `/data01/antdb/data/gtm_slave_1/pg_log` and find that the parameter value of `max_worker_processes` of the slave node is less than the corresponding parameter value of the master node, which causes the startup failure.

```shell
[antdb@oss-irms-181 gtm_slave_1]$ cat pg_log/postgresql-2021-12-01_104526.csv
2021-12-01 10:45:26.926 CST,,,28219,,61a6e1c6.6e3b,3,,2021-12-01 10:45:26 CST,,0,FATAL,22023,"hot standby is not possible because max_worker_processes = 32 is a lower setting than on the master server (its value was 108)",,,,,,,,,""
2021-12-01 10:45:26.929 CST,,,28215,,61a6e1c6.6e37,2,,2021-12-01 10:45:26 CST,,0,LOG,00000,"startup process (PID 28219) exited with exit code 1",,,,,,,,,""
2021-12-01 10:45:26.929 CST,,,28215,,61a6e1c6.6e37,3,,2021-12-01 10:45:26 CST,,0,LOG,00000,"aborting startup due to startup process failure",,,,,,,,,""
2021-12-01 10:45:26.948 CST,,,28215,,61a6e1c6.6e37,4,,2021-12-01 10:45:26 CST,,0,LOG,00000,"database system is shut down",,,,,,,,,""
```

The gtm master node is configured correctly in postgresql.conf

```shell
[antdb@oss-irms-180 gtm_master]$ cat postgresql.conf | grep max_worker_processes
#max_worker_processes = 108		# (change requires restart)
#max_parallel_workers = 8		# maximum number of max_worker_processes that
#max_logical_replication_workers = 4	# taken from max_worker_processes
max_worker_processes = 32
```

It is also correct to use the command `show param gtm_master max;` to view the parameters `max_worker_processes`

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

When installing the slave node through PG streaming replication, the parameter value should be greater than the default value. Delete this parameter or change it to be greater than or equal to 108 to solve the problem.
