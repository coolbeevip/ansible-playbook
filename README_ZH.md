# 使用 Ansible 在 Linux 系统安装中间件 | [English](README.md)

使用 [Ansible Docker](https://hub.docker.com/repository/docker/coolbeevip/ansible) 工具自动化安装流行中间件

## 前提条件

### 客户端

客户端机器需要 Docker 运行环境，然后通过 `docker pull coolbeevip/ansible:2.8.11-alpine` 拉取 ansible 工具

### 目标服务器

假设您有以下三台服务器

| IP地址 | SSH 端口 | SSH 用户名 | SSH 密码 | ROOT 密码 |
| ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | 22022 | coolbee | 123456 | root123 |
| 10.1.207.181 | 22022 | coolbee | 123456 | root123 |
| 10.1.207.182 | 22022 | coolbee | 123456 | root123 |

## 启动 Ansible 工具

我们将使用 ansible 工具连接以上三个服务器，并执行一些简单的命令

```shell
docker run --name ansible --rm -it \
  -e ANSIBLE_SSH_HOSTS=10.1.207.180,10.1.207.181,10.1.207.182 \
  -e ANSIBLE_SSH_PORTS=22022,22022,22022 \
  -e ANSIBLE_SSH_USERS=coolbee,coolbee,coolbee \
  -e ANSIBLE_SSH_PASSS=123456,123456,123456 \
  -e ANSIBLE_SU_PASSS=root123,root123,root123 \
  coolbeevip/ansible:2.8.11-alpine \
  /bin/bash
```

执行后可看到如下信息，并进入命令交互模式

```shell
=============================
ansible 2.8.19
  config file = None
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.7.10 (default, Mar  2 2021, 09:06:08) [GCC 8.3.0]
=============================
Create /etc/ansible/ansible.cfg
Create /etc/ansible/hosts
add hosts 10.1.207.180
add hosts 10.1.207.181
add hosts 10.1.207.182
bash-5.0#
```

使用  `ansible all -m ping` 命令，看到如下信息表示 ansible 工具已经可以正常连接您规划的三个服务器

```shell
bash-5.0# ansible all -m ping
10.1.207.180 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
10.1.207.182 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
10.1.207.181 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
bash-5.0#
```

现在您就可以批量执行任意 shell 命令，例如查看三台服务器的时间

```shell
bash-5.0# ansible all -m shell -a "date"
10.1.207.181 | CHANGED | rc=0 >>
Wed Nov  3 15:04:14 CST 2021

10.1.207.182 | CHANGED | rc=0 >>
Wed Nov  3 15:03:21 CST 2021

10.1.207.180 | CHANGED | rc=0 >>
Wed Nov  3 15:08:00 CST 2021

bash-5.0#
```

查看三台服务器的剩余内存

```shell
bash-5.0# ansible all -m shell -a "free -h"
10.1.207.180 | CHANGED | rc=0 >>
              total        used        free      shared  buff/cache   available
Mem:            31G         25G        499M         65M        4.9G        5.0G
Swap:            0B          0B          0B

10.1.207.181 | CHANGED | rc=0 >>
              total        used        free      shared  buff/cache   available
Mem:            31G        9.0G         17G         25M        4.5G         21G
Swap:            0B          0B          0B

10.1.207.182 | CHANGED | rc=0 >>
              total        used        free      shared  buff/cache   available
Mem:            62G        9.3G         48G         40M        4.9G         52G
Swap:            0B          0B          0B

bash-5.0#
```

至此你已经掌握了如何启动 ansible 工具连接多台目标服务器，并执行了一些简单的命令，我们可以通过 playbook 脚本可以编排更复杂脚本。

## 使用 Ansible Playbook 脚本安装中间件

* [Linux 系统配置](./system/README_ZH.md)
* [Elasticsearch 三节点集群](./elasticsearch/README_ZH.md)
* [Redis 三节点主从哨兵模式集群](./redis/README_ZH.md)
