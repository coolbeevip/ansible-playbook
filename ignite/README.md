# Ansible Playbook 自动化安装 Ignite | [English](README.md)


## 安装计划

服务器规划

| IP地址 | SSH 端口 | SSH 用户名 | SSH 密码 | ROOT 密码 | OS |
| ---- | ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | 22022 | ignite | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.181 | 22022 | ignite | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.182 | 22022 | ignite | 123456 | root123 | CentOS Linux release 7.9.2009 |

**提示：** 可以参考[批量自动化创建用户](https://github.com/coolbeevip/ansible-playbook/blob/main/README_ZH.md#%E5%88%9B%E5%BB%BA%E7%94%A8%E6%88%B7%E5%92%8C%E7%BB%84)

集群节点规划

| IP | Ignite |
| ---- | ---- |
| 10.1.207.180 | |
| 10.1.207.181 | |
| 10.1.207.182 | |

节点安装路径

| 路径 | 描述 |
| ---- | ---- |
| /opt/ignite | 程序安装路径 |
| ~/ignite_uninstall.sh | 集群卸载脚本 |
| ~/ignite.sh | 集群启停脚本 |


## 下载安装包和 Playbook 脚本

在你的笔记本上创建 playbook 脚本存放目录

```shell
mkdir -p ~/my-docker-volume/ansible-playbook
```

下载 playbook 脚本

```shell
cd ~/my-docker-volume/ansible-playbook
git clone https://github.com/coolbeevip/ansible-playbook.git
```

下载 Kibana 和 Metricbeat 安装包到 `~/my-docker-volume/ansible-playbook/packages` 目录

```shell
wget -P ~/my-docker-volume/ansible-playbook/packages https://dlcdn.apache.org/ignite/2.11.1/apache-ignite-2.11.1-bin.zip --no-check-certificate
```

## 配置安装脚本

#### main.yml

这个文件中主要定义了目标服务器的地址，登录用户名以及每个 elasticsearch 节点的名称

```yaml
- hosts: 10.1.207.180
  user: ignite
  ...
- hosts: 10.1.207.181
  user: ignite
  ...
- hosts: 10.1.207.182
  user: ignite
  ...
```

#### vars_ignite.yml


## 开始安装

启动 Ansible 工具连接到目标服务器，并将 `~/my-docker-volume/ansible-playbook` 目录挂在到容器中。

**提示：** ANSIBLE_SSH_USERS，ANSIBLE_SSH_PASSS 配置成您之前在目标服务器上创建的用户名 `ignite` 和密码 `123456`

**提示：** ANSIBLE_SU_PASSS 为 root 用户的密码

```shell
docker run --name ansible --rm -it \
  -e ANSIBLE_SSH_HOSTS=10.1.207.180,10.1.207.181,10.1.207.182 \
  -e ANSIBLE_SSH_PORTS=22022,22022,22022 \
  -e ANSIBLE_SSH_USERS=ignite,ignite,ignite \
  -e ANSIBLE_SSH_PASSS=123456,123456,123456 \
  -e ANSIBLE_SU_PASSS=root123,root123,root123 \
  -v ~/my-docker-volume/ansible-playbook:/ansible-playbook \
  coolbeevip/ansible:2.8.11-alpine \
  /bin/bash
```

#### 安装

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/ignite/main.yml
```

如果你看到如下信息，说明安装完成(必须 failed=0)

```shell
PLAY RECAP *****************************************************************************************************************************************************************************************************************************************************
10.1.207.180               : ok=43   changed=15   unreachable=0    failed=0    skipped=5    rescued=0    ignored=0
10.1.207.181               : ok=37   changed=9    unreachable=0    failed=0    skipped=11   rescued=0    ignored=0
10.1.207.182               : ok=37   changed=9    unreachable=0    failed=0    skipped=11   rescued=0    ignored=0s
```

#### 验证 Ignite 服务

查看进程ID

```shell
bash-5.0# ansible all -m shell -a '~/ignite.sh status'
10.1.207.181 | CHANGED | rc=0 >>
Elasticsearch is Running as PID: 19219
19281

10.1.207.180 | CHANGED | rc=0 >>
Elasticsearch is Running as PID: 17353
17386

10.1.207.182 | CHANGED | rc=0 >>
Elasticsearch is Running as PID: 30660
```

## 常用运维命令

启动服务

```shell
bash-5.0# ansible all -m shell -a '~/ignite.sh start'
```

停止服务

```shell
bash-5.0# ansible all -m shell -a '~/ignite.sh stop'
```

查看服务进程ID

```shell
bash-5.0# ansible all -m shell -a '~/ignite.sh status'
```

## Q & A

#### 如何彻底删除 Ignite 集群

A: `~/ignite_uninstall.sh` 脚本将 **kill Ignite 进程，删除程序文件和所有数据文件**

```shell
bash-5.0# ansible all -m shell -a '~/ignite_uninstall.sh'
```
