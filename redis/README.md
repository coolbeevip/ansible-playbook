# Install Redis Master-Slave Sentinel With Ansible Playbook | [中文](README_ZH.md)

Please create a `redis` user on the linux server before starting. This playbook script uses Redis source code compilation to install one master, two slaves, and three sentinels

## Download Redis Tar & Ansible Playbook scripts

Create a directory for Ansible Playbook scripts

```shell
mkdir -p ~/my-docker-volume/ansible-playbook
```

Download Ansible playbook script

```shell
cd ~/my-docker-volume/ansible-playbook
git clone https://github.com/coolbeevip/ansible-playbook.git
```

Download Redis tar ball from the redis.io web site to `~/my-docker-volume/ansible-playbook/packages`

```shell
wget -P ~/my-docker-volume/ansible-playbook/packages https://download.redis.io/releases/redis-6.2.6.tar.gz --no-check-certificate
```

## Configuration

Edit `redis/main.yml` script, You can define the target server address here. In the following snippet, you can see that three servers and logged in using  `redis`, The `10.1.207.180` is master.

```yaml
- hosts: 10.1.207.180
  user: redis
  ...
  vars:
    im_master: true
  tasks:
    ...
- hosts: 10.1.207.181
  user: redis
  ...
- hosts: 10.1.207.182
  user: redis
  ...
```

The installation directory, version, port, default master node address, and other configuration information are defined in the `var_redis.yml` file.

**NOTICE:** Please check if the variables redis_port, redis_password, redis_master_ip are what you want.

```environment
---
redis_tar: "redis-6.2.6.tar.gz"
redis_tar_unzip_dir: "redis-6.2.6"
redis_home_dir: "/opt/redis"
redis_log_dir: "/opt/redis/logs"
redis_data_dir: "/opt/redis/data"
redis_conf_dir: "/opt/redis/conf"
redis_user: "redis"
redis_group: "redis"
redis_password: "redis"
redis_network_host: "0.0.0.0"
redis_port: 7000
redis_maxmemory: 31457280
redis_maxclients: 1000
redis_master_ip: 10.1.207.180

redis_sentinel_port: 27000
redis_sentinel_master: "mymaster"
redis_sentinel_master_quorum: 2
redis_sentinel_down_after_milliseconds: 5000
redis_sentinel_failover_timeout: 10000
redis_sentinel_parallel_syncs: 2
```

You can find more default configuration in `redis/config/redis.conf.j2` and `redis/config/sentinel.conf.j2` files

## Install

Start the ansible container tool to connect to the target server, And mount directory `~/my-docker-volume/ansible-playbook` in the container.

**NOTICE:** ANSIBLE_SSH_USERS, ANSIBLE_SSH_PASSS is linux user redis username and password

```shell
docker run --name ansible --rm -it \
  -e ANSIBLE_SSH_HOSTS=10.1.207.180,10.1.207.181,10.1.207.182 \
  -e ANSIBLE_SSH_PORTS=22022,22022,22022 \
  -e ANSIBLE_SSH_USERS=redis,redis,redis \
  -e ANSIBLE_SSH_PASSS=123456,123456,123456 \
  -e ANSIBLE_SU_PASSS=root123,root123,root123 \
  -v ~/my-docker-volume/ansible-playbook:/ansible-playbook \
  coolbeevip/ansible:2.8.11-alpine \
  /bin/bash
bash-5.0#  
```

Run Ansible playbook scripts to install

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/redis/main.yml

PLAY [10.1.207.180] ****************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************
ok: [10.1.207.180]

TASK [include_tasks] ***************************************************************************************************************************
included: /ansible-playbook/redis/task_os.yml for 10.1.207.180

TASK [install packages] ************************************************************************************************************************
ok: [10.1.207.180]

TASK [create user redis] ***********************************************************************************************************************
ok: [10.1.207.180]

TASK [create user redis] ***********************************************************************************************************************
ok: [10.1.207.180]

TASK [include_tasks] ***************************************************************************************************************************
included: /ansible-playbook/redis/task_redis_upload.yml for 10.1.207.180

TASK [include vars] ****************************************************************************************************************************
ok: [10.1.207.180]

TASK [create redis home] ***********************************************************************************************************************
changed: [10.1.207.180]

TASK [check upload redis] **********************************************************************************************************************
ok: [10.1.207.180]

TASK [upload redis] ****************************************************************************************************************************
changed: [10.1.207.180]

TASK [check unzip redis] ***********************************************************************************************************************
ok: [10.1.207.180]

TASK [unzip redis] *****************************************************************************************************************************
changed: [10.1.207.180]

TASK [include_tasks] ***************************************************************************************************************************
included: /ansible-playbook/redis/task_redis_install.yml for 10.1.207.180

TASK [include vars] ****************************************************************************************************************************
ok: [10.1.207.180]

TASK [check make redis] ************************************************************************************************************************
ok: [10.1.207.180]

TASK [make redis] ******************************************************************************************************************************
changed: [10.1.207.180]

TASK [check install redis] *********************************************************************************************************************
ok: [10.1.207.180]

TASK [install redis] ***************************************************************************************************************************
changed: [10.1.207.180]

