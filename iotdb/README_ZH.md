# Ansible Playbook 自动化安装 IoTDB 集群 | [English](README.md)

## 安装计划

服务器规划

| IP | SSH 端口 | SSH 用户名 | SSH 密码 | ROOT 密码 | OS |
| ---- | ---- | ---- | ---- | ---- | ---- |
| 10.19.32.51 | 22022 | iotdb | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.19.32.52 | 22022 | iotdb | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.19.32.53 | 22022 | iotdb | 123456 | root123 | CentOS Linux release 7.9.2009 |

**提示：** 可以参考[批量自动化创建用户](https://github.com/coolbeevip/ansible-playbook/blob/main/README_ZH.md#%E5%88%9B%E5%BB%BA%E7%94%A8%E6%88%B7%E5%92%8C%E7%BB%84)

集群节点规划

| IP | IoTDB |
| ---- | ---- |
| 10.19.32.51 | ✓ |
| 10.19.32.52 | ✓ |
| 10.19.32.53 | ✓ |

节点安装路径

| 路径 | 描述 |
| ---- | ---- |
| /opt/iotdb | 程序安装路径 |
| ~/iotdb_uninstall.sh | 集群卸载脚本 |
| ~/iotdb.sh | 启停脚本 |
| /data01/iotdb/conf | 配置脚本目录 |
| /data01/iotdb/wal | 数据目录 |
| /data01/iotdb/traacing | 数据目录 |
| /data01/iotdb/udf | 数据目录 |
| /data01/iotdb/index | 数据目录 |
| /data01/iotdb/system | 数据目录 |
| /data01/iotdb/data/data01 | 数据目录 |
| /data01/iotdb/data/data02 | 数据目录 |
| /data01/iotdb/logs | 日志目录 |

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

下载 iotdb 安装包到 `~/my-docker-volume/ansible-playbook/packages` 目录

```shell
git clone git@github.com:apache/iotdb.git
cd iotdb
mvn clean package -DskipTests
cp cluster/target/iotdb-cluster-0.13.0-SNAPSHOT.zip ~/my-docker-volume/ansible-playbook/packages
```

## 配置安装脚本

> 你可以编辑如下文件修改默认配置

#### main.yml

您可以在此处定义目标服务器，您可以看到这里定义了三个目标服务器，使用 `iotdb` 用户登录。

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

您可以在此处定义安装目录，版本，端口，默认主节点地址等配置信息。

```shell
# 系统内核参数
limits_hard_nproc: '65535'
limits_soft_nproc: '65535'
limits_hard_nofile: '65535'
limits_soft_nofile: '65535'

# 安装介质
iotdb_tar: "apache-iotdb-0.12.4-cluster-bin.zip"       # 安装包
iotdb_tar_unzip_dir: "apache-iotdb-0.12.4-cluster-bin"    # 安装包解压后的目录名

# 安装目录
iotdb_home_dir: "/opt/iotdb"          # 源码包上传目录
iotdb_conf_dir: "/data01/iotdb/conf"  # 配置目录
iotdb_data_dirs: ["/data01/iotdb/data/data01","/data01/iotdb/data/data02"]  # 数据目录
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

您可以在 `iotdb/config/` 目录下找到更多的默认配置文件

## 开始安装

启动 ansible 容器工具连接目标服务器，并将 `~/my-docker-volume/ansible-playbook` 目录挂在到容器中。

**提示：** ANSIBLE_SSH_USERS，ANSIBLE_SSH_PASSS 配置成您之前在目标服务器上创建的用户名 `iotdb` 和密码 `123456`

**提示：** ANSIBLE_SU_PASSS 为 root 用户的密码

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

bash-5.0#  
```

#### 安装集群

* 安装 JDK
* 按装 IoTDB
* 启动 IoTDB

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/iotdb/main.yml
```

如果你看到如下信息，说明安装完成

```shell
PLAY RECAP *****************************************************************************************************************************************************************************************************************************
10.19.32.51                : ok=29   changed=7    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
10.19.32.52                : ok=29   changed=7    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
10.19.32.53                : ok=28   changed=6    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0
```

**提示:** 此脚本只适合初始化安装，重复执行此命令可能会收到 `IoTDB has been installed, please uninstall and then reinstall` 提示，此时需要先要使用 `ansible all -m shell -a '~/iotdb_uninstall.sh'` 命令卸载之前的安装。

#### 验证集群

在每个节点上尝试登录并执行一个简单的 CLI 命令

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

## 常用运维命令

启动 IoTDB

```shell
bash-5.0# ansible all -m shell -a '~/iotdb.sh start'
```

停止 IoTDB

```shell
bash-5.0# ansible all -m shell -a '~/iotdb.sh stop'
```

重启 IotDB

```shell
bash-5.0# ansible all -m shell -a '~/iotdb.sh restart'
```

查看 IoTDB

```shell
bash-5.0# ansible all -m shell -a '~/iotdb.sh status'
```

## Q & A

#### 如何彻底删除 IoTDB

A: `~/iotdb_uninstall.sh` 脚本将 **kill IoTDB 删除程序文件和所有数据文件**

```shell
bash-5.0# ansible all -m shell -a '~/iotdb_uninstall.sh'
```
