# Ansible 自动化安装脚本

## 安装介质

下载安装脚本

```shell
git clone git@github.com:coolbeevip/ansible-playbook.git
```

将以下安装包放到 `ansible-playbook/packages` 目录下

* jdk-8u202-linux-x64.tar.gz
* elasticsearch-7.8.1-linux-x86_64.tar.gz

## 启动安装工具

假设目标服务器为

* 10.1.207.180 用户名 root，密码 root
* 10.1.207.181 用户名 root，密码 root
* 10.1.207.182 用户名 root，密码 root

自动部署工具，将 /ansible-playbook 目录映射到镜像内 /ansible-playbook 目录

```shell
docker run --name ansible --rm -it \
  -e ANSIBLE_SSH_HOSTS=10.1.207.180,10.1.207.181,10.1.207.182 \
  -e ANSIBLE_SSH_PORTS=22022,22022,22022 \
  -e ANSIBLE_SSH_USERS=root,root,root \
  -e ANSIBLE_SSH_PASSS=root,root,root \
  -e ANSIBLE_SU_PASSS=root,root,root \
  -v /Users/zhanglei/mydocker/volume/ansible-playbook:/ansible-playbook \
  coolbeevip/ansible:2.8.11-alpine \
  /bin/bash
```

使用 `ansible all -m ping` 命令测试服务器是否正常连通

```shell
bash-5.0# ansible all -m ping
10.1.207.180 | SUCCESS => {
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
10.1.207.182 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
bash-5.0#
```

## 操作系统配置

执行以下脚本，批量配置系统

```shell
ansible-playbook -C /ansible-playbook/system/main.yml
```

此脚本批量执行以下操作

* 禁用防火墙
* 禁用交换分区
* 设置 vm.swappiness
* 设置文件句柄
* 设置虚拟内存
* 创建用户 elasticsearch，默认密码 123456
* 安装 java
* 安装 docker & docker-compose

[更多说明](system/README.md)

## 安装 Elasticsearch




## 附件

https://www.elastic.co/guide/en/elasticsearch/reference/7.8/system-config.html
