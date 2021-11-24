# Ansible Playbook Install MySQL InnoDB Cluster | [中文](README_ZH.md)

## Overview

MySQL InnoDB Cluster has introduced by the MySQL team for the High Availability ( HA ) purpose . It provides a complete high availability solution for MySQL.

I going to show the three-node InnoDB cluster configuration in the Ansible script

MySQL InnoDB Cluster is the Combination of,

* MySQL shell
* Group Replication ( GR )
* MySQL Router

## Planning Your installation

Planning for server

| IP | SSH PORT | SSH USER | SSH PASSWORD | ROOT PASSWORD |
| ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | 22022 | mysql | 123456 | root123 |
| 10.1.207.181 | 22022 | mysql | 123456 | root123 |
| 10.1.207.182 | 22022 | mysql | 123456 | root123 |

Planning for MySQL nodes

| IP | Node |
| ---- | ---- |
| 10.1.207.180 | MySQL Master Node, MySQL Router |
| 10.1.207.181 | MySQL Slave Node, MySQL Router |
| 10.1.207.182 | MySQL Slave Node, MySQL Router |

Planning for installation directory

> The following path is the default path. Before installation, you can edit the variables in `var_mysql.yml` to change to the path you want. The path that contains **\_fast\_** in the path variable is recommended to be defined on the SSD disk

| PATH | DESCRIPTION |
| ---- | ---- |
| /opt/mysql | installation path of MySQL server、MySQL Shell、MySQL router |
| /etc/my.cnf | MySQL database server configuration file |
| /data01/mysql/run |  MySQL server PID file |
| /data01/mysql/logs | MySQL server log file path |
| /data01/mysql/data | MySQL server log file path  |
| /data01/mysql/dump | MySQL server only allows import and export operations in this directory|
| /data01/mysql/script | Temporary scripts file storage directory during installation |
| /data01/mysql/binlog | MySQL server binary log file storage directory |
| /data01/mysql/relaylog | MySQL server relay log files torage directory |
| /data01/mysql/router/mycluster | Configuration file and startup script of MySQL router |
| /etc/init.d/mysql.server |  MySQL server Startup Script |


## Download MySQL Tar & Ansible Playbook scripts

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

## Installation

Start the ansible container tool to connect to the target server, And mount directory `~/my-docker-volume/ansible-playbook` in the container.

**NOTICE:** ANSIBLE_SSH_USERS，ANSIBLE_SSH_PASSS is linux user redis and password

**NOTICE:** ANSIBLE_SU_PASSS is user root password

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

#### Install MySQL server configuration with three nodes

Run Ansible playbook scripts `main-mysql.yml` for install MySQL server

> This script will automatically modify the operating system kernel parameters, upload the installation media to the three target servers, set the MySQL environment variables, initialize the MySQL database, set the MySQL root password, and start the MySQL service

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/mysql/main-mysql.yml
```

**NOTICE:** This script takes a long time to execute for the first time (because about 1.3GB of installation media needs to be uploaded to all target servers). It takes about 6 minutes to execute in my environment after ignoring the upload time

Verify MySQL node status

> You can see that the MySQL instances on the three servers have been started

```shell
bash-5.0# ansible all -m shell -a '/etc/init.d/mysql.server status'
10.1.207.181 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (25729)

10.1.207.180 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (9462)

10.1.207.182 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (28934)
```

Verify the connection between MySQL instances

> Before creating a cluster, we need to confirm that MySQL instances can connect to each other. When you see **The instance 'xxx' is valid to be used in an InnoDB cluster.**, the connection is normal.

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

#### Configure MySQL cluster

> This script will create a cluster on the master node and join two slave nodes to the cluster

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/mysql/main-cluster.yml
```

**NOTICE:** During the execution of this script, the MySQL instance will automatically restart and wait to synchronize the data of the master node and the slave node. It takes about 4 minutes to execute in my environment.

**NOTICE** After the script is executed, you can see the following cluster status information, one PRIMARY node `oss-irms-180` in read-write mode; two SECONDARY nodes with read-only mode `oss-irms-181` and `oss-irms -182`. And all three nodes are in ONLINE state

