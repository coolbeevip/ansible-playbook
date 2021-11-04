### Use Ansible Playbook to configuration the Linux System

Download jdk-8u202-linux-x64.tar.gz into /opt/myansible/ansible-playbook/packages directory

run this ansible command in ansible container

```shell
ansible-playbook -C /ansible-playbook/system/main.yml
```

* Install wget curl yum-utils lvm2 rsync git device-mapper-persistent-data; etc.
* Install Java
* Install Docker
* Disable swapping
* Configure swappiness
* Increase file descriptors
* Ensure sufficient threads
* Ensure sufficient virtual memory
* TCP retransmission timeout
* Add user elasticsearch

# Installing and configuring Linux System

Execute this command to complete the configuration

```shell
ansible-playbook -C /ansible-playbook/system/main.yml
```

## Check Result

Check disable swapping

```shell
ansible all -m shell -a 'cat /etc/fstab'
ansible all -m shell -a 'cat /proc/swaps'
```

Check vm.swappiness

```shell
ansible all -m shell -a 'sysctl vm.swappiness'
```

Check file descriptors

```shell
ansible all -m shell -a 'ulimit -a'
```

Check virtual memory

```shell
ansible all -m shell -a 'sysctl vm.max_map_count'
```

Check user elasticsearch，default password is `123456`

```shell
ansible all -m shell -a 'id elasticsearch'
```

**Tip:** You can modify the default password

```shell
ansible all -m shell -a 'echo 123456 | passwd elasticsearch --stdin'
```

Check firewalld & iptables

```shell
ansible all -m shell -a 'systemctl status firewalld.service'
ansible all -m shell -a 'systemctl status iptables.service'
```

Check Java

```shell
ansible all -m shell -a 'source /etc/profile && java -version'
```

Check Docker

```shell
ansible all -m shell -a 'docker -v'
ansible all -m shell -a 'docker-compose -v'
```
