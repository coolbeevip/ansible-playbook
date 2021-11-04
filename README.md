# Install Middleware on Linux With Ansible | [中文](README_ZH.md)

Use [Ansible Docker](https://hub.docker.com/repository/docker/coolbeevip/ansible) to automate the installation of popular middleware

## Requirement

### Client Tools

Requires Docker runtime environment and pull Docker image `coolbeevip/ansible:2.8.11-alpine`

### Target Serve

For example, you have three servers

| IP | SSH PORT | SSH USER | SSH PASSWORD | ROOT PASSWORD |
| ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | 22022 | coolbee | 123456 | root123 |
| 10.1.207.181 | 22022 | coolbee | 123456 | root123 |
| 10.1.207.182 | 22022 | coolbee | 123456 | root123 |

## Run Ansible Tools

We will use the Ansible tool to connect to the above three servers and execute some simple commands

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

After execution, you can see the following information and enter the command line interactive mode

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

You can use the `ansible all -m ping` command to view the server connection status

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

Check time of three servers

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

Check the memory of the three servers

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

By now, you have mastered how to start the Ansible tool to connect to multiple target servers and execute some simple commands, Later, you can see the middleware deployment script written using Ansible playbook script, and you can experience it directly.

## Install Middleware use Ansible Playbook

* [Common Linux system configuration](./system/README_ZH.md)
* [Elasticsearch Cluster](./elasticsearch/README_ZH.md)
* [Redis Master-Slave Sentinel](./redis/README_ZH.md)