TASK [include_tasks] ***************************************************************************************************************************
included: /ansible-playbook/redis/task_redis_config.yml for 10.1.207.180

TASK [include vars] ****************************************************************************************************************************
ok: [10.1.207.180]

TASK [create others directories] ***************************************************************************************************************
changed: [10.1.207.180] => (item=/opt/redis/logs)
changed: [10.1.207.180] => (item=/opt/redis/data)
changed: [10.1.207.180] => (item=/opt/redis/conf)

TASK [copy redis configuration file] ***********************************************************************************************************
changed: [10.1.207.180]

TASK [copy sentinel configuration file] ********************************************************************************************************
changed: [10.1.207.180]

PLAY [10.1.207.181] ****************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************
ok: [10.1.207.181]

TASK [include_tasks] ***************************************************************************************************************************
included: /ansible-playbook/redis/task_os.yml for 10.1.207.181

TASK [install packages] ************************************************************************************************************************
ok: [10.1.207.181]

TASK [create user redis] ***********************************************************************************************************************
ok: [10.1.207.181]

TASK [create user redis] ***********************************************************************************************************************
ok: [10.1.207.181]

TASK [include_tasks] ***************************************************************************************************************************
included: /ansible-playbook/redis/task_redis_upload.yml for 10.1.207.181

TASK [include vars] ****************************************************************************************************************************
ok: [10.1.207.181]

TASK [create redis home] ***********************************************************************************************************************
changed: [10.1.207.181]

TASK [check upload redis] **********************************************************************************************************************
ok: [10.1.207.181]

TASK [upload redis] ****************************************************************************************************************************
changed: [10.1.207.181]

TASK [check unzip redis] ***********************************************************************************************************************
ok: [10.1.207.181]

TASK [unzip redis] *****************************************************************************************************************************
changed: [10.1.207.181]

TASK [include_tasks] ***************************************************************************************************************************
included: /ansible-playbook/redis/task_redis_install.yml for 10.1.207.181

TASK [include vars] ****************************************************************************************************************************
ok: [10.1.207.181]

TASK [check make redis] ************************************************************************************************************************
ok: [10.1.207.181]

TASK [make redis] ******************************************************************************************************************************
changed: [10.1.207.181]

TASK [check install redis] *********************************************************************************************************************
ok: [10.1.207.181]

TASK [install redis] ***************************************************************************************************************************
changed: [10.1.207.181]

TASK [include_tasks] ***************************************************************************************************************************
included: /ansible-playbook/redis/task_redis_config.yml for 10.1.207.181

TASK [include vars] ****************************************************************************************************************************
ok: [10.1.207.181]

TASK [create others directories] ***************************************************************************************************************
changed: [10.1.207.181] => (item=/opt/redis/logs)
changed: [10.1.207.181] => (item=/opt/redis/data)
changed: [10.1.207.181] => (item=/opt/redis/conf)

TASK [copy redis configuration file] ***********************************************************************************************************
changed: [10.1.207.181]

TASK [copy sentinel configuration file] ********************************************************************************************************
changed: [10.1.207.181]

PLAY [10.1.207.182] ****************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************
ok: [10.1.207.182]

TASK [include_tasks] ***************************************************************************************************************************
included: /ansible-playbook/redis/task_os.yml for 10.1.207.182

TASK [install packages] ************************************************************************************************************************
ok: [10.1.207.182]

TASK [create user redis] ***********************************************************************************************************************
ok: [10.1.207.182]

TASK [create user redis] ***********************************************************************************************************************
ok: [10.1.207.182]

TASK [include_tasks] ***************************************************************************************************************************
included: /ansible-playbook/redis/task_redis_upload.yml for 10.1.207.182

TASK [include vars] ****************************************************************************************************************************
ok: [10.1.207.182]

TASK [create redis home] ***********************************************************************************************************************
changed: [10.1.207.182]

TASK [check upload redis] **********************************************************************************************************************
ok: [10.1.207.182]

TASK [upload redis] ****************************************************************************************************************************
changed: [10.1.207.182]

TASK [check unzip redis] ***********************************************************************************************************************
ok: [10.1.207.182]

TASK [unzip redis] *****************************************************************************************************************************
changed: [10.1.207.182]

TASK [include_tasks] ***************************************************************************************************************************
included: /ansible-playbook/redis/task_redis_install.yml for 10.1.207.182

TASK [include vars] ****************************************************************************************************************************
ok: [10.1.207.182]

TASK [check make redis] ************************************************************************************************************************
ok: [10.1.207.182]

TASK [make redis] ******************************************************************************************************************************
changed: [10.1.207.182]

TASK [check install redis] *********************************************************************************************************************
ok: [10.1.207.182]

TASK [install redis] ***************************************************************************************************************************
changed: [10.1.207.182]

TASK [include_tasks] ***************************************************************************************************************************
included: /ansible-playbook/redis/task_redis_config.yml for 10.1.207.182

TASK [include vars] ****************************************************************************************************************************
ok: [10.1.207.182]

