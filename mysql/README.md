# Automate Install MySQL InnoDB Cluster with Ansible Playbook | [中文](README_ZH.md)

## Overview

MySQL InnoDB Cluster has introduced by the MySQL team for the High Availability ( HA ) purpose . It provides a complete high availability solution for MySQL.

I going to show the three-node InnoDB cluster configuration in the Ansible script

MySQL InnoDB Cluster is the Combination of,

* MySQL shell
* Group Replication ( GR )
* MySQL Router

## Planning Your Installation

Planning for server

| IP | SSH PORT | SSH USER | SSH PASSWORD | ROOT PASSWORD | OS |
| ---- | ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | 22022 | mysql | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.181 | 22022 | mysql | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.182 | 22022 | mysql | 123456 | root123 | CentOS Linux release 7.9.2009 |

**TIPS：** reference [Create User in Batch Automation](https://github.com/coolbeevip/ansible-playbook#create-user--group)

Planning for MySQL nodes

| IP地址 | MySQL Server | MySQL Router | MySQL Shell |
| ---- | ---- | ---- | ---- |
| 10.1.207.180 | Primary | Primary | Primary |
| 10.1.207.181 | Secondary | Primary | Primary |
| 10.1.207.182 | Secondary | Primary | Primary |

Planning for installation directory

| PATH | DESCRIPTION |
| ---- | ---- |
| /etc/hosts | Map Domain Address with IP Address |
| /opt/mysql | Installation path of MySQL server、MySQL Shell、MySQL router |
| ~/.bash_profile | Configure MySQL environment and PATH |
| ~/.my.cnf | MySQL database server configuration file |
| ~/mysql_uninstall.sh | MySQL InnoDB Uninstall script|
| ~/mysql.server |  MySQL server Startup script |
| ~/mysql_router_start.sh | MySQL router startup script symbolic link|
| ~/mysql_router_stop.sh |  MySQL router stop script symbolic link |
| /data01/mysql/run |  MySQL server PID file |
| /data01/mysql/logs | MySQL server log file path |
| /data01/mysql/data | MySQL server data file path  |
| /data01/mysql/dump | MySQL server only allows import and export operations in this directory|
| /data01/mysql/script | Temporary scripts file storage directory during installation |
| /data01/mysql/binlog | MySQL server binary log file storage directory |
| /data01/mysql/relaylog | MySQL server relay log files torage directory |
| /data01/mysql/router/mycluster | Configuration data logs file of MySQL router |

**TIPS:** The existing /etc/my.cnf /etc/mysql/my.cnf file will be renamed to /etc/my.cnf.deleted /etc/mysql/my.cnf.deleted

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

Download MySQL tar ball from the https://dev.mysql.com/downloads/ web site to `~/my-docker-volume/ansible-playbook/packages`

```shell
wget -P ~/my-docker-volume/ansible-playbook/packages http://ftp.ntu.edu.tw/MySQL/Downloads/MySQL-8.0/mysql-8.0.27-linux-glibc2.12-x86_64.tar.xz --no-check-certificate
wget -P ~/my-docker-volume/ansible-playbook/packages http://ftp.ntu.edu.tw/MySQL/Downloads/MySQL-Shell/mysql-shell-8.0.27-linux-glibc2.12-x86-64bit.tar.gz --no-check-certificate
wget -P ~/my-docker-volume/ansible-playbook/packages http://ftp.ntu.edu.tw/MySQL/Downloads/MySQL-Router/mysql-router-8.0.27-linux-glibc2.12-x86_64.tar.xz --no-check-certificate
```

## Configuration

> You can edit the following configuration files to modify the default parameters

#### main-mysql.yml

Linux Mapping of IP addresses to hostname /etc/hosts

```shell
- hosts: 10.1.207.180
  user: mysql

- hosts: 10.1.207.181
  user: mysql

- hosts: 10.1.207.182
  user: mysql
```

#### main-cluster.yml

Configure IP address of MySQL Cluster Master node, and the system user name

```shell
- hosts: 10.1.207.180
  user: mysql
```

#### main-router.yml

Install MySQL Router Master nodes, and the system user name

```shell
- hosts: 10.1.207.180
  user: mysql

- hosts: 10.1.207.181
  user: mysql

- hosts: 10.1.207.182
  user: mysql
```

### vars_mysql.yml

Linux Mapping of IP addresses to hostname /etc/hosts

```shell
# Linux Mapping of IP addresses to hostname /etc/hosts
hosts:
  10.1.207.180: oss-irms-180
  10.1.207.181: oss-irms-181
  10.1.207.182: oss-irms-182
```

Linux Limits

```shell
# Linux limits
limits_hard_nproc: '65535'
limits_soft_nproc: '65535'
limits_hard_nofile: '65535'
limits_soft_nofile: '65535'
```

Linux user and group

```shell
# Linux user & group
mysql_user: "mysql"
mysql_group: "mysql"
```

MySQL Install binary package and unzip directory

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

Install directory. The path that contains **\_fast\_** in the path variable is recommended to be defined on the SSD disk

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

MySQL root initialization password

```shell
# MySQL administrator user initialization password, recommended only contain letters, numbers, and underscores
mysql_user_root_password: "CoolbeevipWowo"
```

MySQL server configuration my.cnf

```shell
# MySQL server configuration
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

MySQL cluster name

```shell
# MySQL Cluster
cluster_name: mycluster
```

MySQL router configuration

```shell
# MySQL Router configuration
mysql_router_base_port: 36446
mysql_router_max_connections: 3000
mysql_router_max_connect_errors: 300
```

#### my.cnf.j2

For more `my.cnf` configuration, you can modify the `mysql/conf/my.cnf.j2` template file

## Installation

Start the ansible container tool to connect to the target server, And mount directory `~/my-docker-volume/ansible-playbook` in the container.

**TIPS:** ANSIBLE_SSH_USERS，ANSIBLE_SSH_PASSS is linux user mysql and password

**TIPS:** ANSIBLE_SU_PASSS is user root password

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

#### Install MySQL InnoDB Cluster

This script will automate the following operations

* Configure operating system parameters on all server
* Upload the MySQL packages to all server
* Configure MySQL environment variables on all server
* Initialize MySQL database，Configure MySQL root password and start MySQL service on all server
* Configure MySQL Group Replication on MySQL primary server
* Install and start MySQL router on all server

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/mysql/main-mysql.yml /ansible-playbook/mysql/main-cluster.yml /ansible-playbook/mysql/main-router.yml
```

**TIPS:** Because the MySQL installation package will be uploaded to all servers (about 1.3GB) when the script is executed for the first time, so take longer to execute. The first installation on my local machine takes < 25 minutes(upload package taske about 5 minutes, others take about 20 minutes.)

**TIPS:** This script is only used for initial installation. Repeated execution of this command may receive a prompt of `MySQL has been installed, please uninstall and then reinstall`. At this time, you need to use `ansible all -m shell -a '~/mysql_uninstall.sh'` uninstalls.

If you see the following message, the installation is completed

```shell
TASK [Install Succeed] ********************************************************************************************************************************************************************************************************
ok: [10.1.207.180] => {
    "msg": "Install Succeed!"
}
```

#### Verify MySQL InnoDB Cluster

Verify MySQL node status

```shell
bash-5.0# ansible all -m shell -a '~/mysql.server status'
10.1.207.181 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (25729)

10.1.207.180 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (9462)

10.1.207.182 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (28934)
```

Check MySQL cluster status

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

Verify the connection between MySQL instances

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

View MySQL Router process

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

Verify through MySQL Router RW port **36447** to connect to the master node of the database to view MGR group information

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

Verify through MySQL Router RO port **36447** to connect to the master node of the database to view MGR group information

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

#### Remove setup files after complete installation

Delete the temporary script file generated during the installation process (**because it contains sensitive information such as the root password**)

```shell
bash-5.0# ansible all -m shell -a 'rm -rf /data01/mysql/script/*'
```

(**Optional**)Delete the installation binary package file (about 1.3GB of disk space can be freed after deleting files)

```shell
bash-5.0# ansible all -m shell -a 'rm /opt/*.tar.*'
```

**Congratulations！You have completed the installation of MySQL InnoDB cluster**

## Common Maintenance Commands

View the hostname of each server

```shell
bash-5.0# ansible all -m shell -a 'hostname'
10.1.207.181 | CHANGED | rc=0 >>
oss-irms-181

10.1.207.180 | CHANGED | rc=0 >>
oss-irms-180

10.1.207.182 | CHANGED | rc=0 >>
oss-irms-182
```

Check /etc/hosts file.

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

Start MySQL

```shell
bash-5.0# ansible all -m shell -a '~/mysql.server start'
10.1.207.182 | CHANGED | rc=0 >>
Starting MySQL........ SUCCESS!

10.1.207.181 | CHANGED | rc=0 >>
Starting MySQL........ SUCCESS!

10.1.207.180 | CHANGED | rc=0 >>
Starting MySQL........... SUCCESS!
```

Stop MySQL

```shell
bash-5.0# ansible all -m shell -a '~/mysql.server stop'
10.1.207.182 | CHANGED | rc=0 >>
Shutting down MySQL... SUCCESS!

10.1.207.181 | CHANGED | rc=0 >>
Shutting down MySQL... SUCCESS!

10.1.207.180 | CHANGED | rc=0 >>
Shutting down MySQL...... SUCCESS!
```

Check MySQL server status

```shell
bash-5.0# ansible all -m shell -a '~/mysql.server status'
10.1.207.181 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (25729)

10.1.207.180 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (9462)

10.1.207.182 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (28934)
```

Check MySQL cluster status

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

Start MySQL Router

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

Stop MySQL Router

```shell
bash-5.0# ansible all -m shell -a '~/mysql_router_stop.sh'
10.1.207.180 | CHANGED | rc=0 >>


10.1.207.181 | CHANGED | rc=0 >>


10.1.207.182 | CHANGED | rc=0 >>
```

Check MySQL Router process

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

Use MySQL Router to connect to the database and view database variables.

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

#### TASK[initialize mysql] failure when execute main-mysql.yml

Q: Error log `/data01/mysql/logs/mysqld.err` output `Resource temporarily unavailable`

A: Please check free memory

#### Uninstall MySQL InnoDB Cluster

Q: How to Uninstall MySQL InnoDB Cluster

A: `~/mysql_uninstall.sh` script will **kill -9 MySQL server and MySQL router, delete program files and data files**

```shell
bash-5.0# ansible all -m shell -a '~/mysql_uninstall.sh'
10.1.207.180 | CHANGED | rc=0 >>


10.1.207.182 | CHANGED | rc=0 >>


10.1.207.181 | CHANGED | rc=0 >>
```
