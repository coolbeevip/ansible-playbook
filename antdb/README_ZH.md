# Ansible Playbook 安装 AntDB 集群 | [English](README.md)


## 目标服务器

假设您有以下三台服务器

| IP | SSH 端口 | SSH 用户名 | SSH 密码 | ROOT 密码 |
| ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | 22022 | antdb | 123456 | root123 |
| 10.1.207.181 | 22022 | antdb | 123456 | root123 |
| 10.1.207.182 | 22022 | antdb | 123456 | root123 |

分布式集群模块规划如下

| IP | MGR | GTM | Coordinator | DataNode |
| ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | Primary | Primary | | DN1-Primary, DN3-Secondary |
| 10.1.207.181 | | Secondary | CN1 | DN2-Primary, DN1-Secondary |
| 10.1.207.182 | Secondary | | CN2 | DN3-Primary, DN2-Secondary |

节点安装路径

| 路径 | 描述 |
| ---- | ---- |
| /opt/antdb | |
| ~/antdbb_uninstall.sh | AntDB 集群卸载脚本 |
| /data01/antdb/mgr | AntDB MGR 程序文件 |
| /data01/antdb/data | |
| /data01/antdb/tools | |
| /data01/antdb/app | AntDB 集群程序文件 |
| /data01/antdb/core | |

## 下载安装包和 Playbook 脚本

在客户机上创建 playbook 脚本存放目录

```shell
mkdir -p ~/my-docker-volume/ansible-playbook
```

下载 playbook 脚本

```shell
cd ~/my-docker-volume/ansible-playbook
git clone https://github.com/coolbeevip/ansible-playbook.git
```

下载 AntDB 安装包 `antdb.cluster-5.0.009be78c-centos7.9.rpm` 到 `~/my-docker-volume/ansible-playbook/packages` 目录

## 开始安装

启动 ansible 容器工具连接目标服务器，并将 `~/my-docker-volume/ansible-playbook` 目录挂载到容器中。

**提示：** ANSIBLE_SSH_USERS，ANSIBLE_SSH_PASSS 配置成您之前在目标服务器上创建的用户名 `antdb` 和密码 `123456`

**提示：** ANSIBLE_SU_PASSS 为 root 用户的密码

```shell
docker run --name ansible --rm -it \
  -e ANSIBLE_SSH_HOSTS=10.1.207.180,10.1.207.181,10.1.207.182 \
  -e ANSIBLE_SSH_PORTS=22022,22022,22022 \
  -e ANSIBLE_SSH_USERS=antdb,antdb,antdb \
  -e ANSIBLE_SSH_PASSS=123456,123456,123456 \
  -e ANSIBLE_SU_PASSS=root123,root123,root123 \
  -v /Users/zhanglei/mydocker/volume/ansible-playbook:/ansible-playbook \
  coolbeevip/ansible:2.8.11-alpine \
  /bin/bash  
```

#### 安装 Antdb 集群

执行安装脚本

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/antdb/main.yml
```

## 其他

推荐生产最小规模

分布式集群模块规划如下

| IP | MGR | GTM | Coordinator | DataNode |
| ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | Primary | Primary | CN-1 | Primary,Secondary |
| 10.1.207.181 | Secondary | Secondary | CN-2 | Primary,Secondary |
| 10.1.207.182 | | Secondary | CN-3 | Primary,Secondary |
| 10.1.207.183 | | | CN-4 | Primary,Secondary |
| 10.1.207.184 | | | CN-5 | Primary,Secondary |


```shell
bash-5.0# ansible 10.1.207.180 -m shell -a 'psql -p 16432 -d postgres -c "list host;"'
10.1.207.180 | CHANGED | rc=0 >>
   name   | user  | port  | protocol | agentport |   address    |      adbhome
