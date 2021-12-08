# Automate Install Redis Master-Slave Sentinel with Ansible Playbook | [中文](README_ZH.md)

Please create a `redis` user on the linux server before starting. This playbook script uses Redis source code compilation to install one master, two slaves, and three sentinels

## Planning Your Installation

Planning for server

| IP | SSH PORT | SSH USER | SSH PASSWORD | ROOT PASSWORD | OS |
| ---- | ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | 22022 | redis | redis123 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.181 | 22022 | redis | redis123 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.182 | 22022 | redis | redis123 | root123 | CentOS Linux release 7.9.2009 |

**TIPS：** reference [Create User in Batch Automation](https://github.com/coolbeevip/ansible-playbook#create-user--group)

Planning for nodes

| IP | Redis | Sentinel |
| ---- | ---- | ---- |
| 10.1.207.180 | master | ✓ |
| 10.1.207.181 | slave | ✓ |
| 10.1.207.182 | slave | ✓ |

Planning for installation directory

| PATH | DESCRIPTION |
| /opt/redis | source package |
| ~/redis_uninstall.sh | uninstall script |
| ~/redis.sh | Redis start & stop script |
| ~/sentinel.sh | Sentinel start & stop script |
| /data01/redis/bin | redis executable file directory |
| /data01/redis/log | log directory |
| /data01/redis/data | data directory |
| /data01/redis/conf | configuration directory |

## Download Redis Tar & Ansible Playbook scripts

Create a directory for Ansible Playbook scripts

```shell
mkdir -p ~/my-docker-volume/ansible-playbook
```

Download Ansible playbook scripts

```shell
cd ~/my-docker-volume/ansible-playbook
git clone https://github.com/coolbeevip/ansible-playbook.git
```

Download Redis tar ball from the redis.io web site to `~/my-docker-volume/ansible-playbook/packages`

```shell
wget -P ~/my-docker-volume/ansible-playbook/packages https://download.redis.io/releases/redis-6.2.6.tar.gz --no-check-certificate
```

## Configuration

> You can edit the following configuration files to modify the default parameters

#### main-install.yml

Edit `redis/main.yml` script, You can define the target server address here. In the following snippet, you can see that three servers and logged in using  `redis`.

```yaml
- hosts: 10.1.207.180
  user: redis
  ...
- hosts: 10.1.207.181
  user: redis
  ...
- hosts: 10.1.207.182
  user: redis
  ...
```

#### var_redis.yml

You can define the installation directory, redis tar version, port, default master node address, etc.

```shell
# 源码包
redis_tar: "redis-6.2.6.tar.gz"       # 源码包文件名
redis_tar_unzip_dir: "redis-6.2.6"    # 源码包解压后的目录名

# 安装目录
redis_home_dir: "/opt/redis"          # 源码包上传目录
redis_bin_dir: "/data01/redis/bin"    # 编译后程序文件
redis_log_dir: "/data01/redis/logs"   # 运行日志，PID 文件
redis_data_dir: "/data01/redis/data"  # 数据文件
redis_conf_dir: "/data01/redis/conf"  # 配置文件

# 操作系统用户和组
redis_user: "redis"                   # 操作系统用户名
redis_group: "redis"                  # 操作系统组名

# redis configuration
redis_password: "redis"               # redis 密码
redis_network_host: "0.0.0.0"         # 服务绑定 IP 地址
redis_port: 7000                      # Redis 服务端口
redis_maxmemory: 31457280             # 最大内存
redis_maxclients: 1000                # 客户端最大连接数
redis_io_threads_do_reads: "yes"      # 开启多线程支持
redis_io_threads: 4                   # 线程数，线程数一定要小于机器核数并且不要大于 8
redis_master_ip: 10.1.207.180         # 主节点 IP 地址

# sentinel configuration
redis_sentinel_port: 27000            # Sentinel 端口
redis_sentinel_master: "mymaster"     # 监控 master 名称
redis_sentinel_master_quorum: 2       # master 切换投票数，当集群中有 N 个sentinel 认为 master 宕机后就切换
redis_sentinel_down_after_milliseconds: 5000  # master ping 检测超时时间
redis_sentinel_failover_timeout: 10000        # 故障切换超时时间
redis_sentinel_parallel_syncs: 2              # 故障切换后，每次向新的主节点发起复制操作的从节点个数
```

You can find more default configuration in `redis/config/redis.conf.j2` and `redis/config/sentinel.conf.j2` files

## Installation

Start the ansible container tool to connect to the target server, And mount directory `~/my-docker-volume/ansible-playbook` in the container.

**NOTICE:** ANSIBLE_SSH_USERS，ANSIBLE_SSH_PASSS is linux user redis and password

**NOTICE:** ANSIBLE_SU_PASSS is user root password

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

#### Install Redis Master-Slave & Sentinel

This script will automate the following operations

* Install Redis build dependency libraries
* Upload the Redis source package to each server
* Configure Redis execution file PATH path
* Configure Redis and Sentinel configuration files
* Start Redis and Sentinel services

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/redis/main-install.yml /ansible-playbook/redis/main-start-redis.yml /ansible-playbook/redis/main-start-sentinel.yml
```

**TIPS:** Because the Redis installation package (about 2.5MB) will be uploaded to all servers and buid(1 minutes per server) when the script is executed for the first time. The first installation on my local machine takes < 5 minutes.

**TIPS:** This script is only used for initial installation. Repeated execution of this command may receive a prompt of `Redis has been installed, please uninstall and then reinstall`. At this time, you need to use `ansible all -m shell -a '~/redis_uninstall.sh'` uninstalls.

If you see the following message, the installation is completed

```shell
TASK [Install Succeed] ********************************************************************************************************************************************************************************************************
ok: [10.1.207.180] => {
    "msg": "Install Succeed!"
}
```

#### Verify Redis Master-Slave & Sentinel

Check the Redis process information of each target server, you can see that the Redis and Sentinel processes of each node have been started

```shell
bash-5.0# ansible all -m shell -a 'ps -ef | grep [b]in/redis-'
10.1.207.180 | CHANGED | rc=0 >>
redis    19121     1  0 18:59 ?        00:00:00 /data01/redis/bin/redis-server 0.0.0.0:7000
redis    20011     1  0 19:01 ?        00:00:00 /data01/redis/bin/redis-sentinel 0.0.0.0:27000 [sentinel]

10.1.207.182 | CHANGED | rc=0 >>
redis    22537     1  0 18:55 ?        00:00:00 /data01/redis/bin/redis-server 0.0.0.0:7000
redis    23467     1  0 18:59 ?        00:00:00 /data01/redis/bin/redis-sentinel 0.0.0.0:27000 [sentinel]

10.1.207.181 | CHANGED | rc=0 >>
redis    10009     1  0 18:55 ?        00:00:00 /data01/redis/bin/redis-server 0.0.0.0:7000
redis    11187     1  0 18:59 ?        00:00:00 /data01/redis/bin/redis-sentinel 0.0.0.0:27000 [sentinel]
```

View the replication information of each node

```shell
bash-5.0# ansible all -m shell -a 'redis-cli -h 0.0.0.0 -p 7000 -a redis info Replication'

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

Get the address of the master host by using Sentinel and any Redis host

```shell
bash-5.0# ansible all -m shell -a 'redis-cli -h 10.1.207.182 -p 27000 sentinel get-master-addr-by-name mymaster | head -n 1'
10.1.207.181 | CHANGED | rc=0 >>
10.1.207.180

10.1.207.182 | CHANGED | rc=0 >>
10.1.207.180

10.1.207.180 | CHANGED | rc=0 >>
10.1.207.180
```

Benchmark test, each command executes 10,000 operations, the data size is 65536 bytes (64KB), and a random 1/10000 KEY is used. This command takes 1 minute to execute in my local machine

```shell
bash-5.0# ansible all -m shell -a 'redis-benchmark -h 10.1.207.180 -p 7000 -a redis -r 10000 -n 10000 -t get,set,hset,lpush,lpop -q -d 65536'
10.1.207.181 | CHANGED | rc=0 >>
SET: 300.51 requests per second, p50=135.551 msec
GET: 785.85 requests per second, p50=42.847 msec
LPUSH: 216.22 requests per second, p50=154.495 msec
LPOP: 953.29 requests per second, p50=36.191 msec
HSET: 835.77 requests per second, p50=48.127 msec

10.1.207.180 | CHANGED | rc=0 >>
SET: 300.77 requests per second, p50=138.111 msec
GET: 888.42 requests per second, p50=38.559 msec
LPUSH: 216.29 requests per second, p50=156.799 msec
LPOP: 988.04 requests per second, p50=33.919 msec
HSET: 780.34 requests per second, p50=51.551 msec

10.1.207.182 | CHANGED | rc=0 >>
SET: 290.66 requests per second, p50=137.087 msec
GET: 841.89 requests per second, p50=38.463 msec
LPUSH: 210.99 requests per second, p50=157.311 msec
LPOP: 967.12 requests per second, p50=35.103 msec
HSET: 902.04 requests per second, p50=45.375 msec
```

**提示：** For example, `GET: 841.89 requests per second, p50=38.463 msec` means 841 requests per second are completed, and the throughput per second is about 53MB (841*64KB), which basically matches my network bandwidth

## Common Maintenance Commands

**TIPS：** Need to start each node in the order of `Master->Slave->Sentinel`

Start Redis

```shell
bash-5.0# ansible all -m shell -a '~/redis.sh start'
```

Stop Redis

```shell
bash-5.0# ansible all -m shell -a '~/redis.sh stop'
```

Restart Redis

```shell
bash-5.0# ansible all -m shell -a '~/redis.sh restart'
```

Start Sentinel

```shell
bash-5.0# ansible all -m shell -a '~/sentinel.sh start'
```

Stop Sentinel

```shell
bash-5.0# ansible all -m shell -a '~/sentinel.sh stop'
```

Restart Sentinel

```shell
bash-5.0# ansible all -m shell -a '~/sentinel.sh restart'
```

## Q & A

#### How to force uninstall

A: The `~/redis_uninstall.sh` script will **kill -9 all Redis and Sentinel processes and delete programs and all data**

```shell
bash-5.0# ansible all -m shell -a '~/redis_uninstall.sh'
10.1.207.182 | CHANGED | rc=0 >>
Kill Redis Process
Kill Sentinel Process
Delete Redis Data Files
Delete Redis Package
Delete redis_uninstall.sh

10.1.207.181 | CHANGED | rc=0 >>
Kill Redis Process
Kill Sentinel Process
Delete Redis Data Files
Delete Redis Package
Delete redis_uninstall.sh

10.1.207.180 | CHANGED | rc=0 >>
Kill Redis Process
Kill Sentinel Process
Delete Redis Data Files
Delete Redis Package
Delete redis_uninstall.sh
```
