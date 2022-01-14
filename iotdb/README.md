# Ansible Playbook 自动化安装 IoTDB 集群 | [English](README.md)

请先在目标服务器创建 `iotdb` 用户

## 安装计划

服务器规划

| IP | SSH 端口 | SSH 用户名 | SSH 密码 | ROOT 密码 | OS |
| ---- | ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | 22022 | iotdb | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.181 | 22022 | iotdb | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.182 | 22022 | iotdb | 123456 | root123 | CentOS Linux release 7.9.2009 |

**提示：** 可以参考[批量自动化创建用户](https://github.com/coolbeevip/ansible-playbook/blob/main/README_ZH.md#%E5%88%9B%E5%BB%BA%E7%94%A8%E6%88%B7%E5%92%8C%E7%BB%84)

集群节点规划

| IP | IoTDB | Grafana |
| ---- | ---- | ---- |
| 10.1.207.180 | ✓ |  |
| 10.1.207.181 | ✓ |  |
| 10.1.207.182 | ✓ |  |

节点安装路径

| 路径 | 描述 |
| ---- | ---- |
| /opt/iotdb | 源代码上传路径 |
| ~/iotdb_uninstall.sh | 集群卸载脚本 |
| ~/iotdb.sh | 启停脚本 |

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
wget -P ~/my-docker-volume/ansible-playbook/packages https://dlcdn.apache.org/iotdb/0.12.4/apache-iotdb-0.12.4-cluster-bin.zip --no-check-certificate
```

## 配置安装脚本

> 你可以编辑如下文件修改默认配置

#### main.yml

您可以在此处定义目标服务器，您可以看到这里定义了三个目标服务器，使用 `iotdb` 用户登录。

```yaml
- hosts: 10.1.207.180
  user: iotdb
  ...
- hosts: 10.1.207.181
  user: iotdb
  ...
- hosts: 10.1.207.182
  user: iotdb
  ...
```

#### var_iotdb.yml

您可以在此处定义安装目录，版本，端口，默认主节点地址等配置信息。

```shell
# 源码包
redis_tar: "apache-iotdb-0.12.4-cluster-bin.zip"       # 源码包文件名
redis_tar_unzip_dir: "apache-iotdb-0.12.4-cluster"    # 源码包解压后的目录名

# 安装目录
redis_home_dir: "/opt/iotdb"          # 源码包上传目录
redis_bin_dir: "/data01/iotdb/bin"    # 编译后程序文件
redis_log_dir: "/data01/iotdb/logs"   # 运行日志，PID 文件
redis_data_dir: "/data01/iotdb/data"  # 数据文件
redis_conf_dir: "/data01/iotdb/conf"  # 配置文件

# 操作系统用户和组
os_user: "iotdb"                   # 操作系统用户名
os_group: "iotdb"                  # 操作系统组名
```

您可以在 `iotdb/config/iotdb-cluster.properties.j2` 和 `iotdb/config/iotdb-engine.properties.j2` 文件中找到更多的默认配置

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
  -e ANSIBLE_SU_PASSS=Ncxdjr0lxGu1,Ncxdjr0lxGu1,Ncxdjr0lxGu1 \
  -v /Users/zhanglei/mydocker/volume/ansible-playbook:/ansible-playbook \
  coolbeevip/ansible \
  /bin/bash

bash-5.0#  
```

#### 安装集群

这个脚本将自动化完成如下操作：

* 上传 IoTDB 安装包到每个服务器
* 配置 IoTDB
* 启动 IoTDB


```shell
bash-5.0# ansible-playbook -C /ansible-playbook/iotdb/main-install.yml
```

如果你看到如下信息，说明安装完成

```shell
PLAY RECAP *****************************************************************************************************************************************************************************************************************************
10.19.32.51                : ok=29   changed=7    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
10.19.32.52                : ok=29   changed=7    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
10.19.32.53                : ok=28   changed=6    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0
```

**提示:** 因为第一次执行脚本时，会上传 Redis 源码包（约 2.5MB）到所有服务器并在服务器上编译（每服务器约 1分钟）。在我本地环境首次安装大概耗时 5 分钟

**提示:** 此脚本只适合初始化安装，重复执行此命令可能会收到 `Redis has been installed, please uninstall and then reinstall` 提示，此时需要先要使用 `ansible all -m shell -a '~/redis_uninstall.sh'` 命令卸载之前的安装。

#### 验证集群

在每个节点上尝试登录并执行一个简单的 CLi 命令

```shell
bash-5.0# ansible all -m shell -a 'start-cli.sh -h {{ inventory_hostname }} -p 6667 -u root -pw root -e "SHOW STORAGE GROUP"'
```

start-cli.sh -h 10.19.32.52 -p 6667 -u root -pw root -e "SHOW STORAGE GROUP"

## 常用运维命令

**提示：** 哨兵模式集群需要按 `Master->Slave->Sentinel` 顺序启动各个节点

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