----------+-------+-------+----------+-----------+--------------+-------------------
 antdb180 | antdb | 22022 | ssh      |     18432 | 10.1.207.180 | /data01/antdb/app
 antdb181 | antdb | 22022 | ssh      |     18432 | 10.1.207.181 | /data01/antdb/app
 antdb182 | antdb | 22022 | ssh      |     18432 | 10.1.207.182 | /data01/antdb/app
(3 rows)
```


```shell
bash-5.0# ansible 10.1.207.180 -m shell -a 'psql -p 16432 -d postgres -c "list node;"'
10.1.207.180 | CHANGED | rc=0 >>
     name      |   host   |        type        | mastername  | port  | sync_state |               path               | initialized | incluster | zone
---------------+----------+--------------------+-------------+-------+------------+----------------------------------+-------------+-----------+-------
 gtm_master    | antdb180 | gtmcoord master    |             | 16655 |            | /data01/antdb/data/gtm_master    | t           | t         | local
 gtm_slave_1   | antdb181 | gtmcoord slave     | gtm_master  | 16655 | sync       | /data01/antdb/data/gtm_slave_1   | t           | t         | local
 coordinator_1 | antdb181 | coordinator master |             | 15432 |            | /data01/antdb/data/coordinator_1 | t           | t         | local
 coordinator_2 | antdb182 | coordinator master |             | 15432 |            | /data01/antdb/data/coordinator_2 | t           | t         | local
 dn_master_1   | antdb180 | datanode master    |             | 14332 |            | /data01/antdb/data/dn_master_1   | t           | t         | local
 dn_master_2   | antdb181 | datanode master    |             | 14332 |            | /data01/antdb/data/dn_master_2   | t           | t         | local
 dn_master_3   | antdb182 | datanode master    |             | 14332 |            | /data01/antdb/data/dn_master_3   | t           | t         | local
 dn_slave_1    | antdb180 | datanode slave     | dn_master_1 | 14333 | sync       | /data01/antdb/data/dn_slave_1    | t           | t         | local
 dn_slave_2    | antdb181 | datanode slave     | dn_master_2 | 14333 | sync       | /data01/antdb/data/dn_slave_2    | t           | t         | local
 dn_slave_3    | antdb182 | datanode slave     | dn_master_3 | 14333 | sync       | /data01/antdb/data/dn_slave_3    | t           | t         | local
(10 rows)
```

```shell
bash-5.0# ansible 10.1.207.180 -m shell -a 'psql -p 16432 -d postgres -c "monitor all;"'
10.1.207.180 | CHANGED | rc=0 >>
   nodename    |      nodetype      | status | description |     host     | port  | recovery |           boot time           | nodezone
---------------+--------------------+--------+-------------+--------------+-------+----------+-------------------------------+----------
 gtm_master    | gtmcoord master    | t      | running     | 10.1.207.180 | 16655 | false    | 2021-11-29 15:35:37.239812+08 | local
 gtm_slave_1   | gtmcoord slave     | t      | running     | 10.1.207.181 | 16655 | true     | 2021-11-29 15:31:55.145147+08 | local
 coordinator_1 | coordinator master | t      | running     | 10.1.207.181 | 15432 | false    | 2021-11-29 15:32:02.253647+08 | local
 coordinator_2 | coordinator master | t      | running     | 10.1.207.182 | 15432 | false    | 2021-11-29 15:31:09.045575+08 | local
 dn_master_1   | datanode master    | t      | running     | 10.1.207.180 | 14332 | false    | 2021-11-29 15:35:57.878057+08 | local
 dn_master_2   | datanode master    | t      | running     | 10.1.207.181 | 14332 | false    | 2021-11-29 15:32:11.965045+08 | local
 dn_master_3   | datanode master    | t      | running     | 10.1.207.182 | 14332 | false    | 2021-11-29 15:31:18.718835+08 | local
 dn_slave_1    | datanode slave     | t      | running     | 10.1.207.180 | 14333 | true     | 2021-11-29 15:36:01.301736+08 | local
 dn_slave_2    | datanode slave     | t      | running     | 10.1.207.181 | 14333 | true     | 2021-11-29 15:32:18.017186+08 | local
 dn_slave_3    | datanode slave     | t      | running     | 10.1.207.182 | 14333 | true     | 2021-11-29 15:31:27.937421+08 | local
