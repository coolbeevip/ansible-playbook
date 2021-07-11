# ElasticSearch 安装

## 安装介质

下载安装脚本

```shell
git clone git@github.com:coolbeevip/ansible-playbook.git
```

将以下安装包放到 `ansible-playbook/packages` 目录下

* jdk-8u202-linux-x64.tar.gz
* elasticsearch-7.8.1-linux-x86_64.tar.gz

## 启动安装工具

假设本地安装的服务器为

* 10.1.207.180 用户名 root，密码 root
* 10.1.207.181 用户名 root，密码 root
* 10.1.207.182 用户名 root，密码 root

将 /ansible-playbook 目录映射到镜像内 /ansible-playbook 目录

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

测试服务器是否正常连通

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

执行以下脚本批量操作系统

```shell
ansible-playbook -C /ansible-playbook/system/main.yml
```

此脚本包含

* 禁用交换分区

查看修改结果

```shell
ansible all -m shell -a 'cat /etc/fstab'
ansible all -m shell -a 'cat /proc/swaps'
```

* 设置 vm.swappiness

查看修改结果

```shell
ansible all -m shell -a 'sysctl vm.swappiness'
```

* 设置文件句柄

查看修改结果

```shell
ansible all -m shell -a 'ulimit -a'
```

* 设置虚拟内存

查看修改结果

```shell
ansible all -m shell -a 'sysctl vm.max_map_count'
```

* 创建用户 elasticsearch，默认密码 123456

查看新增用户

```shell
ansible all -m shell -a 'id elasticsearch'
```

你可以使用以下命令批量修改 elasticsearch 密码

```shell
ansible all -m shell -a 'echo 123456 | passwd elasticsearch --stdin'
```

* 禁用防火墙

查看

```shell
ansible all -m shell -a 'systemctl status firewalld.service'
ansible all -m shell -a 'systemctl status iptables.service'
```

* 安装 java

查看

```shell
ansible all -m shell -a 'source /etc/profile && java -version'
```

* 安装 docker

查看

```shell
ansible all -m shell -a 'docker -version'
ansible all -m shell -a 'docker-compose -v'
```

## 附件

https://www.elastic.co/guide/en/elasticsearch/reference/7.8/system-config.html
