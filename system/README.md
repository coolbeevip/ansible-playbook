# Configure Linux System with Ansible Playbook | [中文](README_ZH.md)

## Configuration OS(main-os.yml)

This script can complete the following configurations in batches

* Install yum packages wget curl yum-utils lvm2 rsync git device-mapper-persistent-data; etc.
* Disable Swap
* Disable firewalld
* Set vm.swappiness=1
* Set vm.max_map_count=262144
* ulimit -s 1048576
* ulimit -n 65535
* ulimit -u 65535

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/system/main-os.yml
```

Check the key parameters of each system

```shell
bash-5.0# ansible all -m shell -a 'ulimit -a && /sbin/sysctl vm.swappiness && /sbin/sysctl vm.max_map_count && /sbin/swapon -s'
```

Check Firewall status of each system

```shell
bash-5.0# ansible all -m shell -a 'systemctl status firewalld.service'
bash-5.0# ansible all -m shell -a 'systemctl status iptables.service'
```

## Install Java(main-java.yml)

Create a directory for Ansible Playbook scripts

```shell
mkdir -p ~/my-docker-volume/ansible-playbook
```

Download Ansible Playbook scripts

```shell
cd ~/my-docker-volume/ansible-playbook
git clone https://github.com/coolbeevip/ansible-playbook.git
```

Download JDK jdk-8u202-linux-x64.tar.gz to `~/my-docker-volume/ansible-playbook/packages` directory

Run Ansible playbook scripts to install

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/system/main-java.yml
```

Check Java version

```shell
ansible all -m shell -a 'source /etc/profile && java -version'
```

## Install Docker and Docker Compose(main-docker.yml)

Create a directory for Ansible Playbook scripts

```shell
mkdir -p ~/my-docker-volume/ansible-playbook
```

Download Ansible Playbook scripts

```shell
cd ~/my-docker-volume/ansible-playbook
git clone https://github.com/coolbeevip/ansible-playbook.git
```

Run Ansible playbook scripts to install

```shell
ansible-playbook -C /ansible-playbook/system/main-docker.yml
```

Check Docker version

```shell
ansible all -m shell -a 'docker -v'
ansible all -m shell -a 'docker-compose -v'
```
