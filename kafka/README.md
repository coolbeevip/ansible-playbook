# Automate Install Kafka Cluster with Ansible Playbook | [中文](README_ZH.md)

## Overview

* The configuration parameters in this script are based on Kafka 2.6.3 version
* Use use the bundled Zookeeper with the Kafka release package (usually this is no problem, unless you want to install Zookeeper on other servers)
* Install the dedicated JDK in the Kafka installation directory, Java 8 and Java 11 are supported.

## Planning Your Installation

Planning for server

| IP | SSH PORT | SSH USER | SSH PASSWORD | ROOT PASSWORD | OS |
| ---- | ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | 22022 | kafka | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.181 | 22022 | kafka | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.182 | 22022 | kafka | 123456 | root123 | CentOS Linux release 7.9.2009 |

**TIPS：** reference [Create User in Batch Automation](https://github.com/coolbeevip/ansible-playbook#create-user--group)

Planning for Kafka nodes

| IP | Kafka | Zookeeper | Java |
| ---- | ---- | ---- | ---- |
| 10.1.207.180 | ✓ | ✓ | ✓ |
| 10.1.207.181 | ✓ | ✓ | ✓ |
| 10.1.207.182 | ✓ | ✓ | ✓ |

Planning for installation directory

| PATH | DESCRIPTION |
| ---- | ---- |
| /opt/kafka | Installation path of Kafka, Zookeeper, Java |
| ~/zookeeper.sh | Zookeeper start & stop & status script|
| ~/kafka.sh | Kafka start & stop & status script|
| ~/kafka_uninstall.sh | Kafka Cluster Uninstall script|
| /data01/kafka/kafka_data |  Kafka Data |
| /data01/kafka/kafka_log |  Kafka Logs |
| /data01/kafka/zookeeper_data |  Zookeeper Data |
| /data01/kafka/zookeeper_log |  Zookeeper Logs |

## Download Kafka & Java & Ansible Playbook Scripts

Create a directory for Ansible Playbook scripts

```shell
mkdir -p ~/my-docker-volume/ansible-playbook
```

Download Ansible playbook scripts

```shell
cd ~/my-docker-volume/ansible-playbook
git clone https://github.com/coolbeevip/ansible-playbook.git
```

Download Kafka tar ball from the https://kafka.apache.org/downloads web site to `~/my-docker-volume/ansible-playbook/packages`

```shell
wget -P ~/my-docker-volume/ansible-playbook/packages https://dlcdn.apache.org/kafka/2.6.3/kafka_2.12-2.6.3.tgz --no-check-certificate
```

Download JDK tar ball `jdk-8u202-linux-x64.tar.gz` to `~/my-docker-volume/ansible-playbook/packages`

## Configuration

> You can edit the following configuration files to modify the default parameters

#### main.yml

```yaml
- hosts: 10.1.207.180
  user: kafka

- hosts: 10.1.207.181
  user: kafka

- hosts: 10.1.207.182
  user: kafka
```

### vars_kafka.yml

Linux Limits

```yaml
# Linux limits
limits_hard_nproc: '65535'
limits_soft_nproc: '65535'
limits_hard_nofile: '65535'
limits_soft_nofile: '65535'
```

Linux user and group

```yaml
# Linux user & group
kafka_user: "kafka"
kafka_group: "kafka"
```

Kafka & Java binary package and unzip directory

```yaml
# Kafka package
kafka_tar: "kafka_2.12-2.6.3.tgz"
kafka_tar_unzip_dir: "kafka_2.12-2.6.3"

# Java package
java_tar: "jdk-8u202-linux-x64.tar.gz"
java_tar_unzip_dir: "jdk-8u202-linux-x64"
```

Install directory

```yaml
kafka_home_dir: "/opt/kafka"
kafka_data_dir: "/data01/kafka/kafka_data"
kafka_log_dir: "/data01/kafka/kafka_log"
zookeeper_data_dir: "/data01/kafka/zookeeper_data"
zookeeper_log_dir: "/data01/kafka/zookeeper_log"
```

Kafka & Zookeeper IDs

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

Zookeeper server properties

```yaml
zookeeper_client_port: 2181 # the port at which the clients will connect
zookeeper_follower_port: 2888 # the port for follower connections
zookeeper_election_port: 3888 # the port for other server connections during the leader election phase
zookeeper_max_client_cnxns: 0 # setting number of connections to unlimited
zookeeper_tick_time: 2000 # keeps a heartbeat of zookeeper in milliseconds
zookeeper_init_limit: 10  # time for initial synchronization
zookeeper_sync_limit: 5   # how many ticks can pass before timeout
```

Kafka server properties

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

Kafka more details[2.6.X](https://kafka.apache.org/26/documentation.html)

## Installation

Start the ansible container tool to connect to the target server, And mount directory `~/my-docker-volume/ansible-playbook` in the container.

**TIPS:** ANSIBLE_SSH_USERS，ANSIBLE_SSH_PASSS is linux user kafka and password

**TIPS:** ANSIBLE_SU_PASSS is user root password

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

#### Install Kafka Cluster

This script will automate the following operations

* Configure operating system parameters on all server
* Upload the Kafka & Java packages to all server
* ...

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/kafka/main.yml
```

**TIPS:** Because the Kafka & Java package will be uploaded to all servers (about 65MB+194MB) when the script is executed for the first time, so take longer to execute. The first installation on my local machine takes < 25 minutes(upload package taske about 5 minutes, others take about 20 minutes.)

**TIPS:** This script is only used for initial installation. Repeated execution of this command may receive a prompt of `Kafka has been installed, please uninstall and then reinstall`. At this time, you need to use `ansible all -m shell -a '~/kafka_uninstall.sh'` uninstalls.

If you see the following message, the installation is completed

```shell
TASK [Install Succeed] ********************************************************************************************************************************************************************************************************
ok: [10.1.207.180] => {
    "msg": "Install Succeed!"
}
```

#### Verify Kafka Cluster

## Common Maintenance Commands

Start Kafka

```shell
bash-5.0# ansible all -m shell -a '~/kafka.sh start'
```

Stop Kafka

```shell
bash-5.0# ansible all -m shell -a '~/kafka.sh stop'
```

View status of Kafka

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

Start Zookeeper

```shell
bash-5.0# ansible all -m shell -a '~/zookeeper.sh start'
```

Stop Zookeeper

```shell
bash-5.0# ansible all -m shell -a '~/zookeeper.sh stop'
```

View status of Zookeeper

```shell
bash-5.0# ansible all -m shell -a '~/zookeeper.sh status'
10.1.207.180 | CHANGED | rc=0 >>
Zookeeper is Running as PID: 16996

10.1.207.182 | CHANGED | rc=0 >>
Zookeeper is Running as PID: 2742

10.1.207.181 | CHANGED | rc=0 >>
Zookeeper is Running as PID: 3124
```

## Q & A

#### Uninstall Kafka Cluster

Q: How to Uninstall Kafka Cluster

A: `~/kafka_uninstall.sh` script will **kill -9 Kafka and Zookeeper, delete program files and data files**

```shell
bash-5.0# ansible all -m shell -a '~/kafka_uninstall.sh'
10.1.207.180 | CHANGED | rc=0 >>


10.1.207.182 | CHANGED | rc=0 >>


10.1.207.181 | CHANGED | rc=0 >>
```
