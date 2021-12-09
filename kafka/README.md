# Automate Install Kafka Cluster with Ansible Playbook | [中文](README_ZH.md)

## Overview

* The configuration parameters in this script are based on Kafka 2.6.3 version
* Use use the bundled Zookeeper with the Kafka release package (usually this is no problem, unless you want to install Zookeeper on other servers)
* Install the dedicated JDK in the Kafka installation directory, Java 8 and Java 11 are supported.

## Planning Your Installation

Planning for server

| IP | SSH PORT | SSH USER | SSH PASSWORD | ROOT PASSWORD | OS |
| ---- | ---- | ---- | ---- | ---- | ---- |
| 10.1.207.177 | 22022 | kafka | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.178 | 22022 | kafka | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.183 | 22022 | kafka | 123456 | root123 | CentOS Linux release 7.9.2009 |

**TIPS：** reference [Create User in Batch Automation](https://github.com/coolbeevip/ansible-playbook#create-user--group)

Planning for Kafka nodes

| IP | Kafka | Zookeeper | Java |
| ---- | ---- | ---- | ---- |
| 10.1.207.177 | ✓ | ✓ | ✓ |
| 10.1.207.178 | ✓ | ✓ | ✓ |
| 10.1.207.183 | ✓ | ✓ | ✓ |

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
- hosts: 10.1.207.177
  user: kafka

- hosts: 10.1.207.178
  user: kafka

- hosts: 10.1.207.183
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
  10.1.207.177:
    zookeeper_id: 177
    kafka_broker_id: 177
  10.1.207.178:
    zookeeper_id: 178
    kafka_broker_id: 178
  10.1.207.183:
    zookeeper_id: 183
    kafka_broker_id: 183
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
  -e ANSIBLE_SSH_HOSTS=10.1.207.177,10.1.207.178,10.1.207.183 \
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
* Configure kafka & zookeeper properties on all server
* Start zookeeper & kafka on all server

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/kafka/main.yml
```

**TIPS:** Because the Kafka & Java package will be uploaded to all servers (about 65MB+194MB) when the script is executed for the first time, so take longer to execute. The first installation on my local machine takes < 25 minutes(upload package taske about 5 minutes, others take about 20 minutes.)

**TIPS:** This script is only used for initial installation. Repeated execution of this command may receive a prompt of `Kafka has been installed, please uninstall and then reinstall`. At this time, you need to use `ansible all -m shell -a '~/kafka_uninstall.sh'` uninstalls.

If you see the following message, the installation is completed

```shell
TASK [Install Succeed] ********************************************************************************************************************************************************************************************************
ok: [10.1.207.177] => {
    "msg": "Install Succeed!"
}
```

#### Verify Kafka Cluster

Display Zookeeper PID

```shell
bash-5.0# ansible all -m shell -a '~/zookeeper.sh status'
10.1.207.183 | CHANGED | rc=0 >>
Zookeeper is Running as PID: 9477

10.1.207.177 | CHANGED | rc=0 >>
Zookeeper is Running as PID: 31545

10.1.207.178 | CHANGED | rc=0 >>
Zookeeper is Running as PID: 23435
```

Display Kafka PID

```shell
bash-5.0# ansible all -m shell -a '~/kafka.sh status'
10.1.207.177 | CHANGED | rc=0 >>
Kafka is Running as PID: 7569

10.1.207.183 | CHANGED | rc=0 >>
Kafka is Running as PID: 15901

10.1.207.178 | CHANGED | rc=0 >>
Kafka is Running as PID: 26639
```

Check Zookeeper successful return **imok**

```shell
bash-5.0# ansible all -m shell -a 'echo ruok | nc {{ inventory_hostname }} 2181; echo'
10.1.207.177 | CHANGED | rc=0 >>
imok

10.1.207.178 | CHANGED | rc=0 >>
imok

10.1.207.183 | CHANGED | rc=0 >>
imok
```

Execute benchmark command `benchmark-test.sh` on any Kafka node. Send and receive 5 million messages, message size 1KB

```shell
[kafka@oss-irms-177 bin]$ ~/benchmark-test.sh
STEP1/4: Create topic benchmark-topic
Created topic benchmark-topic.

STEP2/4: Producer performance test
41825 records sent, 8363.3 records/sec (7.98 MB/sec), 1178.2 ms avg latency, 2461.0 ms max latency.
50000 records sent, 8488.964346 records/sec (8.10 MB/sec), 1291.81 ms avg latency, 2461.00 ms max latency, 1165 ms 50th, 2115 ms 95th, 2325 ms 99th, 2460 ms 99.9th.

STEP3/4: Consumer performance test
start.time, end.time, data.consumed.in.MB, MB.sec, data.consumed.in.nMsg, nMsg.sec, rebalance.time.ms, fetch.time.ms, fetch.MB.sec, fetch.nMsg.sec
2021-12-09 16:05:17:113, 2021-12-09 16:05:21:696, 47.6837, 10.4045, 50000, 10909.8844, 1639037119044, -1639037114461, -0.0000, -0.0000

STEP4/4: Delete topic benchmark-topic
[kafka@oss-irms-177 bin]$ vi ~/benchmark-test.sh
[kafka@oss-irms-177 bin]$ ~/benchmark-test.sh

STEP1/4: Create topic benchmark-topic
Created topic benchmark-topic.

STEP2/4: Producer performance test
154401 records sent, 30874.0 records/sec (29.44 MB/sec), 621.5 ms avg latency, 1805.0 ms max latency.
80224 records sent, 16041.6 records/sec (15.30 MB/sec), 2028.6 ms avg latency, 3677.0 ms max latency.
209827 records sent, 41957.0 records/sec (40.01 MB/sec), 906.9 ms avg latency, 3408.0 ms max latency.
125145 records sent, 25029.0 records/sec (23.87 MB/sec), 845.3 ms avg latency, 4060.0 ms max latency.
128368 records sent, 25658.2 records/sec (24.47 MB/sec), 1480.5 ms avg latency, 4892.0 ms max latency.
63145 records sent, 12496.5 records/sec (11.92 MB/sec), 2219.6 ms avg latency, 5571.0 ms max latency.
104873 records sent, 18930.1 records/sec (18.05 MB/sec), 1434.3 ms avg latency, 6640.0 ms max latency.
109572 records sent, 21910.0 records/sec (20.90 MB/sec), 1982.4 ms avg latency, 5869.0 ms max latency.
131271 records sent, 25739.4 records/sec (24.55 MB/sec), 831.3 ms avg latency, 4349.0 ms max latency.
136261 records sent, 27246.8 records/sec (25.98 MB/sec), 1584.6 ms avg latency, 5031.0 ms max latency.
85629 records sent, 16803.2 records/sec (16.02 MB/sec), 1628.3 ms avg latency, 5089.0 ms max latency.
73866 records sent, 14612.5 records/sec (13.94 MB/sec), 1716.6 ms avg latency, 6803.0 ms max latency.
105543 records sent, 21100.2 records/sec (20.12 MB/sec), 2119.1 ms avg latency, 7117.0 ms max latency.
77355 records sent, 15471.0 records/sec (14.75 MB/sec), 1912.0 ms avg latency, 5498.0 ms max latency.
101397 records sent, 19846.7 records/sec (18.93 MB/sec), 1793.0 ms avg latency, 6986.0 ms max latency.
200199 records sent, 40031.8 records/sec (38.18 MB/sec), 1042.1 ms avg latency, 4969.0 ms max latency.
97188 records sent, 18189.8 records/sec (17.35 MB/sec), 1126.6 ms avg latency, 4933.0 ms max latency.
185294 records sent, 37058.8 records/sec (35.34 MB/sec), 1307.4 ms avg latency, 6376.0 ms max latency.
249676 records sent, 49925.2 records/sec (47.61 MB/sec), 659.5 ms avg latency, 2987.0 ms max latency.
385945 records sent, 77189.0 records/sec (73.61 MB/sec), 425.9 ms avg latency, 1282.0 ms max latency.
205702 records sent, 41132.2 records/sec (39.23 MB/sec), 792.5 ms avg latency, 3450.0 ms max latency.
174103 records sent, 34820.6 records/sec (33.21 MB/sec), 837.0 ms avg latency, 2014.0 ms max latency.
95234 records sent, 19043.0 records/sec (18.16 MB/sec), 1596.3 ms avg latency, 4084.0 ms max latency.
82637 records sent, 16524.1 records/sec (15.76 MB/sec), 1595.8 ms avg latency, 4584.0 ms max latency.
127413 records sent, 25477.5 records/sec (24.30 MB/sec), 1753.8 ms avg latency, 4631.0 ms max latency.
175331 records sent, 35066.2 records/sec (33.44 MB/sec), 998.8 ms avg latency, 3002.0 ms max latency.
412244 records sent, 82448.8 records/sec (78.63 MB/sec), 382.2 ms avg latency, 1177.0 ms max latency.
150998 records sent, 30139.3 records/sec (28.74 MB/sec), 986.8 ms avg latency, 3473.0 ms max latency.
85045 records sent, 16737.8 records/sec (15.96 MB/sec), 1717.5 ms avg latency, 4346.0 ms max latency.
81438 records sent, 15095.1 records/sec (14.40 MB/sec), 1898.2 ms avg latency, 5302.0 ms max latency.
121741 records sent, 24348.2 records/sec (23.22 MB/sec), 1556.8 ms avg latency, 6129.0 ms max latency.
117458 records sent, 23421.3 records/sec (22.34 MB/sec), 1278.0 ms avg latency, 3955.0 ms max latency.
116391 records sent, 23273.5 records/sec (22.20 MB/sec), 1509.0 ms avg latency, 3597.0 ms max latency.
88623 records sent, 17714.0 records/sec (16.89 MB/sec), 2034.1 ms avg latency, 4512.0 ms max latency.
106804 records sent, 20946.1 records/sec (19.98 MB/sec), 1494.2 ms avg latency, 3443.0 ms max latency.
5000000 records sent, 27473.131278 records/sec (26.20 MB/sec), 1176.38 ms avg latency, 7117.00 ms max latency, 521 ms 50th, 4694 ms 95th, 6050 ms 99th, 6916 ms 99.9th.

STEP3/4: Consumer performance test
start.time, end.time, data.consumed.in.MB, MB.sec, data.consumed.in.nMsg, nMsg.sec, rebalance.time.ms, fetch.time.ms, fetch.MB.sec, fetch.nMsg.sec
2021-12-09 16:09:05:331, 2021-12-09 16:09:39:913, 4768.3716, 137.8859, 5000000, 144583.8876, 1639037347941, -1639037313359, -0.0000, -0.0031

STEP4/4: Delete topic benchmark-topic
```

Producer report:

* 27473.131278 records/sec (26.20 MB/sec)
* 1176.38 ms avg latency）
* 7117.00 ms max latency）

Consumer report:

* data.consumed.in.MB 4768.3716 shows the size of all messages consumed
* MB.sec 137.8859 shows how much data transferred in megabytes per second (Throughput on size)
* data.consumed.in.nMsg 5000000 shows the count of the total messages consumed during this test
* nMsg.sec 144583.8876 shows how many messages were consumed in a second (Throughput on the count of messages)

## Common Maintenance Commands

#### Kafka

Start Kafka

```shell
bash-5.0# ansible all -m shell -a '~/kafka.sh start'
10.1.207.177 | CHANGED | rc=0 >>
Starting kafka

10.1.207.183 | CHANGED | rc=0 >>
Starting kafka

10.1.207.178 | CHANGED | rc=0 >>
Starting kafka
```

Stop Kafka

```shell
bash-5.0# ansible all -m shell -a '~/kafka.sh stop'
10.1.207.183 | CHANGED | rc=0 >>
Shutting down kafka

10.1.207.178 | CHANGED | rc=0 >>
Shutting down kafka

10.1.207.177 | CHANGED | rc=0 >>
Shutting down kafka
```

Display Kafka Process

```shell
bash-5.0# ansible all -m shell -a '~/kafka.sh status'
10.1.207.177 | CHANGED | rc=0 >>
Kafka is Running as PID: 19440

10.1.207.183 | CHANGED | rc=0 >>
Kafka is Running as PID: 20962

10.1.207.178 | CHANGED | rc=0 >>
Kafka is Running as PID: 25640
```

Create Topic

```shell
bash-5.0# ansible 10.1.207.177 -m shell -a 'kafka-topics.sh --create --topic testtopic --replication-factor 2 --partitions 6 --bootstrap-server 10.1.207.177:9003,10.1.207.178:9003,10.1.207.183:9003'
10.1.207.177 | CHANGED | rc=0 >>
Created topic testtopic.
```

#### Zookeeper

Start Zookeeper

```shell
bash-5.0# ansible all -m shell -a '~/zookeeper.sh start'
10.1.207.178 | CHANGED | rc=0 >>
Starting zookeeper

10.1.207.177 | CHANGED | rc=0 >>
Starting zookeeper

10.1.207.183 | CHANGED | rc=0 >>
Starting zookeeper
```

Stop Zookeeper

```shell
bash-5.0# ansible all -m shell -a '~/zookeeper.sh stop'
10.1.207.178 | CHANGED | rc=0 >>
Shutting down zookeeper

10.1.207.183 | CHANGED | rc=0 >>
Shutting down zookeeper

10.1.207.177 | CHANGED | rc=0 >>
Shutting down zookeeper
```

Display Zookeeper process

```shell
bash-5.0# ansible all -m shell -a '~/zookeeper.sh status'
10.1.207.177 | CHANGED | rc=0 >>
Zookeeper is Running as PID: 25589

10.1.207.183 | CHANGED | rc=0 >>
Zookeeper is Running as PID: 958

10.1.207.178 | CHANGED | rc=0 >>
Zookeeper is Running as PID: 26663
```

Output a list of variables that can be used to monitor the health of the Zookeeper cluster.

```shell
bash-5.0# ansible all -m shell -a 'echo mntr | nc {{ inventory_hostname }} 2181'
10.1.207.177 | CHANGED | rc=0 >>
zk_version	3.5.9-83df9301aa5c2a5d284a9940177808c01bc35cef, built on 01/06/2021 20:03 GMT
zk_avg_latency	0
zk_max_latency	0
zk_min_latency	0
zk_packets_received	2
zk_packets_sent	1
zk_num_alive_connections	1
zk_outstanding_requests	0
zk_server_state	leader
zk_znode_count	5
zk_watch_count	0
zk_ephemerals_count	0
zk_approximate_data_size	191
zk_open_file_descriptor_count	130
zk_max_file_descriptor_count	100000
zk_followers	2
zk_synced_followers	2
zk_pending_syncs	0
zk_last_proposal_size	-1
zk_max_proposal_size	-1
zk_min_proposal_size	-1

10.1.207.178 | CHANGED | rc=0 >>
zk_version	3.5.9-83df9301aa5c2a5d284a9940177808c01bc35cef, built on 01/06/2021 20:03 GMT
zk_avg_latency	0
zk_max_latency	0
zk_min_latency	0
zk_packets_received	2
zk_packets_sent	1
zk_num_alive_connections	1
zk_outstanding_requests	0
zk_server_state	follower
zk_znode_count	5
zk_watch_count	0
zk_ephemerals_count	0
zk_approximate_data_size	191
zk_open_file_descriptor_count	128
zk_max_file_descriptor_count	100000

10.1.207.183 | CHANGED | rc=0 >>
zk_version	3.5.9-83df9301aa5c2a5d284a9940177808c01bc35cef, built on 01/06/2021 20:03 GMT
zk_avg_latency	0
zk_max_latency	0
zk_min_latency	0
zk_packets_received	2
zk_packets_sent	1
zk_num_alive_connections	1
zk_outstanding_requests	0
zk_server_state	follower
zk_znode_count	5
zk_watch_count	0
zk_ephemerals_count	0
zk_approximate_data_size	191
zk_open_file_descriptor_count	128
zk_max_file_descriptor_count	100000
```

Display configuration.

```shell
bash-5.0# ansible all -m shell -a 'echo conf | nc {{ inventory_hostname }} 2181'
10.1.207.183 | CHANGED | rc=0 >>
clientPort=2181
secureClientPort=-1
dataDir=/data01/kafka/zookeeper_data/version-2
dataDirSize=67108880
dataLogDir=/data01/kafka/zookeeper_datalog/version-2
dataLogSize=5295
tickTime=2000
maxClientCnxns=0
minSessionTimeout=4000
maxSessionTimeout=40000
serverId=183
initLimit=10
syncLimit=5
electionAlg=3
electionPort=3888
quorumPort=2888
peerType=0
membership:
server.177=10.1.207.177:2888:3888:participant
server.178=10.1.207.178:2888:3888:participant
server.183=10.1.207.183:2888:3888:participant
version=0

10.1.207.177 | CHANGED | rc=0 >>
clientPort=2181
secureClientPort=-1
dataDir=/data01/kafka/zookeeper_data/version-2
dataDirSize=67108880
dataLogDir=/data01/kafka/zookeeper_datalog/version-2
dataLogSize=1715
tickTime=2000
maxClientCnxns=0
minSessionTimeout=4000
maxSessionTimeout=40000
serverId=177
initLimit=10
syncLimit=5
electionAlg=3
electionPort=3888
quorumPort=2888
peerType=0
membership:
server.177=10.1.207.177:2888:3888:participant
server.178=10.1.207.178:2888:3888:participant
server.183=10.1.207.183:2888:3888:participant
version=0

10.1.207.178 | CHANGED | rc=0 >>
clientPort=2181
secureClientPort=-1
dataDir=/data01/kafka/zookeeper_data/version-2
dataDirSize=67108880
dataLogDir=/data01/kafka/zookeeper_datalog/version-2
dataLogSize=2857
tickTime=2000
maxClientCnxns=0
minSessionTimeout=4000
maxSessionTimeout=40000
serverId=178
initLimit=10
syncLimit=5
electionAlg=3
electionPort=3888
quorumPort=2888
peerType=0
membership:
server.177=10.1.207.177:2888:3888:participant
server.178=10.1.207.178:2888:3888:participant
server.183=10.1.207.183:2888:3888:participant
version=0
```

List connection details to this server.

```shell
bash-5.0# ansible all -m shell -a 'echo cons | nc {{ inventory_hostname }} 2181'
10.1.207.177 | CHANGED | rc=0 >>
 /10.1.207.177:29094[0](queued=0,recved=1,sent=0)

10.1.207.178 | CHANGED | rc=0 >>
 /10.1.207.178:37550[0](queued=0,recved=1,sent=0)

10.1.207.183 | CHANGED | rc=0 >>
 /10.1.207.183:43808[0](queued=0,recved=1,sent=0)
```

Display outstanding sessions and ephemeral nodes.

```shell
bash-5.0# ansible all -m shell -a 'echo dump | nc {{ inventory_hostname }} 2181'
10.1.207.178 | CHANGED | rc=0 >>
SessionTracker dump:
Global Sessions(0):
ephemeral nodes dump:
Sessions with Ephemerals (0):
Connections dump:
Connections Sets (1)/(1):
1 expire at Thu Dec 09 10:27:03 CST 2021:
	ip: /10.1.207.178:37608 sessionId: 0x0

10.1.207.177 | CHANGED | rc=0 >>
SessionTracker dump:
Global Sessions(0):
ephemeral nodes dump:
Sessions with Ephemerals (0):
Connections dump:
Connections Sets (1)/(1):
1 expire at Thu Dec 09 10:25:25 CST 2021:
	ip: /10.1.207.177:29470 sessionId: 0x0

10.1.207.183 | CHANGED | rc=0 >>
SessionTracker dump:
Session Sets (0)/(0):
ephemeral nodes dump:
Sessions with Ephemerals (0):
Connections dump:
Connections Sets (1)/(1):
1 expire at Thu Dec 09 10:26:55 CST 2021:
	ip: /10.1.207.183:43966 sessionId: 0x0
```

Display environment settings.

```shell
bash-5.0# ansible all -m shell -a 'echo envi | nc {{ inventory_hostname }} 2181'
10.1.207.178 | CHANGED | rc=0 >>
Environment:
zookeeper.version=3.5.9-83df9301aa5c2a5d284a9940177808c01bc35cef, built on 01/06/2021 20:03 GMT
host.name=oss-irms-178
java.version=1.8.0_202
java.vendor=Oracle Corporation
java.home=/opt/kafka/kafka_2.12-2.6.3/jdk1.8.0_202/jre
java.class.path=.:/opt/kafka/kafka_2.12-2.6.3/jdk1.8.0_202/lib/dt.jar:/opt/kafka/kafka_2.12-2.6.3/jdk1.8.0_202/lib/tools.jar:/opt/kafka/kafka_2.12-2.6.3/jdk1.8.0_202/jre/lib::/opt/kafka/kafka_2.12-2.6.3/bin/../libs/activation-1.1.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/aopalliance-repackaged-2.6.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/argparse4j-0.7.0.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/audience-annotations-0.5.0.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/commons-cli-1.4.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/commons-lang3-3.8.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/connect-api-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/connect-basic-auth-extension-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/connect-file-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/connect-json-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/connect-mirror-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/connect-mirror-client-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/connect-runtime-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/connect-transforms-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/hk2-api-2.6.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/hk2-locator-2.6.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/hk2-utils-2.6.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-annotations-2.10.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-core-2.10.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-databind-2.10.5.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-dataformat-csv-2.10.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-datatype-jdk8-2.10.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-jaxrs-base-2.10.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-jaxrs-json-provider-2.10.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-module-jaxb-annotations-2.10.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-module-paranamer-2.10.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-module-scala_2.12-2.10.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jakarta.activation-api-1.2.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jakarta.annotation-api-1.3.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jakarta.inject-2.6.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jakarta.validation-api-2.0.2.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jakarta.ws.rs-api-2.1.6.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jakarta.xml.bind-api-2.3.2.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/javassist-3.25.0-GA.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/javassist-3.26.0-GA.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/javax.servlet-api-3.1.0.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/javax.ws.rs-api-2.1.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jaxb-api-2.3.0.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jersey-client-2.31.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jersey-common-2.31.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jersey-container-servlet-2.31.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jersey-container-servlet-core-2.31.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jersey-hk2-2.31.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jersey-media-jaxb-2.31.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jersey-server-2.31.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-client-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-continuation-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-http-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-io-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-security-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-server-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-servlet-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-servlets-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-util-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-util-ajax-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jopt-simple-5.0.4.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/kafka_2.12-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/kafka_2.12-2.6.3-sources.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/kafka-clients-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/kafka-log4j-appender-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/kafka-streams-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/kafka-streams-examples-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/kafka-streams-scala_2.12-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/kafka-streams-test-utils-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/kafka-tools-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/log4j-1.2.17.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/lz4-java-1.7.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/maven-artifact-3.8.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/metrics-core-2.2.0.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/netty-buffer-4.1.59.Final.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/netty-codec-4.1.59.Final.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/netty-common-4.1.59.Final.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/netty-handler-4.1.59.Final.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/netty-resolver-4.1.59.Final.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/netty-transport-4.1.59.Final.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/netty-transport-native-epoll-4.1.59.Final.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/netty-transport-native-unix-common-4.1.59.Final.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/osgi-resource-locator-1.0.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/paranamer-2.8.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/plexus-utils-3.2.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/reflections-0.9.12.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/rocksdbjni-5.18.4.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/scala-collection-compat_2.12-2.1.6.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/scala-java8-compat_2.12-0.9.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/scala-library-2.12.11.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/scala-logging_2.12-3.9.2.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/scala-reflect-2.12.11.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/slf4j-api-1.7.30.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/slf4j-log4j12-1.7.30.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/snappy-java-1.1.7.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/zookeeper-3.5.9.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/zookeeper-jute-3.5.9.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/zstd-jni-1.4.4-7.jar
java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
java.io.tmpdir=/tmp
java.compiler=<NA>
os.name=Linux
os.arch=amd64
os.version=3.10.0-957.el7.x86_64
user.name=kafka
user.home=/home/kafka
user.dir=/home/kafka
os.memory.free=1015MB
os.memory.max=1024MB
os.memory.total=1024MB

10.1.207.177 | CHANGED | rc=0 >>
Environment:
zookeeper.version=3.5.9-83df9301aa5c2a5d284a9940177808c01bc35cef, built on 01/06/2021 20:03 GMT
host.name=cluster-endpoint
java.version=1.8.0_202
java.vendor=Oracle Corporation
java.home=/opt/kafka/kafka_2.12-2.6.3/jdk1.8.0_202/jre
java.class.path=.:/opt/kafka/kafka_2.12-2.6.3/jdk1.8.0_202/lib/dt.jar:/opt/kafka/kafka_2.12-2.6.3/jdk1.8.0_202/lib/tools.jar:/opt/kafka/kafka_2.12-2.6.3/jdk1.8.0_202/jre/lib::/opt/kafka/kafka_2.12-2.6.3/bin/../libs/activation-1.1.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/aopalliance-repackaged-2.6.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/argparse4j-0.7.0.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/audience-annotations-0.5.0.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/commons-cli-1.4.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/commons-lang3-3.8.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/connect-api-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/connect-basic-auth-extension-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/connect-file-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/connect-json-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/connect-mirror-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/connect-mirror-client-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/connect-runtime-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/connect-transforms-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/hk2-api-2.6.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/hk2-locator-2.6.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/hk2-utils-2.6.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-annotations-2.10.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-core-2.10.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-databind-2.10.5.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-dataformat-csv-2.10.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-datatype-jdk8-2.10.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-jaxrs-base-2.10.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-jaxrs-json-provider-2.10.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-module-jaxb-annotations-2.10.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-module-paranamer-2.10.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-module-scala_2.12-2.10.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jakarta.activation-api-1.2.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jakarta.annotation-api-1.3.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jakarta.inject-2.6.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jakarta.validation-api-2.0.2.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jakarta.ws.rs-api-2.1.6.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jakarta.xml.bind-api-2.3.2.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/javassist-3.25.0-GA.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/javassist-3.26.0-GA.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/javax.servlet-api-3.1.0.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/javax.ws.rs-api-2.1.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jaxb-api-2.3.0.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jersey-client-2.31.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jersey-common-2.31.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jersey-container-servlet-2.31.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jersey-container-servlet-core-2.31.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jersey-hk2-2.31.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jersey-media-jaxb-2.31.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jersey-server-2.31.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-client-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-continuation-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-http-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-io-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-security-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-server-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-servlet-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-servlets-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-util-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-util-ajax-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jopt-simple-5.0.4.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/kafka_2.12-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/kafka_2.12-2.6.3-sources.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/kafka-clients-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/kafka-log4j-appender-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/kafka-streams-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/kafka-streams-examples-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/kafka-streams-scala_2.12-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/kafka-streams-test-utils-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/kafka-tools-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/log4j-1.2.17.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/lz4-java-1.7.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/maven-artifact-3.8.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/metrics-core-2.2.0.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/netty-buffer-4.1.59.Final.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/netty-codec-4.1.59.Final.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/netty-common-4.1.59.Final.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/netty-handler-4.1.59.Final.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/netty-resolver-4.1.59.Final.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/netty-transport-4.1.59.Final.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/netty-transport-native-epoll-4.1.59.Final.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/netty-transport-native-unix-common-4.1.59.Final.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/osgi-resource-locator-1.0.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/paranamer-2.8.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/plexus-utils-3.2.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/reflections-0.9.12.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/rocksdbjni-5.18.4.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/scala-collection-compat_2.12-2.1.6.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/scala-java8-compat_2.12-0.9.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/scala-library-2.12.11.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/scala-logging_2.12-3.9.2.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/scala-reflect-2.12.11.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/slf4j-api-1.7.30.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/slf4j-log4j12-1.7.30.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/snappy-java-1.1.7.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/zookeeper-3.5.9.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/zookeeper-jute-3.5.9.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/zstd-jni-1.4.4-7.jar
java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
java.io.tmpdir=/tmp
java.compiler=<NA>
os.name=Linux
os.arch=amd64
os.version=3.10.0-957.el7.x86_64
user.name=kafka
user.home=/home/kafka
user.dir=/home/kafka
os.memory.free=977MB
os.memory.max=1024MB
os.memory.total=1024MB

10.1.207.183 | CHANGED | rc=0 >>
Environment:
zookeeper.version=3.5.9-83df9301aa5c2a5d284a9940177808c01bc35cef, built on 01/06/2021 20:03 GMT
host.name=oss-irms-183
java.version=1.8.0_202
java.vendor=Oracle Corporation
java.home=/opt/kafka/kafka_2.12-2.6.3/jdk1.8.0_202/jre
java.class.path=.:/opt/kafka/kafka_2.12-2.6.3/jdk1.8.0_202/lib/dt.jar:/opt/kafka/kafka_2.12-2.6.3/jdk1.8.0_202/lib/tools.jar:/opt/kafka/kafka_2.12-2.6.3/jdk1.8.0_202/jre/lib::/opt/kafka/kafka_2.12-2.6.3/bin/../libs/activation-1.1.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/aopalliance-repackaged-2.6.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/argparse4j-0.7.0.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/audience-annotations-0.5.0.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/commons-cli-1.4.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/commons-lang3-3.8.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/connect-api-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/connect-basic-auth-extension-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/connect-file-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/connect-json-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/connect-mirror-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/connect-mirror-client-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/connect-runtime-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/connect-transforms-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/hk2-api-2.6.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/hk2-locator-2.6.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/hk2-utils-2.6.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-annotations-2.10.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-core-2.10.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-databind-2.10.5.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-dataformat-csv-2.10.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-datatype-jdk8-2.10.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-jaxrs-base-2.10.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-jaxrs-json-provider-2.10.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-module-jaxb-annotations-2.10.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-module-paranamer-2.10.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jackson-module-scala_2.12-2.10.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jakarta.activation-api-1.2.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jakarta.annotation-api-1.3.5.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jakarta.inject-2.6.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jakarta.validation-api-2.0.2.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jakarta.ws.rs-api-2.1.6.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jakarta.xml.bind-api-2.3.2.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/javassist-3.25.0-GA.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/javassist-3.26.0-GA.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/javax.servlet-api-3.1.0.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/javax.ws.rs-api-2.1.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jaxb-api-2.3.0.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jersey-client-2.31.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jersey-common-2.31.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jersey-container-servlet-2.31.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jersey-container-servlet-core-2.31.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jersey-hk2-2.31.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jersey-media-jaxb-2.31.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jersey-server-2.31.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-client-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-continuation-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-http-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-io-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-security-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-server-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-servlet-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-servlets-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-util-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jetty-util-ajax-9.4.39.v20210325.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/jopt-simple-5.0.4.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/kafka_2.12-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/kafka_2.12-2.6.3-sources.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/kafka-clients-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/kafka-log4j-appender-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/kafka-streams-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/kafka-streams-examples-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/kafka-streams-scala_2.12-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/kafka-streams-test-utils-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/kafka-tools-2.6.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/log4j-1.2.17.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/lz4-java-1.7.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/maven-artifact-3.8.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/metrics-core-2.2.0.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/netty-buffer-4.1.59.Final.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/netty-codec-4.1.59.Final.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/netty-common-4.1.59.Final.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/netty-handler-4.1.59.Final.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/netty-resolver-4.1.59.Final.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/netty-transport-4.1.59.Final.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/netty-transport-native-epoll-4.1.59.Final.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/netty-transport-native-unix-common-4.1.59.Final.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/osgi-resource-locator-1.0.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/paranamer-2.8.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/plexus-utils-3.2.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/reflections-0.9.12.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/rocksdbjni-5.18.4.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/scala-collection-compat_2.12-2.1.6.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/scala-java8-compat_2.12-0.9.1.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/scala-library-2.12.11.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/scala-logging_2.12-3.9.2.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/scala-reflect-2.12.11.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/slf4j-api-1.7.30.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/slf4j-log4j12-1.7.30.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/snappy-java-1.1.7.3.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/zookeeper-3.5.9.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/zookeeper-jute-3.5.9.jar:/opt/kafka/kafka_2.12-2.6.3/bin/../libs/zstd-jni-1.4.4-7.jar
java.library.path=/usr/java/packages/lib/amd64:/usr/lib64:/lib64:/lib:/usr/lib
java.io.tmpdir=/tmp
java.compiler=<NA>
os.name=Linux
os.arch=amd64
os.version=3.10.0-957.el7.x86_64
user.name=kafka
user.home=/home/kafka
user.dir=/home/kafka
os.memory.free=1012MB
os.memory.max=1024MB
os.memory.total=1024MB
```

Display the total size of snapshot and log files in bytes.

```shell
bash-5.0# ansible all -m shell -a 'echo dirs | nc {{ inventory_hostname }} 2181'
10.1.207.177 | CHANGED | rc=0 >>
datadir_size: 67108880
logdir_size: 1715

10.1.207.178 | CHANGED | rc=0 >>
datadir_size: 67108880
logdir_size: 2857

10.1.207.183 | CHANGED | rc=0 >>
datadir_size: 67108880
logdir_size: 5295
```

Check if server is running in read-only mode.

```shell
bash-5.0# ansible all -m shell -a 'echo isro | nc {{ inventory_hostname }} 2181'
10.1.207.178 | CHANGED | rc=0 >>
rw

10.1.207.177 | CHANGED | rc=0 >>
rw

10.1.207.183 | CHANGED | rc=0 >>
rw
```

## Q & A

#### Uninstall Kafka Cluster

Q: How to Uninstall Kafka Cluster

A: `~/kafka_uninstall.sh` script will **kill -9 Kafka and Zookeeper, delete program files and data files**

```shell
bash-5.0# ansible all -m shell -a '~/kafka_uninstall.sh'
10.1.207.177 | CHANGED | rc=0 >>
Delete Kafka unzip directory
Delete Kafka Data directory

10.1.207.178 | CHANGED | rc=0 >>
Delete Kafka unzip directory
Delete Kafka Data directory

10.1.207.183 | CHANGED | rc=0 >>
Delete Kafka unzip directory
Delete Kafka Data directory
```
