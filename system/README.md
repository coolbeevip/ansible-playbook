# 安装系统

## 禁用交换分区

查看修改结果

```shell
ansible all -m shell -a 'cat /etc/fstab'
ansible all -m shell -a 'cat /proc/swaps'
```

## 设置 vm.swappiness

查看修改结果

```shell
ansible all -m shell -a 'sysctl vm.swappiness'
```

## 设置文件句柄

查看修改结果

```shell
ansible all -m shell -a 'ulimit -a'
```

## 设置虚拟内存

查看修改结果

```shell
ansible all -m shell -a 'sysctl vm.max_map_count'
```

## 创建用户 elasticsearch，默认密码 123456

查看新增用户

```shell
ansible all -m shell -a 'id elasticsearch'
```

你可以使用以下命令批量修改 elasticsearch 密码

```shell
ansible all -m shell -a 'echo 123456 | passwd elasticsearch --stdin'
```

## 禁用防火墙

查看

```shell
ansible all -m shell -a 'systemctl status firewalld.service'
ansible all -m shell -a 'systemctl status iptables.service'
```

## 安装 java

查看

```shell
ansible all -m shell -a 'source /etc/profile && java -version'
```

## 安装 docker

查看

```shell
ansible all -m shell -a 'docker -v'
ansible all -m shell -a 'docker-compose -v'
```