```shell
TASK [check mysql_cluster_status output] *******************************************************************************************************************************************************************
ok: [10.1.207.180] => {
    "check_result.stdout_lines": [
        "{",
        "    \"clusterName\": \"mycluster\", ",
        "    \"defaultReplicaSet\": {",
        "        \"name\": \"default\", ",
        "        \"primary\": \"oss-irms-180:3336\", ",
        "        \"ssl\": \"REQUIRED\", ",
        "        \"status\": \"OK\", ",
        "        \"statusText\": \"Cluster is ONLINE and can tolerate up to ONE failure.\", ",
        "        \"topology\": {",
        "            \"oss-irms-180:3336\": {",
        "                \"address\": \"oss-irms-180:3336\", ",
        "                \"memberRole\": \"PRIMARY\", ",
        "                \"mode\": \"R/W\", ",
        "                \"readReplicas\": {}, ",
        "                \"replicationLag\": null, ",
        "                \"role\": \"HA\", ",
        "                \"status\": \"ONLINE\", ",
        "                \"version\": \"8.0.27\"",
        "            }, ",
        "            \"oss-irms-181:3336\": {",
        "                \"address\": \"oss-irms-181:3336\", ",
        "                \"memberRole\": \"SECONDARY\", ",
        "                \"mode\": \"R/O\", ",
        "                \"readReplicas\": {}, ",
        "                \"replicationLag\": null, ",
        "                \"role\": \"HA\", ",
        "                \"status\": \"ONLINE\", ",
        "                \"version\": \"8.0.27\"",
        "            }, ",
        "            \"oss-irms-182:3336\": {",
        "                \"address\": \"oss-irms-182:3336\", ",
        "                \"memberRole\": \"SECONDARY\", ",
        "                \"mode\": \"R/O\", ",
        "                \"readReplicas\": {}, ",
        "                \"replicationLag\": null, ",
        "                \"role\": \"HA\", ",
        "                \"status\": \"ONLINE\", ",
        "                \"version\": \"8.0.27\"",
        "            }",
        "        }, ",
        "        \"topologyMode\": \"Single-Primary\"",
        "    }, ",
        "    \"groupInformationSourceMember\": \"oss-irms-180:3336\"",
        "}"
    ]
}
```

**NOTICE:** See [MySQL InnoDB Cluster](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-innodb-cluster.html) for a more complete description of MySQL cluster.

#### Install MySQL router

Pre-Installation

> Need to allow the complete communication between the cluster nodes based on the hostname and IP. Usually, the /etc/hosts file is automatically modified when the previous script is installed.

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

Run Ansible playbook scripts `main-router.yml` for install MySQL router

> This script will automatically generate the mysqlrouter.conf configuration file and start the MySQL Router service

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/mysql/main-router.yml
```

View MySQL Router process

```shell
bash-5.0# ansible all -m shell -a 'ps -ef | grep mysql-router'
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

Verify through MySQL Router RW port **36446** to connect to the master node of the database to view MGR group information

```shell
bash-5.0# ansible all -m shell -a 'source ~/.bash_profile && mysql -h 10.1.207.180 -P 36446 -uroot -pCoolbeevipWowo mysql -e "select * from performance_schema.replication_group_members;"'
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

```
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

**At this point, you have completed the installation of the MySQL InnoDB cluster**

#### Remove setup files after complete installation

Delete the temporary script file generated during the installation process (**because it contains sensitive information such as the root password**)

```shell
bash-5.0# ansible all -m shell -a 'rm /data01/mysql/script/*'
```

(**Optional**)Delete the installation package file (about 1.3GB of disk space can be freed after deleting files)

```shell
bash-5.0# ansible all -m shell -a 'rm /opt/*.tar.*'
```

## Common operation and maintenance commands

Start MySQL

```shell
bash-5.0# ansible all -m shell -a '/etc/init.d/mysql.server start'
10.1.207.182 | CHANGED | rc=0 >>
Starting MySQL........ SUCCESS!

10.1.207.181 | CHANGED | rc=0 >>
Starting MySQL........ SUCCESS!

10.1.207.180 | CHANGED | rc=0 >>
Starting MySQL........... SUCCESS!
```

Stop MySQL

```shell
bash-5.0# ansible all -m shell -a '/etc/init.d/mysql.server stop'
10.1.207.182 | CHANGED | rc=0 >>
Shutting down MySQL... SUCCESS!

10.1.207.181 | CHANGED | rc=0 >>
Shutting down MySQL... SUCCESS!

10.1.207.180 | CHANGED | rc=0 >>
Shutting down MySQL...... SUCCESS!
```

Check MySQL server status

```shell
bash-5.0# ansible all -m shell -a '/etc/init.d/mysql.server status'
10.1.207.181 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (25729)

10.1.207.180 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (9462)

10.1.207.182 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (28934)
```

Check MySQL cluster status

```shell
bash-5.0# ansible 10.1.207.180 -m shell -a 'source ~/.bash_profile && mysqlsh --password="123!@#" root@10.1.207.180:3336 -- cluster status'
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
bash-5.0# ansible all -m shell -a '/data01/mysql/router/mycluster/start.sh'
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
bash-5.0# ansible all -m shell -a '/data01/mysql/router/mycluster/stop.sh'
10.1.207.180 | CHANGED | rc=0 >>


10.1.207.181 | CHANGED | rc=0 >>


10.1.207.182 | CHANGED | rc=0 >>
```

Check MySQL Router process

```shell
bash-5.0# ansible all -m shell -a 'ps -ef | grep mysql-router'
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
