# Ansible Playbook 安装 AntDB 集群 | [English](README.md)


## 目标服务器

假设您有以下三台服务器

| IP | SSH 端口 | SSH 用户名 | SSH 密码 | ROOT 密码 |
| ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | 22022 | antdb | 123456 | root123 |
| 10.1.207.181 | 22022 | antdb | 123456 | root123 |
| 10.1.207.182 | 22022 | antdb | 123456 | root123 |

分布式集群模块规划如下

| IP | MGR | GTM | DataNode | Coordinator |
| ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | Primary | | Primary | |
| 10.1.207.181 | Secondary | Primary | Secondary | Primary |
| 10.1.207.182 | | Secondary | | Secondary |

节点安装路径

| 路径 | 描述 |
| ---- | ---- |
| /opt/antdb | |
| ~/antdbb_uninstall.sh | AntDB 集群卸载脚本 |
| /data01/antdb/mgr | AntDB MGR 程序文件 |
| /data01/antdb/data | |
| /data01/antdb/tools | |
| /data01/antdb/app | AntDB 集群程序文件 |
| /data01/antdb/core | |

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

下载 AntDB 安装包 `antdb.cluster-5.0.009be78c-centos7.9.rpm` 到 `~/my-docker-volume/ansible-playbook/packages` 目录

## 开始安装

启动 ansible 容器工具连接目标服务器，并将 `~/my-docker-volume/ansible-playbook` 目录挂载到容器中。

**提示：** ANSIBLE_SSH_USERS，ANSIBLE_SSH_PASSS 配置成您之前在目标服务器上创建的用户名 `antdb` 和密码 `123456`

**提示：** ANSIBLE_SU_PASSS 为 root 用户的密码

```shell
docker run --name ansible --rm -it \
  -e ANSIBLE_SSH_HOSTS=10.1.207.180,10.1.207.181,10.1.207.182 \
  -e ANSIBLE_SSH_PORTS=22022,22022,22022 \
  -e ANSIBLE_SSH_USERS=antdb,antdb,antdb \
  -e ANSIBLE_SSH_PASSS=123456,123456,123456 \
  -e ANSIBLE_SU_PASSS=root123,root123,root123 \
  -v /Users/zhanglei/mydocker/volume/ansible-playbook:/ansible-playbook \
  coolbeevip/ansible:2.8.11-alpine \
  /bin/bash  
```

#### 安装三节点 Antdb 集群

执行安装脚本

> 此命令会配置操作系统内核参数、自动上传安装介质到三个目标服务器，设置 MySQL 环境变量，初始化 MySQL 数据库，设置 MySQL root 密码，启动 MySQL 服务

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/antdb/main.yml
```
