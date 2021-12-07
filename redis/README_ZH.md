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
| 10.1.207.180 | master | ✓ |
| 10.1.207.181 | slave | ✓ |
| 10.1.207.182 | slave | ✓ |

节点安装路径

| 路径 | 描述 |
| ---- | ---- |
| /opt/redis | 源代码上传路径 |
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

> 你可以编辑如下文件修改默认配置

#### main-install.yml

您可以在此处定义目标服务器，您可以看到这里定义了三个目标服务器，使用 `redis` 用户登录。

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

您可以在此处定义安装目录，版本，端口，默认主节点地址等配置信息。

```properties
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

您可以在 `redis/config/redis.conf.j2` 和 `redis/config/sentinel.conf.j2` 文件中找到更多的默认配置

## 开始安装

启动 ansible 容器工具连接目标服务器，并将 `~/my-docker-volume/ansible-playbook` 目录挂在到容器中。

**提示：** ANSIBLE_SSH_USERS，ANSIBLE_SSH_PASSS 配置成您之前在目标服务器上创建的用户名 `redis` 和密码 `redis123`

**提示：** ANSIBLE_SU_PASSS 为 root 用户的密码

```shell
docker run --name ansible --rm -it \
  -e ANSIBLE_SSH_HOSTS=10.1.207.180,10.1.207.181,10.1.207.182 \
  -e ANSIBLE_SSH_PORTS=22022,22022,22022 \
  -e ANSIBLE_SSH_USERS=redis,redis,redis \
  -e ANSIBLE_SSH_PASSS=redis123,redis123,redis123 \
  -e ANSIBLE_SU_PASSS=root123,root123,root123 \
  -v ~/my-docker-volume/ansible-playbook:/ansible-playbook \
  coolbeevip/ansible:2.8.11-alpine \
  /bin/bash
bash-5.0#  
```

## 安装集群

这个脚本将自动化完成如下操作：

* 安装 Redis 编译用依赖库
* 上传 Redis 源码包到每个服务器
* 配置 Redis 执行文件 PATH 路径
* 配置 Redis 和 Sentinel 配置文件
* 启动 Redis 和 Sentinel 服务

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

**提示:** 因为第一次执行脚本时，会上传 Redis 源码包（约 2.5MB）到所有服务器并在服务器上编译（每服务器约 1分钟）。在我本地环境首次安装大概耗时 5 分钟

**提示:** 此脚本只适合初始化安装，重复执行此命令可能会收到 `Redis has been installed, please uninstall and then reinstall` 提示，此时需要先要使用 `ansible all -m shell -a '~/redis_uninstall.sh'` 命令卸载之前的安装。

## 验证集群

查看每个目标服务器 redis 进程信息，可以看到每个节点 redis 和 sentinel 进程都已经启动

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

查看每个节点的复制信息

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

连接任意哨兵节点获取 Redis Master 地址

```shell
bash-5.0# ansible all -m shell -a 'redis-cli -h 10.1.207.182 -p 27000 sentinel get-master-addr-by-name mymaster | head -n 1'
10.1.207.181 | CHANGED | rc=0 >>
10.1.207.180

10.1.207.182 | CHANGED | rc=0 >>
10.1.207.180

10.1.207.180 | CHANGED | rc=0 >>
10.1.207.180
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