TASK [create others directories] ***************************************************************************************************************
changed: [10.1.207.182] => (item=/opt/redis/logs)
changed: [10.1.207.182] => (item=/opt/redis/data)
changed: [10.1.207.182] => (item=/opt/redis/conf)

TASK [copy redis configuration file] ***********************************************************************************************************
changed: [10.1.207.182]

TASK [copy sentinel configuration file] ********************************************************************************************************
changed: [10.1.207.182]

PLAY RECAP *************************************************************************************************************************************
10.1.207.180               : ok=23   changed=8    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
10.1.207.181               : ok=23   changed=8    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
10.1.207.182               : ok=23   changed=8    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

At this point, the installation is complete, and you can see the Redis node that has been configured in the installation directory of each target service.

You can use the following script to delete the temporary files generated during the installation

```shell
bash-5.0# ansible all -m shell -a "rm -rf /opt/redis/redis-6.2.6 /opt/redis/redis-6.2.6.tar.gz"
```

## Start & Stop

### Start Redis

The sentinel mode cluster needs to start each role in the order of `Master->Slave->Sentinel`, the startup sequence is as follows

start Redis node

```shell
bash-5.0# ansible all -m shell -a '/opt/redis/bin/redis-server /opt/redis/conf/redis.conf'
```

start Redis Sentinel node

```shell
bash-5.0# ansible all -m shell -a '/opt/redis/bin/redis-sentinel /opt/redis/conf/sentinel.conf'
```

Check the Redis process information of the target server, you can see that the Redis and sentinel processes of each node have been started

```shell
bash-5.0# ansible all -m shell -a 'ps -ef | grep redis'
10.1.207.180 | CHANGED | rc=0 >>
redis     27426     1  0 16:21 ?        00:00:00 /opt/redis/bin/redis-server 0.0.0.0:7000
redis     27545     1  0 16:21 ?        00:00:00 /opt/redis/bin/redis-sentinel 0.0.0.0:27000 [sentinel]
redis     27784 27782  0 16:22 pts/1    00:00:00 /bin/sh -c ps -ef | grep redis
redis     27786 27784  0 16:22 pts/1    00:00:00 grep redis

10.1.207.182 | CHANGED | rc=0 >>
redis     10404     1  0 16:16 ?        00:00:00 /opt/redis/bin/redis-server 0.0.0.0:7000
redis     10485     1  0 16:16 ?        00:00:00 /opt/redis/bin/redis-sentinel 0.0.0.0:27000 [sentinel]
redis     10653 10651  0 16:17 pts/0    00:00:00 /bin/sh -c ps -ef | grep redis
redis     10655 10653  0 16:17 pts/0    00:00:00 grep redis

10.1.207.181 | CHANGED | rc=0 >>
redis     22964     1  0 16:17 ?        00:00:00 /opt/redis/bin/redis-server 0.0.0.0:7000
redis     23045     1  0 16:17 ?        00:00:00 /opt/redis/bin/redis-sentinel 0.0.0.0:27000 [sentinel]
redis     23215 23214  0 16:18 pts/1    00:00:00 /bin/sh -c ps -ef | grep redis
redis     23217 23215  0 16:18 pts/1    00:00:00 grep redis
```

View the status of each node, you can see the master and slave node information

```shell
bash-5.0# ansible all -m shell -a '/opt/redis/bin/redis-cli -h 0.0.0.0 -p 7000 -a redis info Replication'

10.1.207.182 | CHANGED | rc=0 >>
# Replication
role:slave
master_host:10.1.207.180
master_port:7000
master_link_status:up
master_last_io_seconds_ago:1
master_sync_in_progress:0
slave_read_repl_offset:73780
slave_repl_offset:73780
slave_priority:100
slave_read_only:1
replica_announced:1
connected_slaves:0
master_failover_state:no-failover
master_replid:4dc6d9bc49043bc1bc8c495f88c470523af3b030
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:73780
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:73780Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.

10.1.207.181 | CHANGED | rc=0 >>
# Replication
role:slave
master_host:10.1.207.180
master_port:7000
master_link_status:up
master_last_io_seconds_ago:0
master_sync_in_progress:0
slave_read_repl_offset:73919
slave_repl_offset:73919
slave_priority:100
slave_read_only:1
replica_announced:1
connected_slaves:0
master_failover_state:no-failover
master_replid:4dc6d9bc49043bc1bc8c495f88c470523af3b030
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:73919
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:73919Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.

10.1.207.180 | CHANGED | rc=0 >>
# Replication
role:master
connected_slaves:2
slave0:ip=10.1.207.182,port=7000,state=online,offset=73502,lag=0
slave1:ip=10.1.207.181,port=7000,state=online,offset=73502,lag=1
master_failover_state:no-failover
master_replid:4dc6d9bc49043bc1bc8c495f88c470523af3b030
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:73780
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:73780Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.

bash-5.0#
```

### Stop Redis

stop Redis Sentinel node

```shell
bash-5.0# ansible all -m shell -a 'kill $(cat /opt/redis/logs/sentinel.pid && echo)'
```

stop Redis node

```shell
bash-5.0# ansible all -m shell -a 'kill $(cat /opt/redis/logs/redis.pid && echo)'
```
