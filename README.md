# Install Middleware on Linux With Ansible

* Configuration Linux System
* Install Elasticsearch Cluster

## Prerequisites

Download ansible playbook

create directory `/opt/myansible` and git clone playbook

```shell
mkdir /opt/myansible && cd /opt/myansible
git clone git@github.com:coolbeevip/ansible-playbook.git
```

Start an Ansible container with volume `/opt/myansible/ansible-playbook`

```shell
docker run --name ansible --rm -it \
  -e ANSIBLE_SSH_HOSTS=10.1.207.180,10.1.207.181,10.1.207.182 \
  -e ANSIBLE_SSH_PORTS=22,22,22 \
  -e ANSIBLE_SSH_USERS=user,user,user \
  -e ANSIBLE_SSH_PASSS=userpass,userpass,userpass \
  -e ANSIBLE_SU_PASSS=rootpass,rootpass,rootpass \
  -v /opt/myansible/ansible-playbook:/ansible-playbook \
  coolbeevip/ansible:2.8.11-alpine \
  /bin/bash
```

Build your ansbile inventory with environment variables

* ANSIBLE_SSH_HOSTS: IP address
* ANSIBLE_SSH_PORTS: SSH port
* ANSIBLE_SSH_USERS: SSH username
* ANSIBLE_SSH_PASSS: SSH password
* ANSIBLE_SU_PASSS: su password

Use `ansible all -m ping` Try to connect to host

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

## Ansible Playbooks

### Use ansible-playbook to configuring the Linux

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

[more](system/README.md)

### Install Elasticsearch Cluster

## 安装 Elasticsearch

Download elasticsearch-7.8.1-linux-x86_64.tar.gz into /opt/myansible/ansible-playbook/packages directory

Start an Ansible container with 'elasticsearch' user

```shell
docker run --name ansible --rm -it \
  -e ANSIBLE_SSH_HOSTS=10.1.207.180,10.1.207.181,10.1.207.182 \
  -e ANSIBLE_SSH_PORTS=22022,22022,22022 \
  -e ANSIBLE_SSH_USERS=elasticsearch,elasticsearch,elasticsearch \
  -e ANSIBLE_SSH_PASSS=123456,123456,123456 \
  -e ANSIBLE_SU_PASSS=rootpass,rootpass,rootpass \
  -v /opt/myansible/ansible-playbook:/ansible-playbook \
  coolbeevip/ansible:2.8.11-alpine \
  /bin/bash
```

run this ansible command in ansible container

```shell
ansible-playbook -C /ansible-playbook/elasticsearch/main.yml
```

Star Elasticsearch Cluster

```shell
ansible all -m shell -a 'nohup /opt/elasticsearch/elasticsearch-7.8.1/bin/elasticsearch -p /tmp/elasticsearch-pid -d >/dev/null 2>&1 &'
```

Stop Elasticsearch Cluster

```shell
ansible all -m shell -a 'kill $(cat /tmp/elasticsearch-pid && echo)'
```

[more](elasticsearch/README.md)
