# Automate Install IoTDB Cluster with Ansible Playbook | [中文](README_ZH.md)

## Planning Your Installation

Planning for server


| IP | SSH PORT | SSH USER | SSH PASSWORD | ROOT PASSWORD | OS |
| ---- | ---- | ---- | ---- | ---- | ---- |
| 10.19.32.51 | 22022 | iotdb | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.19.32.52 | 22022 | iotdb | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.19.32.53 | 22022 | iotdb | 123456 | root123 | CentOS Linux release 7.9.2009 |

**TIPS：** reference [Create User in Batch Automation](https://github.com/coolbeevip/ansible-playbook#create-user--group)

Planning for IoTDB nodes

| IP | IoTDB |
| ---- | ---- |
| 10.19.32.51 | ✓ |
| 10.19.32.52 | ✓ |
| 10.19.32.53 | ✓ |

Planning for installation directory

| 路径 | 描述 |
| ---- | ---- |
| /opt/iotdb | Installation path of IoTDB, Java |
| ~/iotdb_uninstall.sh | IoTDB Cluster Uninstall script |
| ~/iotdb.sh | start & stop & status shell |
| /data01/iotdb/conf | configuration directory |
| /data01/iotdb/wal | data directory |
| /data01/iotdb/traacing | data directory |
| /data01/iotdb/udf | data directory |
| /data01/iotdb/index | data directory |
| /data01/iotdb/system | data directory |
| /data01/iotdb/data/data01 | data directory |
| /data01/iotdb/data/data02 | data directory |
| /data01/iotdb/logs | log directory |

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

Download IoTDB tar ball from the https://iotdb.apache.org/Download/ web site to `~/my-docker-volume/ansible-playbook/packages`

```shell
git clone git@github.com:apache/iotdb.git
cd iotdb
mvn clean package -DskipTests
cp cluster/target/iotdb-cluster-0.13.0-SNAPSHOT.zip ~/my-docker-volume/ansible-playbook/packages
```

Download JDK tar ball `jdk-8u202-linux-x64.tar.gz` to `~/my-docker-volume/ansible-playbook/packages`

## Configuration

> You can edit the following configuration files to modify the default parameters

#### main.yml

You can define the target server address here. In the following snippet, you can see that three servers and logged in using  `iotdb`.

```yaml
- hosts: 10.19.32.51
  user: iotdb
  ...
- hosts: 10.19.32.52
  user: iotdb
  ...
- hosts: 10.19.32.53
  user: iotdb
  ...
```

#### var_iotdb.yml

```shell
# 系统内核参数
limits_hard_nproc: '65535'
limits_soft_nproc: '65535'
limits_hard_nofile: '65535'
limits_soft_nofile: '65535'

# 安装介质
iotdb_tar: "apache-iotdb-0.13.0-SNAPSHOT-cluster-bin.zip"       # 安装包
iotdb_tar_unzip_dir: "apache-iotdb-0.13.0-SNAPSHOT-cluster-bin"    # 安装包解压后的目录名

# 安装目录
iotdb_home_dir: "/opt/iotdb"          # 源码包上传目录
iotdb_conf_dir: "/data01/iotdb/conf"  # 配置目录
iotdb_data_dirs: ["/data01/iotdb/data/data01","/data01/iotdb/data/data02"]  # 数据目录, 建议使用不同的磁盘
iotdb_wal_dir: "/data01/iotdb/wal"  # 数据目录
iotdb_tracing_dir: "/data01/iotdb/tracing"  # 数据目录
iotdb_udf_dir: "/data01/iotdb/udf"  # 数据目录
iotdb_index_dir: "/data01/iotdb/index"  # 数据目录
iotdb_system_dir: "/data01/iotdb/system"  # 数据目录
iotdb_logs_dir: "/data01/iotdb/logs"  # 日志目录

# 操作系统用户和组
os_user: "iotdb"                   # 操作系统用户名
os_group: "iotdb"                  # 操作系统组名

# 集群相关配置
iotdb:
  cluster:
    seed_nodes: 10.19.32.51:9003,10.19.32.52:9003,10.19.32.53:9003
    default_replica_num: 3
    internal_meta_port: 9003
    internal_data_port: 40010
  engine:
    rpc_port: 6667
    avg_series_point_number_threshold: 500000 # 每个序列最多缓存这么多点就会刷盘
    memtable_size_threshold: 1073741824 # 这个参数越大，写入速度快, 默认 1GB    
```

For more default configuration, please refer to the `iotdb/config/` directory

## Installation

Start the ansible container tool to connect to the target server, And mount directory `~/my-docker-volume/ansible-playbook` in the container.

**NOTICE:** ANSIBLE_SSH_USERS，ANSIBLE_SSH_PASSS is linux user iotdb and password

**NOTICE:** ANSIBLE_SU_PASSS is root password

```shell
docker run --name ansible --rm -it \
  -e ANSIBLE_SSH_HOSTS=10.19.32.51,10.19.32.52,10.19.32.53 \
  -e ANSIBLE_SSH_PORTS=22022,22022,22022 \
  -e ANSIBLE_SSH_USERS=iotdb,iotdb,iotdb \
  -e ANSIBLE_SSH_PASSS=123456,123456,123456 \
  -e ANSIBLE_SU_PASSS=root123,root123,root123 \
  -v /Users/zhanglei/mydocker/volume/ansible-playbook:/ansible-playbook \
  coolbeevip/ansible \
  /bin/bash
```

#### Install IoTDB Cluster

* Install JDK
* Install IoTDB Cluster
* Startup IoTDB Cluster

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/iotdb/main.yml
```

If you see the following message, the installation is completed

```shell
PLAY RECAP *****************************************************************************************************************************************************************************************************************************
10.19.32.51                : ok=29   changed=7    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
10.19.32.52                : ok=29   changed=7    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
10.19.32.53                : ok=28   changed=6    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0
```

**TIPS:** This script is only used for first installation. Repeated execution of this command may receive a prompt of `IoTDB has been installed, please uninstall and then reinstall`. At this time, you need to use `ansible all -m shell -a '~/iotdb_uninstall.sh'` uninstalls.

#### Verify the cluster

```shell
bash-5.0# ansible all -m shell -a 'start-cli.sh -h {{ inventory_hostname }} -p 6667 -u root -pw root -e "SHOW STORAGE GROUP"'
10.19.32.51 | CHANGED | rc=0 >>
+-------------+
|storage group|
+-------------+
+-------------+
Empty set.
It costs 0.290s
10.19.32.53 | CHANGED | rc=0 >>
+-------------+
|storage group|
+-------------+
+-------------+
Empty set.
It costs 0.491s
10.19.32.52 | CHANGED | rc=0 >>
+-------------+
|storage group|
+-------------+
+-------------+
Empty set.
It costs 0.512s
```

## Common Maintenance Commands

Start IoTDB

```shell
bash-5.0# ansible all -m shell -a '~/iotdb.sh start'
```

Stop IoTDB

```shell
bash-5.0# ansible all -m shell -a '~/iotdb.sh stop'
```

Restart IotDB

```shell
bash-5.0# ansible all -m shell -a '~/iotdb.sh restart'
```

Lookup IoTDB PID

```shell
bash-5.0# ansible all -m shell -a '~/iotdb.sh status'
```

## Q & A

#### Uninstall IoTDB Cluster

This script will **Stop IoTDB processes and delete programs and data directories**

```shell
bash-5.0# ansible all -m shell -a '~/iotdb_uninstall.sh'
```
