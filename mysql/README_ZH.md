# Ansible Playbook 安装 MySQL 集群 | [English](README.md)

## 集群概述


## 目标服务器

服务器规划

| IP地址 | SSH 端口 | SSH 用户名 | SSH 密码 | ROOT 密码 |
| ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | 22022 | mysql | 123456 | root123 |
| 10.1.207.181 | 22022 | mysql | 123456 | root123 |
| 10.1.207.182 | 22022 | mysql | 123456 | root123 |

集群节点规划

| IP地址 | 模块 | 类型 |
| ---- | ---- | ---- |
| 10.1.207.180 | MySQL Node, MySQL Router | master |
| 10.1.207.181 | MySQL Node, MySQL Router | slave |
| 10.1.207.182 | MySQL Node, MySQL Router | slave |

节点安装路径

> 以下路径是默认路径，在安装前可以编辑 `var_mysql.yml` 中的变量改变成你想要的路径，其中路径变量中包含 **\_fast\_** 字样的路径建议你定义在 SSD 磁盘上

| 路径 | 描述 | 备注 |
| ---- | ---- | ---- |
| /opt/mysql | 程序安装路径 |  |
| /etc/my.cnf | MySQL 配置文件路径 |  |
| /data01/mysql/run | MySQL PID 文件存放路径 |  |
| /data01/mysql/logs | MySQL LOG 文件存放路径 |  |
| /data01/mysql/data | MySQL DATA 文件存放路径 |  |
| /data01/mysql/dump | MySQL DUMP 文件存放路径 |  |
| /data01/mysql/script | 安装过程中临时脚本存放路径 | 安装后可删除 |
| /data01/mysql/binlog | MySQL BINLOG 文件存放路径 |  |
| /data01/mysql/relaylog | MySQL RELAYLOG 文件存放路径 |  |
| /etc/init.d/mysql.server | 服务脚本路径 |  |


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

下载 mysql 安装包到 `~/my-docker-volume/ansible-playbook/packages` 目录

```shell
wget -P ~/my-docker-volume/ansible-playbook/packages http://ftp.ntu.edu.tw/MySQL/Downloads/MySQL-8.0/mysql-8.0.27-linux-glibc2.12-x86_64.tar.xz --no-check-certificate
wget -P ~/my-docker-volume/ansible-playbook/packages http://ftp.ntu.edu.tw/MySQL/Downloads/MySQL-Shell/mysql-shell-8.0.27-linux-glibc2.12-x86-64bit.tar.gz --no-check-certificate
wget -P ~/my-docker-volume/ansible-playbook/packages http://ftp.ntu.edu.tw/MySQL/Downloads/MySQL-Router/mysql-router-8.0.27-linux-glibc2.12-x86_64.tar.xz --no-check-certificate
```

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
  -e ANSIBLE_SU_PASSS=xdjr0lxGu,xdjr0lxGu,xdjr0lxGu \
  -v /Users/zhanglei/mydocker/volume/ansible-playbook:/ansible-playbook \
  coolbeevip/ansible:2.8.11-alpine \
  /bin/bash  
```

#### 安装三节点 mysql 实例

执行安装脚本

> 此命令会自动上传安装介质到三个目标服务器，初始化 mysql 数据库，设置 mysql root 密码，启动 mysql 服务

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/mysql/main-mysql.yml
```

检查 mysql 节点状态

> 可以看到三台服务器上的 mysql 服务都已经启动

```shell
bash-5.0# ansible all -m shell -a '/etc/init.d/mysql.server status'
10.1.207.181 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (25729)

10.1.207.180 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (9462)

10.1.207.182 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (28934)
```

校验三节点之间是否可以正常连接

> 我们选择在主节点上执行实例间连接检查，可以看到节点之间是可以相互连接的。你可以看到每个节点都提示 The instance 'xxx' is valid to be used in an InnoDB cluster.

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

#### 初始化三节点集群

> 执行集群初始化脚本，创建集群，设置主节点并增加两外两个从节点

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/mysql/main-cluster.yml
```

在命令执行结束后，你可以看到集群状态信息，你可以看到 `oss-irms-180` 作为 PRIMARY 节点，具有读写模式；`oss-irms-181` 和 `oss-irms-182` 作为 SECONDARY 节点，具有只读模式。并且三个节点都处于 ONLINE 状态

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

**提示：** 更多集群说明请参见 [MySQL InnoDB Cluster](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-innodb-cluster.html)

#### 安装三节点 MySQL Router


#### 清理安装介质

清理安装时生成的脚本文件和安装介质

> /data01/mysql/script 目录下是存储初始化的脚本，安装完毕后可以删除（**因为里面包含 root 密码等敏感信息**）

```shell
bash-5.0# ansible all -m shell -a 'rm /data01/mysql/script/*'
```

> /opt/mysql 目录下的安装介质装完后可以删除（非必须）

```shell
bash-5.0# ansible all -m shell -a 'rm /opt/*.tar.*'
```

## 运维命令

启动 mysql

```shell
bash-5.0# ansible all -m shell -a '/etc/init.d/mysql.server start'
10.1.207.182 | CHANGED | rc=0 >>
Starting MySQL........ SUCCESS!

10.1.207.181 | CHANGED | rc=0 >>
Starting MySQL........ SUCCESS!

10.1.207.180 | CHANGED | rc=0 >>
Starting MySQL........... SUCCESS!
```

停止 mysql

```shell
bash-5.0# ansible all -m shell -a '/etc/init.d/mysql.server stop'
10.1.207.182 | CHANGED | rc=0 >>
Shutting down MySQL... SUCCESS!

10.1.207.181 | CHANGED | rc=0 >>
Shutting down MySQL... SUCCESS!

10.1.207.180 | CHANGED | rc=0 >>
Shutting down MySQL...... SUCCESS!
```

检查 mysql 服务状态

```shell
bash-5.0# ansible all -m shell -a '/etc/init.d/mysql.server status'
10.1.207.181 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (25729)

10.1.207.180 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (9462)

10.1.207.182 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (28934)
```

检查 mysql 集群状态

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

## Q & A

Q: initialize mysql 时失败，查看 `/data01/mysql/logs/mysqld.err` 文件中提示 `Resource temporarily unavailable`
A: 请检查服务器内存是否够用

Q: 如何彻底删除数据文件
A: 先停止 mysql 然后删除所有数据文件

```shell
bash-5.0# ansible all -m shell -a '/etc/init.d/mysql.server stop'
bash-5.0# ansible all -m shell -a 'rm -rf /data01/mysql/data/* /data01/mysql/logs/* /data01/mysql/run/* /data01/mysql/script/* /data01/mysql/dump/* /data01/mysql/binlog/* /data01/mysql/relaylog/*'
```
