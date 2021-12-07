# Ansible Playbook 自动化安装 MySQL InnoDB 集群 | [English](README.md)

## 集群概述

MySQL InnoDB Cluster 是 MySQL 团队为了高可用性 (HA) 目的而引入的。它为 MySQL 提供了完整的高可用解决方案。

我将Ansible 脚本中展示三个节点的 InnoDB 集群配置。

MySQL InnoDB 集群有以下服务组成

* MySQL shell
* Group Replication ( GR )
* MySQL Router

## 安装计划

服务器规划

| IP地址 | SSH 端口 | SSH 用户名 | SSH 密码 | ROOT 密码 | OS |
| ---- | ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | 22022 | mysql | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.181 | 22022 | mysql | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.182 | 22022 | mysql | 123456 | root123 | CentOS Linux release 7.9.2009 |

**提示：** 可以参考[批量自动化创建用户](https://github.com/coolbeevip/ansible-playbook/blob/main/README_ZH.md#%E5%88%9B%E5%BB%BA%E7%94%A8%E6%88%B7%E5%92%8C%E7%BB%84)

集群节点规划

| IP地址 | MySQL Server | MySQL Router | MySQL Shell |
| ---- | ---- | ---- | ---- |
| 10.1.207.180 | Primary | ✓ | ✓ |
| 10.1.207.181 | Secondary | ✓ | ✓ |
| 10.1.207.182 | Secondary | ✓ | ✓ |

节点安装路径

| 路径 | 描述 |
| ---- | ---- |
| /etc/hosts | 所有节点 IP 地址和主机名映射 |
| /opt/mysql | MySQL server、MySQL Shell、MySQL router 程序安装路径 |
| ~/.bash_profile | 设置 MySQL 环境变量 |
| ~/.my.cnf | MySQL 配置文件 |
| ~/mysql_uninstall.sh | MySQL InnoDB 集群卸载脚本 |
| ~/mysql.server | MySQL server 的启停脚本 |
| ~/mysql_router_start.sh | MySQL router 启动脚本软连接 |
| ~/mysql_router_stop.sh |  MySQL router 停止脚本软连接 |
| /data01/mysql/run | MySQL server pid 文件路径 |
| /data01/mysql/logs | MySQL server 日志文件路径 |
| /data01/mysql/data | MySQL server 数据文件路径 |
| /data01/mysql/dump | MySQL server 只允许在这个目录下进行导入导出操作 |
| /data01/mysql/script | 安装过程中临时脚本存放的目录 |
| /data01/mysql/binlog | MySQL server bin-log 文件存储路径 |
| /data01/mysql/relaylog | MySQL server relay-log 文件存储路径 |
| /data01/mysql/router/mycluster | MySQL router 的配置文件、数据文件、日志文件 |

**提示:** 系统自带 /etc/my.cnf /etc/mysql/my.cnf 文件将被重命名为 /etc/my.cnf.deleted /etc/mysql/my.cnf.deleted

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

下载 MySQL 安装包到 `~/my-docker-volume/ansible-playbook/packages` 目录

```shell
wget -P ~/my-docker-volume/ansible-playbook/packages http://ftp.ntu.edu.tw/MySQL/Downloads/MySQL-8.0/mysql-8.0.27-linux-glibc2.12-x86_64.tar.xz --no-check-certificate
wget -P ~/my-docker-volume/ansible-playbook/packages http://ftp.ntu.edu.tw/MySQL/Downloads/MySQL-Shell/mysql-shell-8.0.27-linux-glibc2.12-x86-64bit.tar.gz --no-check-certificate
wget -P ~/my-docker-volume/ansible-playbook/packages http://ftp.ntu.edu.tw/MySQL/Downloads/MySQL-Router/mysql-router-8.0.27-linux-glibc2.12-x86_64.tar.xz --no-check-certificate
```
## 配置安装脚本

> 您可以编辑以下配置文件，修改默认参数

#### main-mysql.yml

安装 MySQL Server 的服务器 IP 地址，以及系统用户名

```shell
- hosts: 10.1.207.180
  user: mysql

- hosts: 10.1.207.181
  user: mysql

- hosts: 10.1.207.182
  user: mysql
```

#### main-cluster.yml

配置 MySQL 集群主节点的服务器 IP 地址，以及系统用户名

```shell
- hosts: 10.1.207.180
  user: mysql
```

#### main-router.yml

安装 MySQL Router 的服务器 IP 地址，以及系统用户名

```shell
- hosts: 10.1.207.180
  user: mysql

- hosts: 10.1.207.181
  user: mysql

- hosts: 10.1.207.182
  user: mysql
```

#### vars_mysql.yml

主机 IP 地址以及主机名映射关系

```shell
# Linux Mapping of IP addresses to hostname /etc/hosts
hosts:
  10.1.207.180: oss-irms-180
  10.1.207.181: oss-irms-181
  10.1.207.182: oss-irms-182
```

操作系统 Limits

```shell
# Linux limits
limits_hard_nproc: '65535'
limits_soft_nproc: '65535'
limits_hard_nofile: '65535'
limits_soft_nofile: '65535'
```

安装用的用户名、用户组

```shell
# Linux user & group
mysql_user: "mysql"
mysql_group: "mysql"
```

安装介质名称以及解压后的目录名

```shell
# MySQL server package
mysql_tar: "mysql-8.0.27-linux-glibc2.12-x86_64.tar.xz"
mysql_tar_unzip_dir: "mysql-8.0.27-linux-glibc2.12-x86_64"

# MySQL shell package
mysql_shell_tar: "mysql-shell-8.0.27-linux-glibc2.12-x86-64bit.tar.gz"
mysql_shell_tar_unzip_dir: "mysql-shell-8.0.27-linux-glibc2.12-x86-64bit"

# MySQL router package
mysql_router_tar: "mysql-router-8.0.27-linux-glibc2.12-x86_64.tar.xz"
mysql_router_tar_unzip_dir: "mysql-router-8.0.27-linux-glibc2.12-x86_64"
```

安装路径，其中路径变量中包含 **\_fast\_** 字样的路径建议你定义在 SSD 磁盘上

```shell
# MySQL InnoDB Cluster install directory
mysql_home_dir: "/opt/mysql"
mysql_run_dir: "/data01/mysql/run"
mysql_log_dir: "/data01/mysql/logs"
mysql_data_dir: "/data01/mysql/data"
mysql_dump_dir: "/data01/mysql/dump"
mysql_script_dir: "/data01/mysql/script"
## SSD disk is recommended for fast directory
mysql_fast_data_dir: "/data01/mysql/data"
mysql_fast_binlog_dir: "/data01/mysql/binlog"
mysql_fast_relaylog_dir: "/data01/mysql/relaylog"
## MySQL router work directory
mysql_router_dir: "/data01/mysql/router"
```

MySQL root 初始化密码

```shell
# MySQL administrator user initialization password, recommended only contain letters, numbers, and underscores
mysql_user_root_password: "CoolbeevipWowo"
```

MySQL server 配置

```shell
# MySQL server configuration my.conf
mysqld_port: 3336
mysqld_max_connections: 1000
mysqld_max_connect_errors: 300
mysqld_default_time_zone: "+08:00"
mysqld_mysqlx_port: 33360
mysqld_group_replication_port: 33361
mysqld_character_set_server: utf8mb4
mysqld_collation_server: utf8mb4_general_ci
mysqld_innodb_buffer_pool_size: 10G
client_default_character_set: utf8mb4
```

MySQL cluster 配置

```shell
# MySQL Cluster
cluster_name: mycluster
```

MySQL router 配置

```shell
# MySQL Router configuration
mysql_router_base_port: 36446
mysql_router_max_connections: 3000
mysql_router_max_connect_errors: 300
```

#### my.cnf.j2

更多的 my.cnf 配置你可以直接修改 mysql/conf/my.cnf.j2 模版文件

## 开始安装

启动 ansible 容器工具连接目标服务器，并将 `~/my-docker-volume/ansible-playbook` 目录挂载到容器中。

**提示：** ANSIBLE_SSH_USERS，ANSIBLE_SSH_PASSS 配置成您之前在目标服务器上创建的用户名 `mysql` 和密码 `123456`

**提示：** ANSIBLE_SU_PASSS 为 root 用户的密码

```shell
docker run --name ansible --rm -it \
  -e ANSIBLE_SSH_HOSTS=10.1.207.180,10.1.207.181,10.1.207.182 \
  -e ANSIBLE_SSH_PORTS=22022,22022,22022 \
  -e ANSIBLE_SSH_USERS=mysql,mysql,mysql \
  -e ANSIBLE_SSH_PASSS=123456,123456,123456 \
  -e ANSIBLE_SU_PASSS=root123,root123,root123 \
  -v /Users/zhanglei/mydocker/volume/ansible-playbook:/ansible-playbook \
  coolbeevip/ansible:2.8.11-alpine \
  /bin/bash  
```

#### 安装 MySQL 集群

这个脚本将自动化完成如下操作：

* 配置操作系统参数
* 上传安装介质到每个服务器
* 配置每个服务器上的 MySQL 环境变量
* 初始化每个服务器的 MySQL 数据库，设置 root 密码并启动
* 子主节点服务器上配置 MySQL 主从复制关系
* 在每个服务器上安装 MySQL router 并启动

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/mysql/main-mysql.yml /ansible-playbook/mysql/main-cluster.yml /ansible-playbook/mysql/main-router.yml
```

**提示:** 因为第一次执行脚本时，会上传MySQL 安装包到所有服务器（约1.3GB），所以执行时间较长（取决于你的客户端和服务器之间的网络速度）。 你也可以在执行以上脚本前手动将安装包上传到服务器的安装路径 `/opt/mysql` 下。在我本地环境首次安装大概耗时 25 分钟（上传安装包大概 5 分钟，安装集群大概 20 分钟）

**提示:** 此脚本只适合初始化安装，重复执行此命令可能会收到 `MySQL has been installed, please uninstall and then reinstall` 提示，此时需要先要使用 `ansible all -m shell -a '~/mysql_uninstall.sh'` 命令卸载之前的安装。

如果你看到如下信息，说明安装完成

```shell
TASK [Install Succeed] ********************************************************************************************************************************************************************************************************
ok: [10.1.207.180] => {
    "msg": "Install Succeed!"
}
```

#### 验证 MySQL 集群

检查 MySQL 状态

```shell
bash-5.0# ansible all -m shell -a '~/mysql.server status'
10.1.207.181 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (25729)

10.1.207.180 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (9462)

10.1.207.182 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (28934)
```

检查 MySQL 集群状态

```shell
bash-5.0# ansible 10.1.207.180 -m shell -a 'source ~/.bash_profile && mysqlsh --password="CoolbeevipWowo" root@10.1.207.180:3336 -- cluster status'
10.1.207.180 | CHANGED | rc=0 >>
{
    "clusterName": "mycluster",
    "defaultReplicaSet": {
        "name": "default",
        "primary": "oss-irms-180:3336",
        "ssl": "REQUIRED",
        "status": "OK",
        "statusText": "Cluster is ONLINE and can tolerate up to ONE failure.",
        "topology": {
            "oss-irms-180:3336": {
                "address": "oss-irms-180:3336",
                "memberRole": "PRIMARY",
                "mode": "R/W",
                "readReplicas": {},
                "replicationLag": null,
                "role": "HA",
                "status": "ONLINE",
                "version": "8.0.27"
            },
            "oss-irms-181:3336": {
                "address": "oss-irms-181:3336",
                "memberRole": "SECONDARY",
                "mode": "R/O",
                "readReplicas": {},
                "replicationLag": null,
                "role": "HA",
                "status": "ONLINE",
                "version": "8.0.27"
            },
            "oss-irms-182:3336": {
                "address": "oss-irms-182:3336",
                "memberRole": "SECONDARY",
                "mode": "R/O",
                "readReplicas": {},
                "replicationLag": null,
                "role": "HA",
                "status": "ONLINE",
                "version": "8.0.27"
            }
        },
        "topologyMode": "Single-Primary"
    },
    "groupInformationSourceMember": "oss-irms-180:3336"
}
```

校验 MySQL 节点之间是否可以正常连接

```shell
bash-5.0# ansible 10.1.207.180 -m shell -a 'source ~/.bash_profile && mysqlsh --no-password < /data01/mysql/script/mysql_members_validate.sql'
10.1.207.180 | CHANGED | rc=0 >>

Checking whether existing tables comply with Group Replication requirements...
Checking instance configuration...
Checking whether existing tables comply with Group Replication requirements...
Checking instance configuration...

Checking whether existing tables comply with Group Replication requirements...
Checking instance configuration...Validating local MySQL instance listening at port 3336 for use in an InnoDB cluster...
This instance reports its own address as oss-irms-180:3336
Clients and other cluster members will communicate with it through this address by default. If this is not correct, the report_host MySQL system variable should be changed.
No incompatible tables detected
Instance configuration is compatible with InnoDB cluster
The instance 'oss-irms-180:3336' is valid to be used in an InnoDB cluster.
Validating MySQL instance at oss-irms-181:3336 for use in an InnoDB cluster...
This instance reports its own address as oss-irms-181:3336
Clients and other cluster members will communicate with it through this address by default. If this is not correct, the report_host MySQL system variable should be changed.
No incompatible tables detected
Instance configuration is compatible with InnoDB cluster
The instance 'oss-irms-181:3336' is valid to be used in an InnoDB cluster.
Validating MySQL instance at oss-irms-182:3336 for use in an InnoDB cluster...
This instance reports its own address as oss-irms-182:3336
Clients and other cluster members will communicate with it through this address by default. If this is not correct, the report_host MySQL system variable should be changed.
No incompatible tables detected
Instance configuration is compatible with InnoDB cluster
The instance 'oss-irms-182:3336' is valid to be used in an InnoDB cluster.
```

查看 MySQL Router 进程

```shell
bash-5.0# ansible all -m shell -a 'ps -ef | grep [m]ysql-router'
10.1.207.180 | CHANGED | rc=0 >>
mysql    30445     1  1 17:05 ?        00:00:02 /opt/mysql/mysql-router-8.0.27-linux-glibc2.12-x86_64/bin/mysqlrouter -c /data01/mysql/router/mycluster/mysqlrouter.conf
mysql    30993 30991  0 17:07 pts/1    00:00:00 /bin/sh -c ps -ef | grep mysql-router
mysql    31000 30993  0 17:07 pts/1    00:00:00 grep mysql-router

10.1.207.182 | CHANGED | rc=0 >>
mysql    26006     1  1 17:01 ?        00:00:02 /opt/mysql/mysql-router-8.0.27-linux-glibc2.12-x86_64/bin/mysqlrouter -c /data01/mysql/router/mycluster/mysqlrouter.conf
mysql    26376 26375  0 17:03 pts/2    00:00:00 /bin/sh -c ps -ef | grep mysql-router
mysql    26378 26376  0 17:03 pts/2    00:00:00 grep mysql-router

10.1.207.181 | CHANGED | rc=0 >>
mysql    16000     1  1 17:01 ?        00:00:02 /opt/mysql/mysql-router-8.0.27-linux-glibc2.12-x86_64/bin/mysqlrouter -c /data01/mysql/router/mycluster/mysqlrouter.conf
mysql    16463 16462  0 17:03 pts/3    00:00:00 /bin/sh -c ps -ef | grep mysql-router
mysql    16465 16463  0 17:03 pts/3    00:00:00 grep mysql-router
```

测试通过 MySQL Router RW 端口 **36447** 连接数据库主节点执行查看 MGR 组信息

```shell
bash-5.0# ansible all -m shell -a 'source ~/.bash_profile && mysql -h 10.1.207.180 -P 36447 -uroot -pCoolbeevipWowo mysql -e "select * from performance_schema.replication_group_members;"'
10.1.207.180 | CHANGED | rc=0 >>
CHANNEL_NAME	MEMBER_ID	MEMBER_HOST	MEMBER_PORT	MEMBER_STATE	MEMBER_ROLE	MEMBER_VERSION	MEMBER_COMMUNICATION_STACK
group_replication_applier	5e11bf00-4cf5-11ec-8798-5254005e1dd1	oss-irms-181	3336	ONLINE	SECONDARY	8.0.27	XCom
group_replication_applier	93a9227d-4cf5-11ec-9851-5254001a7e4c	oss-irms-182	3336	ONLINE	SECONDARY	8.0.27	XCom
group_replication_applier	9aed150e-4cf5-11ec-8819-525400506ca8	oss-irms-180	3336	ONLINE	PRIMARY	8.0.27	XCommysql: [Warning] Using a password on the command line interface can be insecure.

10.1.207.181 | CHANGED | rc=0 >>
CHANNEL_NAME	MEMBER_ID	MEMBER_HOST	MEMBER_PORT	MEMBER_STATE	MEMBER_ROLE	MEMBER_VERSION	MEMBER_COMMUNICATION_STACK
group_replication_applier	5e11bf00-4cf5-11ec-8798-5254005e1dd1	oss-irms-181	3336	ONLINE	SECONDARY	8.0.27	XCom
group_replication_applier	93a9227d-4cf5-11ec-9851-5254001a7e4c	oss-irms-182	3336	ONLINE	SECONDARY	8.0.27	XCom
group_replication_applier	9aed150e-4cf5-11ec-8819-525400506ca8	oss-irms-180	3336	ONLINE	PRIMARY	8.0.27	XCommysql: [Warning] Using a password on the command line interface can be insecure.

10.1.207.182 | CHANGED | rc=0 >>
CHANNEL_NAME	MEMBER_ID	MEMBER_HOST	MEMBER_PORT	MEMBER_STATE	MEMBER_ROLE	MEMBER_VERSION	MEMBER_COMMUNICATION_STACK
group_replication_applier	5e11bf00-4cf5-11ec-8798-5254005e1dd1	oss-irms-181	3336	ONLINE	SECONDARY	8.0.27	XCom
group_replication_applier	93a9227d-4cf5-11ec-9851-5254001a7e4c	oss-irms-182	3336	ONLINE	SECONDARY	8.0.27	XCom
group_replication_applier	9aed150e-4cf5-11ec-8819-525400506ca8	oss-irms-180	3336	ONLINE	PRIMARY	8.0.27	XCommysql: [Warning] Using a password on the command line interface can be insecure.
```

测试通过 MySQL Router RO 端口 **36447** 连接数据库从节点执行查看 MGR 组信息

```shell
bash-5.0# ansible all -m shell -a 'source ~/.bash_profile && mysql -h 10.1.207.180 -P 36447 -uroot -pCoolbeevipWowo mysql -e "select * from performance_schema.replication_group_members;"'
10.1.207.181 | CHANGED | rc=0 >>
CHANNEL_NAME	MEMBER_ID	MEMBER_HOST	MEMBER_PORT	MEMBER_STATE	MEMBER_ROLE	MEMBER_VERSION	MEMBER_COMMUNICATION_STACK
group_replication_applier	5e11bf00-4cf5-11ec-8798-5254005e1dd1	oss-irms-181	3336	ONLINE	SECONDARY	8.0.27	XCom
group_replication_applier	93a9227d-4cf5-11ec-9851-5254001a7e4c	oss-irms-182	3336	ONLINE	SECONDARY	8.0.27	XCom
group_replication_applier	9aed150e-4cf5-11ec-8819-525400506ca8	oss-irms-180	3336	ONLINE	PRIMARY	8.0.27	XCommysql: [Warning] Using a password on the command line interface can be insecure.

10.1.207.180 | CHANGED | rc=0 >>
CHANNEL_NAME	MEMBER_ID	MEMBER_HOST	MEMBER_PORT	MEMBER_STATE	MEMBER_ROLE	MEMBER_VERSION	MEMBER_COMMUNICATION_STACK
group_replication_applier	5e11bf00-4cf5-11ec-8798-5254005e1dd1	oss-irms-181	3336	ONLINE	SECONDARY	8.0.27	XCom
group_replication_applier	93a9227d-4cf5-11ec-9851-5254001a7e4c	oss-irms-182	3336	ONLINE	SECONDARY	8.0.27	XCom
group_replication_applier	9aed150e-4cf5-11ec-8819-525400506ca8	oss-irms-180	3336	ONLINE	PRIMARY	8.0.27	XCommysql: [Warning] Using a password on the command line interface can be insecure.

10.1.207.182 | CHANGED | rc=0 >>
CHANNEL_NAME	MEMBER_ID	MEMBER_HOST	MEMBER_PORT	MEMBER_STATE	MEMBER_ROLE	MEMBER_VERSION	MEMBER_COMMUNICATION_STACK
group_replication_applier	5e11bf00-4cf5-11ec-8798-5254005e1dd1	oss-irms-181	3336	ONLINE	SECONDARY	8.0.27	XCom
group_replication_applier	93a9227d-4cf5-11ec-9851-5254001a7e4c	oss-irms-182	3336	ONLINE	SECONDARY	8.0.27	XCom
group_replication_applier	9aed150e-4cf5-11ec-8819-525400506ca8	oss-irms-180	3336	ONLINE	PRIMARY	8.0.27	XCommysql: [Warning] Using a password on the command line interface can be insecure.
```

#### 安装完成后删除安装文件

删除过程中产生的临时脚本文件（**因为里面包含 root 密码等敏感信息**）

```shell
bash-5.0# ansible all -m shell -a 'rm -rf /data01/mysql/script/*'
```

(**可选**)删除安装包文件（删除后可以释放出约1.3GB的磁盘空间）

```shell
bash-5.0# ansible all -m shell -a 'rm /opt/*.tar.*'
```

**恭喜！您已经完成 MySQL InnoDB 集群的安装**

## 常用运维命令

查看每个机器的主机名

```shell
bash-5.0# ansible all -m shell -a 'hostname'
10.1.207.181 | CHANGED | rc=0 >>
oss-irms-181

10.1.207.180 | CHANGED | rc=0 >>
oss-irms-180

10.1.207.182 | CHANGED | rc=0 >>
oss-irms-182
```

查看每个主机 /etc/hosts 文件中是否配置了每个服务器的主机名和IP地址

```shell
bash-5.0# ansible all -m shell -a 'cat /etc/hosts'
10.1.207.181 | CHANGED | rc=0 >>
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.1.207.180 oss-irms-180
10.1.207.181 oss-irms-181
10.1.207.182 oss-irms-182

10.1.207.182 | CHANGED | rc=0 >>
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.1.207.180 oss-irms-180
10.1.207.181 oss-irms-181
10.1.207.182 oss-irms-182

10.1.207.180 | CHANGED | rc=0 >>
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.1.207.180 oss-irms-180
10.1.207.181 oss-irms-181
10.1.207.182 oss-irms-182
```

启动 MySQL

```shell
bash-5.0# ansible all -m shell -a '~/mysql.server start'
10.1.207.182 | CHANGED | rc=0 >>
Starting MySQL........ SUCCESS!

10.1.207.181 | CHANGED | rc=0 >>
Starting MySQL........ SUCCESS!

10.1.207.180 | CHANGED | rc=0 >>
Starting MySQL........... SUCCESS!
```

停止 MySQL

```shell
bash-5.0# ansible all -m shell -a '~/mysql.server stop'
10.1.207.182 | CHANGED | rc=0 >>
Shutting down MySQL... SUCCESS!

10.1.207.181 | CHANGED | rc=0 >>
Shutting down MySQL... SUCCESS!

10.1.207.180 | CHANGED | rc=0 >>
Shutting down MySQL...... SUCCESS!
```

检查 MySQL 服务状态

```shell
bash-5.0# ansible all -m shell -a '~/mysql.server status'
10.1.207.181 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (25729)

10.1.207.180 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (9462)

10.1.207.182 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (28934)
```

检查 MySQL 集群状态

```shell
bash-5.0# ansible 10.1.207.180 -m shell -a 'source ~/.bash_profile && mysqlsh --password="CoolbeevipWowo" root@10.1.207.180:3336 -- cluster status'
10.1.207.180 | CHANGED | rc=0 >>
{
    "clusterName": "mycluster",
    "defaultReplicaSet": {
        "name": "default",
        "primary": "oss-irms-180:3336",
        "ssl": "REQUIRED",
        "status": "OK",
        "statusText": "Cluster is ONLINE and can tolerate up to ONE failure.",
        "topology": {
            "oss-irms-180:3336": {
                "address": "oss-irms-180:3336",
                "memberRole": "PRIMARY",
                "mode": "R/W",
                "readReplicas": {},
                "replicationLag": null,
                "role": "HA",
                "status": "ONLINE",
                "version": "8.0.27"
            },
            "oss-irms-181:3336": {
                "address": "oss-irms-181:3336",
                "memberRole": "SECONDARY",
                "mode": "R/O",
                "readReplicas": {},
                "replicationLag": null,
                "role": "HA",
                "status": "ONLINE",
                "version": "8.0.27"
            },
            "oss-irms-182:3336": {
                "address": "oss-irms-182:3336",
                "memberRole": "SECONDARY",
                "mode": "R/O",
                "readReplicas": {},
                "replicationLag": null,
                "role": "HA",
                "status": "ONLINE",
                "version": "8.0.27"
            }
        },
        "topologyMode": "Single-Primary"
    },
    "groupInformationSourceMember": "oss-irms-180:3336"
}
```

启动 MySQL Router

```shell
bash-5.0# ansible all -m shell -a '~/mysql_router_start.sh'
10.1.207.181 | CHANGED | rc=0 >>
PID 26307 written to '/data01/mysql/router/mycluster/mysqlrouter.pid'
logging facility initialized, switching logging to loggers specified in configuration

10.1.207.180 | CHANGED | rc=0 >>
PID 8813 written to '/data01/mysql/router/mycluster/mysqlrouter.pid'
logging facility initialized, switching logging to loggers specified in configuration

10.1.207.182 | CHANGED | rc=0 >>
PID 1158 written to '/data01/mysql/router/mycluster/mysqlrouter.pid'
logging facility initialized, switching logging to loggers specified in configuration
```

停止 MySQL Router

```shell
bash-5.0# ansible all -m shell -a '~/mysql_router_stop.sh'
10.1.207.180 | CHANGED | rc=0 >>


10.1.207.181 | CHANGED | rc=0 >>


10.1.207.182 | CHANGED | rc=0 >>
```

检查 MySQL Router 进程

```shell
bash-5.0# ansible all -m shell -a 'ps -ef | grep [m]ysql-router'
10.1.207.180 | CHANGED | rc=0 >>
mysql     8813     1  1 18:00 ?        00:00:01 /opt/mysql/mysql-router-8.0.27-linux-glibc2.12-x86_64/bin/mysqlrouter -c /data01/mysql/router/mycluster/mysqlrouter.conf
mysql     9154  9153  9 18:02 pts/1    00:00:00 /bin/sh -c ps -ef | grep mysql-router
mysql     9158  9154  0 18:02 pts/1    00:00:00 grep mysql-router

10.1.207.181 | CHANGED | rc=0 >>
mysql    26307     1  1 17:57 ?        00:00:01 /opt/mysql/mysql-router-8.0.27-linux-glibc2.12-x86_64/bin/mysqlrouter -c /data01/mysql/router/mycluster/mysqlrouter.conf
mysql    26633 26632  0 17:58 pts/3    00:00:00 /bin/sh -c ps -ef | grep mysql-router
mysql    26635 26633  0 17:58 pts/3    00:00:00 grep mysql-router

10.1.207.182 | CHANGED | rc=0 >>
mysql     1158     1  2 17:56 ?        00:00:01 /opt/mysql/mysql-router-8.0.27-linux-glibc2.12-x86_64/bin/mysqlrouter -c /data01/mysql/router/mycluster/mysqlrouter.conf
mysql     1420  1419  0 17:57 pts/2    00:00:00 /bin/sh -c ps -ef | grep mysql-router
mysql     1423  1420  0 17:57 pts/2    00:00:00 grep mysql-router
```

使用 MySQL Router 连接到数据库并查看数据库变量

```shell
bash-5.0# ansible all -m shell -a 'source ~/.bash_profile && mysql -h 10.1.207.180 -P 36446 -uroot -pCoolbeevipWowo mysql -e "show variables like \"%max_connections%\";"'
10.1.207.181 | CHANGED | rc=0 >>
Variable_name	Value
max_connections	1000
mysqlx_max_connections	100mysql: [Warning] Using a password on the command line interface can be insecure.

10.1.207.180 | CHANGED | rc=0 >>
Variable_name	Value
max_connections	1000
mysqlx_max_connections	100mysql: [Warning] Using a password on the command line interface can be insecure.

10.1.207.182 | CHANGED | rc=0 >>
Variable_name	Value
max_connections	1000
mysqlx_max_connections	100mysql: [Warning] Using a password on the command line interface can be insecure.
```

## Q & A

#### 执行 main-mysql.yml 时 TASK[initialize mysql] 失败

Q: 查看 `/data01/mysql/logs/mysqld.err` 文件中提示 `Resource temporarily unavailable`

A: 请检查服务器内存是否够用

#### 如何彻底删除 MySQL InnoDB 集群

Q: 如何彻底删除 MySQL InnoDB 集群

A: `~/mysql_uninstall.sh` 脚本将 **kill -9 MySQL server 和 MySQL router，删除程序文件和所有数据文件**

```shell
bash-5.0# ansible all -m shell -a '~/mysql_uninstall.sh'
10.1.207.180 | CHANGED | rc=0 >>


10.1.207.182 | CHANGED | rc=0 >>


10.1.207.181 | CHANGED | rc=0 >>
```