(10 rows)
```

连接 Coordinator 节点（默认端口 15432）成功

```shell
[antdb@oss-irms-182 ~]$ psql -h 10.1.207.181 -p 15432 –d postgres
psql: FATAL:  role "postgres" does not exist
[antdb@oss-irms-182 ~]$ psql -h 10.1.207.181 -p 15432 -d postgres
psql (5.0.1 based on PG 11.10)
Type "help" for help.

postgres=# \q
[antdb@oss-irms-182 ~]$ psql -h 10.1.207.182 -p 15432 -d postgres
psql (5.0.1 based on PG 11.10)
Type "help" for help.

postgres=#
```

```shell
# 添加主机信息
# add host <node_name>(port=22, protocol='ssh', adbhome='<antdb_app_dir>', address='<ip>', agentport=<agentport>, user='<user>');
#
# example:
# add host antdb180(port=22022, protocol='ssh', adbhome='/data01/antdb/app', address='10.1.207.180', agentport=18432,user='antdb');
# add host antdb181(port=22022, protocol='ssh', adbhome='/data01/antdb/app', address='10.1.207.181', agentport=18432,user='antdb');
# add host antdb182(port=22022, protocol='ssh', adbhome='/data01/antdb/app', address='10.1.207.182', agentport=18432,user='antdb');

# 添加 GTM 节点
# add gtmcoord master <gtm_master_name>(host=<node_name>, port=<antdb_gtm_port>, path='<antdb_data_dir>/<gtm_master_name>');
# add gtmcoord slave <gtm_slave_name> for <gtm_master_name>(host='node_name', port=antdb_gtm_port, path='<antdb_data_dir>/<gtm_slave_name>');
#
# example:
# add gtmcoord master gtm_master(host='antdb180', port=16655, path='/data01/antdb/data/gtm_master');
# add gtmcoord slave gtm_slave_1 for gtm_master(host='antdb181', port=16655, path='/data01/antdb/data/gtm_slave_1');


# 添加 Coordinator 节点
# add coordinator master <coordinator_master_name>(host='<node_name>', port=<antdb_coordinator_port>, path='<antdb_data_dir>/<coordinator_master_name>');
#
# example:
# add coordinator master cn_master_1(host='antdb181', port=15432, path='/data01/antdb/data/cn_master_1');
# add coordinator master cn_master_2(host='antdb182', port=15432, path='/data01/antdb/data/cn_master_2');



# 添加 DataNode 节点
# add datanode master <datanode_master_name>(host='<node_name>', port=<antdb_datanode_port>, path='<antdb_data_dir>/<datanode_master_name>');
# add datanode slave <datanode_slave_name> for <datanode_master_name>(host='<node_name>', port=<antdb_datanode_port>, path='<antdb_data_dir>/<datanode_slave_name>');
#
# example:
# add datanode master dn_master_1(host='antdb180', port=14332, path='/data01/antdb/data/dn_master_1');
# add datanode slave dn_slave_1 for dn_master_1(host='antdb181', port=14332, path='/data01/antdb/data/dn_slave_1');
# add datanode master dn_master_2(host='antdb181', port=14332, path='/data01/antdb/data/dn_master_2');
# add datanode slave dn_slave_2 for dn_master_2(host='antdb182', port=14332, path='/data01/antdb/data/dn_slave_2');
# add datanode master dn_master_3(host='antdb182', port=14332, path='/data01/antdb/data/dn_master_3');
# add datanode slave dn_slave_3 for dn_master_3(host='antdb180', port=14332, path='/data01/antdb/data/dn_slave_3');
```
