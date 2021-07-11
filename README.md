# ElasticSearch 安装

## 服务器

10.1.207.180
10.1.207.181
10.1.207.182

## Ansible

```shell
docker run --name ansible --rm -it \
  -e ANSIBLE_SSH_HOSTS=10.1.207.180,10.1.207.181,10.1.207.182 \
  -e ANSIBLE_SSH_PORTS=22022,22022,22022 \
  -e ANSIBLE_SSH_USERS=root,root,root \
  -e ANSIBLE_SSH_PASSS=root,root,root \
  -e ANSIBLE_SU_PASSS=xxx,xxx,xxx \
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

##  操作系统配置

> https://www.elastic.co/guide/en/elasticsearch/reference/7.8/system-config.html



ansible-playbook -C /ansible-playbook/system/main.yml



###  Disable all swap files

临时禁用交换

```shell
ansible all -m shell -a 'swapoff -a'
```

永久禁用交换 `/etc/fstab` 注释包含 `swap` 的行

```
ansible all -m shell -a 'sed -i "s/^.*swap/#&/g"  /etc/fstab'
```

查看包含 swap 的行是否已经被注释

```shell
ansible all -m shell -a 'cat /etc/fstab'
```

查看是否还有使用的交换设备

```shell
ansible all -m shell -a 'cat /proc/swaps'
```

### Configure swappiness

> 减少了内核交换的倾向，在正常情况下不应该导致交换，同时仍然允许整个系统在紧急情况下交换

设置

```shell
ansible all -m shell -a 'echo "vm.swappiness=1" >> /etc/sysctl.conf'
ansible all -m shell -a 'sysctl -p'
```

查看是否生效

```shell
ansible all -m shell -a 'sysctl vm.swappiness'
```

###  Increase file descriptors

设置文件句柄

```shell
ansible all -m shell -a 'echo "* soft nofile 65535" >> /etc/security/limits.conf'
ansible all -m shell -a 'echo "* hard nofile 65535" >> /etc/security/limits.conf'
ansible all -m shell -a 'echo "* soft nproc 65535" >> /etc/security/limits.conf'
ansible all -m shell -a 'echo "* hard nproc 65535" >> /etc/security/limits.conf'
```

查看是否生效

```shell
ansible all -m shell -a 'ulimit -a'
```

###  Virtual memory

设置虚拟内存

```shell
ansible all -m shell -a 'echo "vm.max_map_count=262144" >> /etc/sysctl.conf'
ansible all -m shell -a 'sysctl -p'
```

查看是否生效

```shell
ansible all -m shell -a 'sysctl vm.max_map_count'
```

### 安装常用工具

```shell
ansible all -m shell -a 'yum install -y rsync wget curl'
```

### 创建用户

```shell
ansible all -m shell -a 'adduser elasticsearch'
ansible all -m shell -a 'echo 123456 | passwd elasticsearch --stdin'
```

## 复制脚本文件

将部署介质和脚本文件复制到 ansible 的外部卷目录下

* jdk-8u202-linux-x64.tar.gz
* main.yml
* java_install.yml

```
/Users/zhanglei/mydocker/volume/ansible-playbook
```

### Install Java

> 将 jdk-8u202-linux-x64.tar.gz 文件复制到

```shell
ansible-playbook -C /ansible-playbook/java_install.yml
```

检查是否安装成功

```shell
ansible all -m shell -a 'source /etc/profile && java -version'
```
