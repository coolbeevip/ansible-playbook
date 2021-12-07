# Ansible Playbook 自动化安装 Redis 主从哨兵 | [English](README.md)

请先在目标服务器创建 `redis` 用户，此脚本在此用户下使用源代码编译的方式安装一主两从三哨兵的集群

## 安装计划

服务器规划

| IP | SSH 端口 | SSH 用户名 | SSH 密码 | ROOT 密码 | OS |
| ---- | ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | 22022 | redis | redis123 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.181 | 22022 | redis | redis123 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.182 | 22022 | redis | redis123 | root123 | CentOS Linux release 7.9.2009 |

**提示：** 可以参考[批量自动化创建用户](https://github.com/coolbeevip/ansible-playbook/blob/main/README_ZH.md#%E5%88%9B%E5%BB%BA%E7%94%A8%E6%88%B7%E5%92%8C%E7%BB%84)

集群节点规划

| IP | Redis | Sentinel |
| ---- | ---- | ---- |
| 10.1.207.180 | ✓ | ✓ |
| 10.1.207.181 | ✓ | ✓ |
| 10.1.207.182 | ✓ | ✓ |

节点安装路径

| 路径 | 描述 |
| ---- | ---- |
| /opt/redis | RPM 安装介质 |
| ~/redis_uninstall.sh | 集群卸载脚本 |
| ~/redis.sh | Redis 启停脚本 |
| ~/sentinel.sh | Sentinel 启停脚本 |
| /data01/redis/bin | redis 编译后的程序 |
| /data01/redis/log | 日志目录 |
| /data01/redis/data | 数据目录 |
| /data01/redis/conf | 配置文件目录  |

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

下载 redis 安装包到 `~/my-docker-volume/ansible-playbook/packages` 目录

```shell
wget -P ~/my-docker-volume/ansible-playbook/packages https://download.redis.io/releases/redis-6.2.6.tar.gz --no-check-certificate
```

## 配置安装脚本

打开 `redis/main-install.yml` 脚本，您可以在此处定义目标服务器，您可以看到这里定义了三个目标服务器，使用 `redis` 用户登录，并将 `10.1.207.180` 服务器设置为 master 节点

```yaml
- hosts: 10.1.207.180
  user: redis
  ...
  vars:
    im_master: true
  tasks:
    ...
- hosts: 10.1.207.181
  user: redis
  ...
- hosts: 10.1.207.182
  user: redis
  ...
```

在 `var_redis.yml` 文件中定义了安装目录，版本，端口，默认主节点地址等配置信息。其中 **redis_port**、**redis_master_ip** 和您的真实规划有关

```environment
---
redis_tar: "redis-6.2.6.tar.gz"
redis_tar_unzip_dir: "redis-6.2.6"
redis_home_dir: "/opt/redis"
redis_log_dir: "/opt/redis/logs"
redis_data_dir: "/opt/redis/data"
redis_conf_dir: "/opt/redis/conf"
redis_user: "redis"
redis_group: "redis"
redis_password: "redis"
redis_network_host: "0.0.0.0"
redis_port: 7000
redis_maxmemory: 31457280
redis_maxclients: 1000
redis_master_ip: 10.1.207.180

redis_sentinel_port: 27000
redis_sentinel_master: "mymaster"
redis_sentinel_master_quorum: 2
redis_sentinel_down_after_milliseconds: 5000
redis_sentinel_failover_timeout: 10000
redis_sentinel_parallel_syncs: 2
```

您可以在 `redis/config/redis.conf.j2` 和 `redis/config/sentinel.conf.j2` 文件中找到更多的默认配置

## 安装集群

启动 ansible 容器工具连接目标服务器，并将 `~/my-docker-volume/ansible-playbook` 目录挂在到容器中。

**提示：** ANSIBLE_SSH_USERS，ANSIBLE_SSH_PASSS 配置成您之前在目标服务器上创建的用户名 `redis` 和密码 `123456`

**提示：** ANSIBLE_SU_PASSS 为 root 用户的密码

```shell
docker run --name ansible --rm -it \
  -e ANSIBLE_SSH_HOSTS=10.1.207.180,10.1.207.181,10.1.207.182 \
  -e ANSIBLE_SSH_PORTS=22022,22022,22022 \
  -e ANSIBLE_SSH_USERS=redis,redis,redis \
  -e ANSIBLE_SSH_PASSS=123456,123456,123456 \
  -e ANSIBLE_SU_PASSS=root123,root123,root123 \
  -v ~/my-docker-volume/ansible-playbook:/ansible-playbook \
  coolbeevip/ansible:2.8.11-alpine \
  /bin/bash
bash-5.0#  
```

执行安装脚本

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/redis/main-install.yml /ansible-playbook/redis/main-start-redis.yml /ansible-playbook/redis/main-start-sentinel.yml
```

如果你看到如下信息，说明安装完成

```shell
TASK [Install Succeed] ********************************************************************************************************************************************************************************************************
ok: [10.1.207.180] => {
    "msg": "Install Succeed!"
}
```

**提示：** 此脚本在我的环境下执行耗时大约 3 分钟

## 验证集群

查看目标服务器 redis 进程信息，可以看到每个节点的redis和哨兵进程都已经启动

```shell
bash-5.0# ansible all -m shell -a 'ps -ef | grep [b]in/redis-'
10.1.207.181 | CHANGED | rc=0 >>
redis    10009     1  0 18:55 ?        00:00:00 /data01/redis/bin/redis-server 0.0.0.0:7000

10.1.207.180 | CHANGED | rc=0 >>
redis    19121     1  0 18:59 ?        00:00:00 /data01/redis/bin/redis-server 0.0.0.0:7000

10.1.207.182 | CHANGED | rc=0 >>
redis    22537     1  0 18:55 ?        00:00:00 /data01/redis/bin/redis-server 0.0.0.0:7000
```

查看每个节点的状态，您可以看到主从节点信息

```shell
bash-5.0# ansible all -m shell -a '/opt/redis/bin/redis-cli -h 0.0.0.0 -p 7000 -a redis info Replication'

10.1.207.182 | CHANGED | rc=0 >>
# Replication
role:slave
master_host:10.1.207.180
master_port:7000
master_link_status:up
master_last_io_seconds_ago:1
master_sync_in_progress:0
slave_read_repl_offset:73780
slave_repl_offset:73780
slave_priority:100
slave_read_only:1
replica_announced:1
connected_slaves:0
master_failover_state:no-failover
master_replid:4dc6d9bc49043bc1bc8c495f88c470523af3b030
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:73780
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:73780Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.

10.1.207.181 | CHANGED | rc=0 >>
# Replication
role:slave
master_host:10.1.207.180
master_port:7000
master_link_status:up
master_last_io_seconds_ago:0
master_sync_in_progress:0
slave_read_repl_offset:73919
slave_repl_offset:73919
slave_priority:100
slave_read_only:1
replica_announced:1
connected_slaves:0
master_failover_state:no-failover
master_replid:4dc6d9bc49043bc1bc8c495f88c470523af3b030
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:73919
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:73919Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.

10.1.207.180 | CHANGED | rc=0 >>
# Replication
role:master
connected_slaves:2
slave0:ip=10.1.207.182,port=7000,state=online,offset=73502,lag=0
slave1:ip=10.1.207.181,port=7000,state=online,offset=73502,lag=1
master_failover_state:no-failover
master_replid:4dc6d9bc49043bc1bc8c495f88c470523af3b030
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:73780
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:73780Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.

bash-5.0#
```

## 常用运维命令

**提示：** 哨兵模式集群需要按 `Master->Slave->Sentinel` 顺序启动各个节点

启动 Redis

```shell
bash-5.0# ansible all -m shell -a '~/redis.sh start'
```

停止 Redis

```shell
bash-5.0# ansible all -m shell -a '~/redis.sh stop'
```

重启 Redis

```shell
bash-5.0# ansible all -m shell -a '~/redis.sh restart'
```

启动 Sentinel

```shell
bash-5.0# ansible all -m shell -a '~/sentinel.sh start'
```

停止 Sentinel

```shell
bash-5.0# ansible all -m shell -a '~/sentinel.sh stop'
```

重启 Sentinel

```shell
bash-5.0# ansible all -m shell -a '~/sentinel.sh restart'
```

## Q & A

#### 如何彻底删除 Redis 主从哨兵集群

A: `~/redis_uninstall.sh` 脚本将 **kill Redis 和 Sentinel，删除程序文件和所有数据文件**

```shell
bash-5.0# ansible all -m shell -a '~/redis_uninstall.sh'
10.1.207.180 | CHANGED | rc=0 >>


10.1.207.182 | CHANGED | rc=0 >>


10.1.207.181 | CHANGED | rc=0 >>
