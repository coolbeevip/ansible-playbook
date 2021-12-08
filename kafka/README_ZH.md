# Ansible Playbook 自动化安装 Kafka 集群 | [English](README.md)

## 概述

* 此脚本中的配置基于 Kafka 2.6.3 版本
* 使用 Kafka 发布包中自带的 Zookeeper 组件（通常这没有问题，除非你要将 Zookeeper 安装在单独的服务器上）
* 在 Kafka 安装目录下独立安装 JDK，目前只支持 Java 8 和 Java 11。

## 安装计划

服务器规划

| IP | SSH 端口 | SSH 用户 | SSH 密码 | ROOT 密码 | OS |
| ---- | ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | 22022 | kafka | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.181 | 22022 | kafka | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.182 | 22022 | kafka | 123456 | root123 | CentOS Linux release 7.9.2009 |

**提示：** 可以参考[批量自动化创建用户](https://github.com/coolbeevip/ansible-playbook/blob/main/README_ZH.md#%E5%88%9B%E5%BB%BA%E7%94%A8%E6%88%B7%E5%92%8C%E7%BB%84)

集群节点规划

| IP地址 | Kafka | Zookeeper | Java |
| ---- | ---- | ---- | ---- |
| 10.1.207.180 | ✓ | ✓ | ✓ |
| 10.1.207.181 | ✓ | ✓ | ✓ |
| 10.1.207.182 | ✓ | ✓ | ✓ |

节点安装路径

| 路径 | 描述 |
| ---- | ---- |
| /opt/kafka | Kafka, Zookeeper, Java 软件安装路径 |
| ~/zookeeper.sh | Zookeeper 启动，停止，状态命令脚本|
| ~/kafka.sh | Kafka 启动，停止，状态命令脚本|
| ~/kafka_uninstall.sh | Kafka 集群卸载脚本|
| /data01/kafka/kafka_data | Kafka 数据目录 |
| /data01/kafka/kafka_log | Kafka 日志目录 |
| /data01/kafka/zookeeper_data | Zookeeper 数据目录 |
| /data01/kafka/zookeeper_log | Zookeeper 日志目录 |

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

从 https://kafka.apache.org/downloads 下载 Kafka 安装包到 `~/my-docker-volume/ansible-playbook/packages` 目录

```shell
wget -P ~/my-docker-volume/ansible-playbook/packages https://dlcdn.apache.org/kafka/2.6.3/kafka_2.12-2.6.3.tgz --no-check-certificate
```

下载 JDK 安装包 `jdk-8u202-linux-x64.tar.gz` 到 `~/my-docker-volume/ansible-playbook/packages` 目录

## 配置安装脚本

> 您可以编辑以下配置文件，修改默认参数

#### main.yml

安装 Kafka 集群的服务器 IP 地址，以及系统用户名

```yaml
- hosts: 10.1.207.180
  user: kafka

- hosts: 10.1.207.181
  user: kafka

- hosts: 10.1.207.182
  user: kafka
```

### vars_kafka.yml

操作系统 Limits

```yaml
# Linux limits
limits_hard_nproc: '65535'
limits_soft_nproc: '65535'
limits_hard_nofile: '65535'
limits_soft_nofile: '65535'
```

安装用的用户名、用户组

```yaml
# Linux user & group
kafka_user: "kafka"
kafka_group: "kafka"
```

安装介质名称以及解压后的目录名

```yaml
# Kafka package
kafka_tar: "kafka_2.12-2.6.3.tgz"
kafka_tar_unzip_dir: "kafka_2.12-2.6.3"

# Java package
java_tar: "jdk-8u202-linux-x64.tar.gz"
java_tar_unzip_dir: "jdk-8u202-linux-x64"
```

安装目录

```yaml
kafka_home_dir: "/opt/kafka"
kafka_data_dir: "/data01/kafka/kafka_data"
kafka_log_dir: "/data01/kafka/kafka_log"
zookeeper_data_dir: "/data01/kafka/zookeeper_data"
zookeeper_log_dir: "/data01/kafka/zookeeper_log"
```

kafka & Zookeeper 在每个服务器上的 ID

```yaml
# Node configuration
hosts:
  10.1.207.180:
    zookeeper_id: 180
    kafka_broker_id: 180
  10.1.207.181:
    zookeeper_id: 181
    kafka_broker_id: 181
  10.1.207.182:
    zookeeper_id: 182
    kafka_broker_id: 182
```    

Zookeeper 服务配置

```yaml
zookeeper_client_port: 2181 # the port at which the clients will connect
zookeeper_follower_port: 2888 # the port for follower connections
zookeeper_election_port: 3888 # the port for other server connections during the leader election phase
zookeeper_max_client_cnxns: 0 # setting number of connections to unlimited
zookeeper_tick_time: 2000 # keeps a heartbeat of zookeeper in milliseconds
zookeeper_init_limit: 10  # time for initial synchronization
zookeeper_sync_limit: 5   # how many ticks can pass before timeout
```

Kafka 服务配置

```yaml
kafka_server_port: 9003
kafka_server_num_partitions: 8 # default number of partitions
kafka_server_offsets_topic_replication_factor: 3 # The replication factor for the group metadata internal topics "__consumer_offsets"
kafka_server_transaction_state_log_replication_factor: 3 # The replication factor for the group metadata internal topics "__transaction_state"
kafka_server_transaction_state_log_min_isr: 2
kafka_server_default_replication_factor: 3 # default replica count based on the number of brokers
kafka_server_min_insync_replicas: 2 # to protect yourself against broker failure
kafka_server_zookeeper_connection_timeout_ms: 6000 # timeout for connecting with zookeeper
```

更多Kafka 配置参见[2.6.X](https://kafka.apache.org/26/documentation.html)

## 开始安装

启动 ansible 容器工具连接目标服务器，并将 `~/my-docker-volume/ansible-playbook` 目录挂载到容器中。

提示： ANSIBLE_SSH_USERS，ANSIBLE_SSH_PASSS 配置成您之前在目标服务器上创建的用户名 kafka 和密码 123456

提示： ANSIBLE_SU_PASSS 为 root 用户的密码

```shell
docker run --name ansible --rm -it \
  -e ANSIBLE_SSH_HOSTS=10.1.207.180,10.1.207.181,10.1.207.182 \
  -e ANSIBLE_SSH_PORTS=22022,22022,22022 \
  -e ANSIBLE_SSH_USERS=kafka,kafka,kafka \
  -e ANSIBLE_SSH_PASSS=123456,123456,123456 \
  -e ANSIBLE_SU_PASSS=root123,root123,root123 \
  -v /Users/zhanglei/mydocker/volume/ansible-playbook:/ansible-playbook \
  coolbeevip/ansible:2.8.11-alpine \
  /bin/bash  
```

#### 安装 Kafka 集群

这个脚本将自动化完成如下操作：

* 在每个服务器上配置系统参数 & 创建安装目录
* 上传 Kafka & JDK 安装介质上传到每个服务器
* 配置每个服务器上的 server.properties 和 zookeeper.properties
* 启动 Zookeeper & Kafka 服务
* 自动创建测试用 Topic 并简单测试

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/kafka/main.yml
```

**提示:** 因为第一次执行脚本时，会上传MySQL 安装包到所有服务器（约 260MB），所以执行时间较长（取决于你的客户端和服务器之间的网络速度）。 你也可以在执行以上脚本前手动将安装包上传到服务器的安装路径 /opt/kafka 下。在我本地环境首次安装大概耗时 25 分钟（上传安装包大概 5 分钟，安装集群大概 20 分钟）

**提示:** 此脚本只适合初始化安装，重复执行此命令可能会收到 Kafka has been installed, please uninstall and then reinstall 提示，此时需要先要使用 ansible all -m shell -a '~/kafka_uninstall.sh' 命令卸载之前的安装。

如果你看到如下信息，说明安装完成

```shell
TASK [Install Succeed] ********************************************************************************************************************************************************************************************************
ok: [10.1.207.180] => {
    "msg": "Install Succeed!"
}
```

#### 验证 Kafka 集群

Test the Zookeeper server state.
测试服务是否处于正确状态。如果确实如此，那么服务返回“imok ”，否则不做任何响应

```shell
$ echo ruok | nc 10.1.207.180 9002;echo
```

Show list of active brokers IDs on the cluster.

```shell
zookeeper-shell.sh 10.1.207.180:9002 ls /brokers/ids
```

Returns the details of the broker with the given ID

```shell
zookeeper-shell.sh 10.1.207.180:9002 get /brokers/ids/<id>
```

Outputs a list of variables that could be used for monitoring the health of the cluster.

```shell
$ echo mntr | nc 10.1.207.180 9002
```

## 常用运维命令

#### Kafka

启动 Kafka

```shell
bash-5.0# ansible all -m shell -a '~/kafka.sh start'
```

停止 Kafka

```shell
bash-5.0# ansible all -m shell -a '~/kafka.sh stop'
```

查看 Kafka 状态

```shell
bash-5.0# ansible all -m shell -a '~/kafka.sh status'
10.1.207.182 | CHANGED | rc=0 >>
Kafka is Running as PID: 4140
32752
32753

10.1.207.181 | CHANGED | rc=0 >>
Kafka is Running as PID: 6949
24057
24058

10.1.207.180 | CHANGED | rc=0 >>
Kafka is Running as PID: 7940
7941
```

#### Zookeeper

启动 Zookeeper

```shell
bash-5.0# ansible all -m shell -a '~/zookeeper.sh start'
```

停止 Zookeeper

```shell
bash-5.0# ansible all -m shell -a '~/zookeeper.sh stop'
```

查看 Zookeeper 状态

```shell
bash-5.0# ansible all -m shell -a '~/zookeeper.sh status'
10.1.207.180 | CHANGED | rc=0 >>
Zookeeper is Running as PID: 16996

10.1.207.182 | CHANGED | rc=0 >>
Zookeeper is Running as PID: 2742

10.1.207.181 | CHANGED | rc=0 >>
Zookeeper is Running as PID: 3124
```

Display configuration.

```shell
$ echo conf | nc 10.1.207.180 9002
```

List connection details to this server.

```shell
$ echo cons | nc 10.1.207.180 9002
```

Display outstanding sessions and ephemeral nodes.

```shell
$ echo dump | nc 10.1.207.180 9002
```

Display environment settings.

```shell
$ echo envi | nc 10.1.207.180 9002
```

Display the total size of snapshot and log files in bytes.

```shell
$ echo dirs | nc 10.1.207.180 9002
```

Check if server is running in read-only mode.

```shell
$ echo isro | nc 10.1.207.180 9002; echo
```

## Q & A

#### 如何彻底删除 Kafka 集群

Q: 如何彻底删除使用自脚本安装的集群程序和数据

A: `~/kafka_uninstall.sh` 脚本将 **kill -9 MySQL server 和 MySQL router，删除程序文件和所有数据文件**

```shell
bash-5.0# ansible all -m shell -a '~/kafka_uninstall.sh'
10.1.207.180 | CHANGED | rc=0 >>


10.1.207.182 | CHANGED | rc=0 >>


10.1.207.181 | CHANGED | rc=0 >>
```

#### 生产环境

**CPUs：** Kafka 对 CPU 要求较低（除非你开启了 SSL），建议使用不低于 24 核的 CPUs

**内存：** Kafka 严重依赖文件系统和内存，建议使用不低于 32GB 内存的机器（64GB最好）。

**磁盘：** 不要与应用程序日志或者其他操作系统文件系统共享 Kafka 数据盘，以确保最小的延迟。最优策略为每个节点使用 RAID5  或者 RAID10 挂载数据目录，每个逻辑盘不超过 8 块（对于大多数用例，建议将 RAID 10 作为最佳选项。它提供了改进的读写性能、数据保护和快速重建时间）。

你也可以挂载多块磁盘来最大化吞吐量，并且设置参数 `num.io.thread` 与挂盘数一致，估算算法如下：

每个节点挂盘数 <= CPU逻辑核数 / 2

**网络：**

快速可靠的网络是分布式系统中必不可少的性能组件。低延迟确保节点可以轻松通信，而高带宽有助于分片移动和恢复。现代数据中心网络（1 GbE、10 GbE）足以满足绝大多数集群的需求。建议使用[iperf](https://coolbeevip.github.io/posts/linux/linux-commands-network/)先测试一下。

**文件系统：** 建议在 XFS 或 ext4 上运行 Kafka

**JVM：** 不需要设置超过 6 GB 的堆大小。这将导致 32 GB 机器上的文件系统缓存高达 28-30 GB，您需要足够的内存来缓冲活动的读取器和写入器。你可以用一下公示估算：

* 每秒吞度量 = 每条消息字节数 * 每秒消息数；

例如：每条消息字节数 600 字节，每秒消息数 17000，那么吞吐率大概为 10MB/秒

* 缓冲大小 = 需要缓冲的时间 * 每秒吞度量 / (Topic 分区数 * 副本数)；

例如：需要缓冲 30 秒，每秒吞吐率 10MB，Topic 5 个分区，3 个副本，那么需要缓冲区为 180MB