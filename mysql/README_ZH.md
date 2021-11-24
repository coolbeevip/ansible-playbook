# Ansible Playbook 安装 MySQL 集群 | [English](README.md)


## 目标服务器

假设您有以下三台服务器

| IP地址 | SSH 端口 | SSH 用户名 | SSH 密码 | ROOT 密码 |
| ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | 22022 | mysql | 123456 | root123 |
| 10.1.207.181 | 22022 | mysql | 123456 | root123 |
| 10.1.207.182 | 22022 | mysql | 123456 | root123 |

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

启动 ansible 容器工具连接目标服务器，并将 `~/my-docker-volume/ansible-playbook` 目录挂在到容器中。

**提示：** ANSIBLE_SSH_USERS，ANSIBLE_SSH_PASSS 配置成您之前在目标服务器上创建的用户名 `redis` 和密码 `123456`

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

安装三节点 mysql 实例

> 自动上传安装介质，初始化 root 用户，启动 mysql 实例

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/mysql/main-mysql.yml
```

检查 mysql 节点状态

```shell
bash-5.0# ansible all -m shell -a '/opt/mysql/mysql-8.0.27-linux-glibc2.12-x86_64/support-files/mysql.server status'
10.1.207.181 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (25729)

10.1.207.180 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (9462)

10.1.207.182 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (28934)
```

校验三节点值间的连接是否正常

> 我们任选一个节点执行校验状态，可以看到节点之间是可以相互连接的

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

创建三节点集群

> 检查 mysql 实例间连接是否正常，选择一个节点创建集群并设置为主节点，将另两个节点作为从节点加入到集群，执行完毕后会等待三个节点重启完毕

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/mysql/main-cluster.yml
```

mysqlsh root@127.0.0.1:3336 -- cluster status

至此，已经安装完毕，您可以在每个目标服务的安装目录下看到已经配置完毕的 mysql 节点。

启动 mysql

```shell
bash-5.0# ansible all -m shell -a '/opt/mysql/mysql-8.0.27-linux-glibc2.12-x86_64/support-files/mysql.server start'
10.1.207.182 | CHANGED | rc=0 >>
Starting MySQL........ SUCCESS!

10.1.207.181 | CHANGED | rc=0 >>
Starting MySQL........ SUCCESS!

10.1.207.180 | CHANGED | rc=0 >>
Starting MySQL........... SUCCESS!
```

停止 mysql

```shell
bash-5.0# ansible all -m shell -a '/opt/mysql/mysql-8.0.27-linux-glibc2.12-x86_64/support-files/mysql.server stop'
10.1.207.182 | CHANGED | rc=0 >>
Shutting down MySQL... SUCCESS!

10.1.207.181 | CHANGED | rc=0 >>
Shutting down MySQL... SUCCESS!

10.1.207.180 | CHANGED | rc=0 >>
Shutting down MySQL...... SUCCESS!
```

检查 mysql 服务状态

```shell
bash-5.0# ansible all -m shell -a '/opt/mysql/mysql-8.0.27-linux-glibc2.12-x86_64/support-files/mysql.server status'
10.1.207.181 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (25729)

10.1.207.180 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (9462)

10.1.207.182 | CHANGED | rc=0 >>
 SUCCESS! MySQL running (28934)
```

清理安装时生成的脚本文件和安装介质

> /data01/mysql/script 目录下是存储初始化的脚本，安装完毕后可以删除（**因为里面包含 root 密码等敏感信息**）

```shell
bash-5.0# ansible all -m shell -a 'rm /data01/mysql/script/*'
```

> /opt/mysql 目录下的安装介质装完后可以删除

```shell
bash-5.0# ansible all -m shell -a 'rm /opt/*.tar.*'
```

## Q & A

Q: initialize mysql 时失败，查看 `/data01/mysql/logs/mysqld.err` 文件中提示 `Resource temporarily unavailable`
A: 请检查服务器内存是否够用

Q: 如何彻底删除数据文件
A:

```shell
bash-5.0# ansible all -m shell -a '/opt/mysql/mysql-8.0.27-linux-glibc2.12-x86_64/support-files/mysql.server stop'
bash-5.0# ansible all -m shell -a 'rm -rf /data01/mysql/data/* /data01/mysql/logs/* /data01/mysql/run/* /data01/mysql/script/* /data01/mysql/dump/* /data01/mysql/binlog/* /data01/mysql/relaylog/*'
```
