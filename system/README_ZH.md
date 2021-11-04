# 使用 Ansible Playbook 配置 Linux 系统

## 配置操作系统(main-os.yml)

此脚本可批量完成以下配置

* 安装常用工具 wget curl yum-utils lvm2 rsync git device-mapper-persistent-data; etc.
* 禁用系统交换空间(**通常 Elasticsearch, Redis 等这类使用堆外内存的中间件，建议禁用交换空间**)
* 禁用防火墙
* 设置交换优先级和虚拟内存 vm.swappiness vm.max_map_count
* 设置 Linux PAM limits nofile nproc

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/system/main-os.yml
```

查看主要系统参数

```shell
bash-5.0# ansible all -m shell -a 'ulimit -a && /sbin/sysctl vm.swappiness && /sbin/sysctl vm.max_map_count && /sbin/swapon -s'
```

检查防火墙服务状态

```shell
bash-5.0# ansible all -m shell -a 'systemctl status firewalld.service'
bash-5.0# ansible all -m shell -a 'systemctl status iptables.service'
```

## 安装 JAVA(main-java.yml)

在你的笔记本上创建 playbook 脚本存放目录

```shell
mkdir -p ~/my-docker-volume/ansible-playbook
```

下载 playbook 脚本

```shell
cd ~/my-docker-volume/ansible-playbook
git clone https://github.com/coolbeevip/ansible-playbook.git
```

下载 jdk 安装包到 `~/my-docker-volume/ansible-playbook/packages` 目录

执行安装命令

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/system/main-java.yml
```

检查 Java 版本

```shell
ansible all -m shell -a 'source /etc/profile && java -version'
```

## 安装 Docker 和 Docker Compose(main-docker.yml)

在你的笔记本上创建 playbook 脚本存放目录

```shell
mkdir -p ~/my-docker-volume/ansible-playbook
```

下载 playbook 脚本

```shell
cd ~/my-docker-volume/ansible-playbook
git clone https://github.com/coolbeevip/ansible-playbook.git
```

执行安装命令

```shell
ansible-playbook -C /ansible-playbook/system/main-docker.yml
```

检查 Docker 版本

```shell
ansible all -m shell -a 'docker -v'
ansible all -m shell -a 'docker-compose -v'
```
