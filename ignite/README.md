# Ansible Playbook 自动化安装 Ignite | [English](README.md)


## 安装计划

服务器规划

| IP地址 | SSH 端口 | SSH 用户名 | SSH 密码 | ROOT 密码 | OS |
| ---- | ---- | ---- | ---- | ---- | ---- |
| 10.1.207.180 | 22022 | ignite | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.181 | 22022 | ignite | 123456 | root123 | CentOS Linux release 7.9.2009 |
| 10.1.207.182 | 22022 | ignite | 123456 | root123 | CentOS Linux release 7.9.2009 |

**提示：** 可以参考[批量自动化创建用户](https://github.com/coolbeevip/ansible-playbook/blob/main/README_ZH.md#%E5%88%9B%E5%BB%BA%E7%94%A8%E6%88%B7%E5%92%8C%E7%BB%84)

集群节点规划

| IP | Ignite |
| ---- | ---- |
| 10.1.207.180 | ✓ |
| 10.1.207.181 | ✓ |
| 10.1.207.182 | ✓ |

节点安装路径

| 路径 | 描述 |
| ---- | ---- |
| /opt/ignite | 程序安装路径 |
| ~/ignite_uninstall.sh | 集群卸载脚本 |
| ~/ignite.sh | 集群启停脚本 |


## 下载安装包和 Playbook 脚本

在你的笔记本上创建 playbook 脚本存放目录

```shell
mkdir -p ~/my-docker-volume/ansible-playbook
```

下载 playbook 脚本

```shell
cd ~/my-docker-volume/ansible-playbook
git clone https://github.com/coolbeevip/ansible-playbook.git
```

下载 Ignite 安装包和扩展库到 `~/my-docker-volume/ansible-playbook/packages` 目录

```shell
wget -P ~/my-docker-volume/ansible-playbook/packages https://dlcdn.apache.org/ignite/2.11.1/apache-ignite-2.11.1-bin.zip --no-check-certificate
wget -P ~/my-docker-volume/ansible-playbook/packages https://repo1.maven.org/maven2/mysql/mysql-connector-java/8.0.20/mysql-connector-java-8.0.20.jar --no-check-certificate
```

## 配置安装脚本

#### main.yml

这个文件中主要定义了目标服务器的地址，登录用户名以及每个 ignite 节点的名称

```yaml
- hosts: 10.1.207.180
  user: ignite
  ...
- hosts: 10.1.207.181
  user: ignite
  ...
- hosts: 10.1.207.182
  user: ignite
  ...
```

#### vars_ignite.yml


## 开始安装

启动 Ansible 工具连接到目标服务器，并将 `~/my-docker-volume/ansible-playbook` 目录挂在到容器中。

**提示：** ANSIBLE_SSH_USERS，ANSIBLE_SSH_PASSS 配置成您之前在目标服务器上创建的用户名 `ignite` 和密码 `123456`

**提示：** ANSIBLE_SU_PASSS 为 root 用户的密码

```shell
docker run --name ansible --rm -it \
  -e ANSIBLE_SSH_HOSTS=10.1.207.180,10.1.207.181,10.1.207.182 \
  -e ANSIBLE_SSH_PORTS=22022,22022,22022 \
  -e ANSIBLE_SSH_USERS=ignite,ignite,ignite \
  -e ANSIBLE_SSH_PASSS=123456,123456,123456 \
  -e ANSIBLE_SU_PASSS=root123,root123,root123 \
  -v ~/my-docker-volume/ansible-playbook:/ansible-playbook \
  coolbeevip/ansible:2.8.11-alpine \
  /bin/bash
```

#### 安装

```shell
bash-5.0# ansible-playbook -C /ansible-playbook/ignite/main.yml
```

如果你看到如下信息，说明安装完成(必须 failed=0)

```shell
PLAY RECAP *****************************************************************************************************************************************************************************************************************************************************
10.1.207.180               : ok=43   changed=15   unreachable=0    failed=0    skipped=5    rescued=0    ignored=0
10.1.207.181               : ok=37   changed=9    unreachable=0    failed=0    skipped=11   rescued=0    ignored=0
10.1.207.182               : ok=37   changed=9    unreachable=0    failed=0    skipped=11   rescued=0    ignored=0s
```

#### 验证 Ignite 服务

查看进程ID

```shell
bash-5.0# ansible all -m shell -a '~/ignite.sh status'
10.1.207.181 | CHANGED | rc=0 >>
Elasticsearch is Running as PID: 19219
19281

10.1.207.180 | CHANGED | rc=0 >>
Elasticsearch is Running as PID: 17353
17386

10.1.207.182 | CHANGED | rc=0 >>
Elasticsearch is Running as PID: 30660
```

如果您在安装时启用了认证参数 `authenticationEnabled=true`，那么集群初始化时为 **INACTIVE** 状态，你需要任选一个节点执行以下命令激活集群

```shell
bash-5.0# ansible 10.1.207.180 -m shell -a 'echo y | control.sh --host 10.1.207.181 --port 11211 --user ignite --password ignite --set-state ACTIVE'
10.1.207.180 | CHANGED | rc=0 >>
Warning: the command will change state of cluster with name "956b92aed71-e2b27fe4-32ac-4500-8d60-5431ea3a4202" to ACTIVE.
Press 'y' to continue . . . Control utility [ver. 2.11.1#20211220-sha1:eae1147d]
2021 Copyright(C) Apache Software Foundation
User: ignite
Time: 2021-12-24T10:02:54.240
Warning: --password is insecure. Whenever possible, use interactive prompt for password (just discard --password option).
Command [SET-STATE] started
Arguments: --host 10.1.207.181 --port 11211 --user ignite --password ***** --set-state ACTIVE
--------------------------------------------------------------------------------
Cluster state changed to ACTIVE
Command [SET-STATE] finished with code: 0
Control utility has completed execution at: 2021-12-24T10:02:55.579
Execution time: 1339 ms
```

任选一个节点查看集群状态，执行以下命令并查看结果中是否显示 **Cluster is active**，如果显示

```shell
bash-5.0# ansible 10.1.207.180 -m shell -a 'control.sh --host 10.1.207.181 --port 11211 --user ignite --password ignite --state'
10.1.207.180 | CHANGED | rc=0 >>
Control utility [ver. 2.11.1#20211220-sha1:eae1147d]
2021 Copyright(C) Apache Software Foundation
User: ignite
Time: 2021-12-24T09:55:11.037
Warning: --password is insecure. Whenever possible, use interactive prompt for password (just discard --password option).
Command [STATE] started
Arguments: --host 10.1.207.181 --port 11211 --user ignite --password ***** --state
--------------------------------------------------------------------------------
Cluster  ID: 600845d1-e538-4c15-8541-97e6fe03d924
Cluster tag: vigilant_poincare
--------------------------------------------------------------------------------
Cluster is active
Command [STATE] finished with code: 0
Control utility has completed execution at: 2021-12-24T09:55:11.868
Execution time: 831 ms
```

登录任意节点，使用 SQLLine 连接到 ignite 集群，开启认证后默认账号密码是 **ignite**。

```shell
$ sqlline.sh -u "jdbc:ignite:thin://127.0.0.1:10800;user=ignite;password=ignite"
Enter username for jdbc:ignite:thin://127.0.0.1:10800;user=ignite;password=ignite:
Enter password for jdbc:ignite:thin://127.0.0.1:10800;user=ignite;password=ignite:
sqlline version 1.9.0
0: jdbc:ignite:thin://127.0.0.1:10800>
```

修改管理员用户 **ignite** 的密码为  **ignite123**

```sql
0: jdbc:ignite:thin://127.0.0.1:10800> ALTER USER "ignite" WITH PASSWORD 'ignite123';
No rows affected (0.19 seconds)

增加一个测试用户 **test**，密码为 **test123**

0: jdbc:ignite:thin://127.0.0.1:10800> CREATE USER "test" WITH PASSWORD 'test123';
No rows affected (0.334 seconds)
```

**提示：** 用户名用双引号扩起来表示大小写敏感，否则都会被认为是大写。

使用新建的 **test** 用户连接，**退出 SQLLine 使用 !quit 命令**

```shell
0: jdbc:ignite:thin://127.0.0.1:10800>!quit

$ sqlline.sh -u "jdbc:ignite:thin://127.0.0.1:10800;user=test;password=test123"
Enter username for jdbc:ignite:thin://127.0.0.1:10800;user=test;password=test123:
Enter password for jdbc:ignite:thin://127.0.0.1:10800;user=test;password=test123:
sqlline version 1.9.0
0: jdbc:ignite:thin://127.0.0.1:10800>
```

创建表

```shell
0: jdbc:ignite:thin://127.0.0.1:10800> CREATE TABLE city (id LONG PRIMARY KEY, name VARCHAR) WITH "template=replicated";
No rows affected (1.831 seconds)
```

插入数据

```shell
0: jdbc:ignite:thin://127.0.0.1:10800> INSERT INTO city (id, name) values (1,'北京');
1 row affected (0.485 seconds)
```

查询数据

```shell
0: jdbc:ignite:thin://127.0.0.1:10800> select * from city;
+----+------+
| ID | NAME |
+----+------+
| 1  | 北京   |
+----+------+
1 row selected (0.109 seconds)
```

* Ignite SQL[参考](https://ignite.apache.org/docs/latest/sql-reference/ddl)
* SQLLine 工具命令[参考](http://sqlline.sourceforge.net/#manual)

## 常用运维命令

启动服务

```shell
bash-5.0# ansible all -m shell -a '~/ignite.sh start'
10.1.207.181 | CHANGED | rc=0 >>
Starting Ignite

10.1.207.182 | CHANGED | rc=0 >>
Starting Ignite

10.1.207.180 | CHANGED | rc=0 >>
Starting Ignite
```

停止服务

```shell
bash-5.0# ansible all -m shell -a '~/ignite.sh stop'
10.1.207.181 | CHANGED | rc=0 >>
Shutting down Ignite

10.1.207.182 | CHANGED | rc=0 >>
Shutting down Ignite

10.1.207.180 | CHANGED | rc=0 >>
Shutting down Ignite
```

查看服务进程ID

```shell
bash-5.0# ansible all -m shell -a '~/ignite.sh status'
10.1.207.182 | CHANGED | rc=0 >>
Ignite is Running as PID: 5314

10.1.207.181 | CHANGED | rc=0 >>
Ignite is Running as PID: 19374

10.1.207.180 | CHANGED | rc=0 >>
Ignite is Running as PID: 32625
```

获取集群状态

```shell
$ control.sh --host 10.1.207.181 --port 11211 --user ignite --password ignite --state
Control utility [ver. 2.11.1#20211220-sha1:eae1147d]
2021 Copyright(C) Apache Software Foundation
User: ignite
Time: 2021-12-23T15:52:26.892
Command [STATE] started
Arguments: --host 10.1.207.181 --port 11211 --state
--------------------------------------------------------------------------------
Cluster  ID: 194216a5-f73b-4689-9347-1c4679204d7c
Cluster tag: dazzling_ellis
--------------------------------------------------------------------------------
Cluster is active
Command [STATE] finished with code: 0
Control utility has completed execution at: 2021-12-23T15:52:28.609
Execution time: 1717 msstate
```

获取基线中注册的节点

```shell
$ control.sh --host 10.1.207.181 --port 11211 --baseline
Control utility [ver. 2.11.1#20211220-sha1:eae1147d]
2021 Copyright(C) Apache Software Foundation
User: ignite
Time: 2021-12-23T15:54:55.829
Command [BASELINE] started
Arguments: --host 10.1.207.181 --port 11211 --baseline
--------------------------------------------------------------------------------
Cluster state: active
Current topology version: 3
Baseline auto adjustment enabled: softTimeout=0
Baseline auto-adjust are not scheduled

Current topology version: 3 (Coordinator: ConsistentId=10.1.207.182,127.0.0.1,172.17.0.1,172.18.0.1:48500, Address=172.17.0.1, Order=1)

Baseline nodes:
    ConsistentId=10.1.207.180,127.0.0.1,172.17.0.1,172.18.0.1,172.19.0.1:48500, Address=172.17.0.1, State=ONLINE, Order=2
    ConsistentId=10.1.207.181,127.0.0.1,172.17.0.1,172.18.0.1,172.19.0.1,172.19.0.1,172.20.0.1:48500, Address=172.17.0.1, State=ONLINE, Order=3
    ConsistentId=10.1.207.182,127.0.0.1,172.17.0.1,172.18.0.1:48500, Address=172.17.0.1, State=ONLINE, Order=1
--------------------------------------------------------------------------------
Number of baseline nodes: 3

Other nodes not found.
Command [BASELINE] finished with code: 0
Control utility has completed execution at: 2021-12-23T15:54:57.291
Execution time: 1462 ms
```

使用 Visor 查看缓存统计信息

```shell
$ ignitevisorcmd.sh -cfg=/data01/ignite/config/ignite-config.xml
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=128M; support was removed in 8.0
十二月 27, 2021 10:34:33 上午 org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions
信息: Loading XML bean definitions from URL [file:/data01/ignite/config/ignite-config.xml]
十二月 27, 2021 10:34:34 上午 org.springframework.context.support.AbstractApplicationContext prepareRefresh
信息: Refreshing org.springframework.context.support.GenericApplicationContext@3b07a0d6: startup date [Mon Dec 27 10:34:34 CST 2021]; root of context hierarchy
[10:34:36]    __________  ________________
[10:34:36]   /  _/ ___/ |/ /  _/_  __/ __/
[10:34:36]  _/ // (7 7    // /  / / / _/
[10:34:36] /___/\___/_/|_/___/ /_/ /___/
[10:34:36]
[10:34:36] ver. 2.11.1#20211220-sha1:eae1147d
[10:34:36] 2021 Copyright(C) Apache Software Foundation
[10:34:36]
[10:34:36] Ignite documentation: http://ignite.apache.org
[10:34:36]
[10:34:36] Quiet mode.
[10:34:36]   ^-- Logging to file '/data01/ignite/work/log/ignite-303135e2.0.log'
[10:34:36]   ^-- Logging by 'JavaLogger [quiet=true, config=null]'
[10:34:36]   ^-- To see **FULL** console log here add -DIGNITE_QUIET=false or "-v" to ignite.{sh|bat}
[10:34:36]
[10:34:36] OS: Linux 3.10.0-957.el7.x86_64 amd64
[10:34:36] VM information: Java(TM) SE Runtime Environment 1.8.0_202-b08 Oracle Corporation Java HotSpot(TM) 64-Bit Server VM 25.202-b08
[10:34:36] Please set system property '-Djava.net.preferIPv4Stack=true' to avoid possible problems in mixed environments.
[10:34:36] Configured plugins:
[10:34:36]   ^-- None
[10:34:36]
[10:34:36] Configured failure handler: [hnd=StopNodeOrHaltFailureHandler [tryStop=false, timeout=0, super=AbstractFailureHandler [ignoredFailureTypes=UnmodifiableSet [SYSTEM_WORKER_BLOCKED, SYSTEM_CRITICAL_OPERATION_TIMEOUT]]]]
[10:34:36] Message queue limit is set to 0 which may lead to potential OOMEs when running cache operations in FULL_ASYNC or PRIMARY_SYNC modes due to message queues growth on sender and receiver sides.
[10:34:37] Security status [authentication=on, sandbox=off, tls/ssl=off]
[10:34:38] Failed to send multicast message (is multicast enabled on this node?).
[10:34:40] Performance suggestions for grid  (fix if possible)
[10:34:40] To disable, set -DIGNITE_PERFORMANCE_SUGGESTIONS_DISABLED=true
[10:34:40]   ^-- Enable G1 Garbage Collector (add '-XX:+UseG1GC' to JVM options)
[10:34:40]   ^-- Set max direct memory size if getting 'OOME: Direct buffer memory' (add '-XX:MaxDirectMemorySize=<size>[g|G|m|M|k|K]' to JVM options)
[10:34:40]   ^-- Speed up flushing of dirty pages by OS (alter vm.dirty_expire_centisecs parameter by setting to 500)
[10:34:40] Refer to this page for more performance suggestions: https://apacheignite.readme.io/docs/jvm-and-system-tuning
[10:34:40]
[10:34:40] To start Console Management & Monitoring run ignitevisorcmd.{sh|bat}
[10:34:40]
[10:34:40] Ignite node started OK (id=303135e2)
[10:34:40] >>> Ignite cluster is in INACTIVE state (limited functionality available). Use control.(sh|bat) script or IgniteCluster.state(ClusterState.ACTIVE) to change the state.

Some useful commands:
+--------------------------------------------+
| Type 'top'    | to see full topology.      |
| Type 'node'   | to see node statistics.    |
| Type 'cache'  | to see cache statistics.   |
| Type 'tasks'  | to see tasks statistics.   |
| Type 'config' | to see node configuration. |
+--------------------------------------------+

Type 'help' to get help.

+----------------------------------------------------------------+
| Status               | Connected                               |
| Ignite instance name | <default>                               |
| Config path          | /data01/ignite/config/ignite-config.xml |
| Uptime               | 00:00:00                                |
+----------------------------------------------------------------+
___    _________________________ ________
__ |  / /____  _/__  ___/__  __ \___  __ \
__ | / /  __  /  _____ \ _  / / /__  /_/ /
__ |/ /  __/ /   ____/ / / /_/ / _  _, _/
_____/   /___/   /____/  \____/  /_/ |_|

ADMIN CONSOLE
2021 Copyright(C) Apache Software Foundation

+----------------------------------------------------------------+
| Status               | Connected                               |
| Ignite instance name | <default>                               |
| Config path          | /data01/ignite/config/ignite-config.xml |
| Uptime               | 00:00:00                                |
+----------------------------------------------------------------+

Type 'help' for more information.
Type 'open' to join the grid.
Type 'quit' to quit form Visor console.

visor> cache -a
Time of the snapshot: 2021-12-27 10:36:27
+==================================================================================================================================================================+
|       Name(@)        |    Mode     | Nodes | Total entries (Heap / Off-heap) | Primary entries (Heap / Off-heap) |   Hits    |  Misses   |   Reads   |  Writes   |
+==================================================================================================================================================================+
| ADDRESS_CACHE(@c0)   | PARTITIONED | 3     | 2080 (0 / 2080)                 | min: 661 (0 / 661)                | min: 0    | min: 0    | min: 0    | min: 0    |
|                      |             |       |                                 | avg: 693.33 (0.00 / 693.33)       | avg: 0.00 | avg: 0.00 | avg: 0.00 | avg: 0.00 |
|                      |             |       |                                 | max: 728 (0 / 728)                | max: 0    | max: 0    | max: 0    | max: 0    |
+----------------------+-------------+-------+---------------------------------+-----------------------------------+-----------+-----------+-----------+-----------+
| SQL_PUBLIC_CITY(@c1) | REPLICATED  | 3     | 0 (0 / 0)                       | min: 0 (0 / 0)                    | min: 0    | min: 0    | min: 0    | min: 0    |
|                      |             |       |                                 | avg: 0.00 (0.00 / 0.00)           | avg: 0.00 | avg: 0.00 | avg: 0.00 | avg: 0.00 |
|                      |             |       |                                 | max: 0 (0 / 0)                    | max: 0    | max: 0    | max: 0    | max: 0    |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------+

Cache 'ADDRESS_CACHE(@c0)':
+------------------------------------------------------+
| Name(@)                         | ADDRESS_CACHE(@c0) |
| Total entries (Heap / Off-heap) | 2080 (0 / 2080)    |
| Nodes                           | 3                  |
| Total size Min/Avg/Max          | 661 / 693.33 / 728 |
|   Heap size Min/Avg/Max         | 0 / 0.00 / 0       |
|   Off-heap size Min/Avg/Max     | 661 / 693.33 / 728 |
+------------------------------------------------------+

Nodes for: ADDRESS_CACHE(@c0)
+====================================================================================================================+
|       Node ID8(@), IP       | CPUs | Heap Used | CPU Load |   Up Time    |  Size (Primary / Backup)  | Hi/Mi/Rd/Wr |
+====================================================================================================================+
| C58242AF(@n0), 10.1.207.181 | 4    | 2.53 %    | 0.43 %   | 00:41:14.599 | Total: 728 (728 / 0)      | Hi: 0       |
|                             |      |           |          |              |   Heap: 0 (0 / <n/a>)     | Mi: 0       |
|                             |      |           |          |              |   Off-Heap: 728 (728 / 0) | Rd: 0       |
|                             |      |           |          |              |   Off-Heap Memory: <n/a>  | Wr: 0       |
+-----------------------------+------+-----------+----------+--------------+---------------------------+-------------+
| 7D07604D(@n1), 10.1.207.180 | 4    | 2.40 %    | 0.57 %   | 00:41:14.683 | Total: 661 (661 / 0)      | Hi: 0       |
|                             |      |           |          |              |   Heap: 0 (0 / <n/a>)     | Mi: 0       |
|                             |      |           |          |              |   Off-Heap: 661 (661 / 0) | Rd: 0       |
|                             |      |           |          |              |   Off-Heap Memory: <n/a>  | Wr: 0       |
+-----------------------------+------+-----------+----------+--------------+---------------------------+-------------+
| 0E95B5BE(@n2), 10.1.207.182 | 4    | 0.99 %    | 0.50 %   | 00:41:14.493 | Total: 691 (691 / 0)      | Hi: 0       |
|                             |      |           |          |              |   Heap: 0 (0 / <n/a>)     | Mi: 0       |
|                             |      |           |          |              |   Off-Heap: 691 (691 / 0) | Rd: 0       |
|                             |      |           |          |              |   Off-Heap Memory: <n/a>  | Wr: 0       |
+--------------------------------------------------------------------------------------------------------------------+
'Hi' - Number of cache hits.
'Mi' - Number of cache misses.
'Rd' - number of cache reads.
'Wr' - Number of cache writes.

Aggregated queries metrics:
  Minimum execution time: 00:00:00.000
  Maximum execution time: 00:00:00.000
  Average execution time: 00:00:00.000
  Total number of executions: 0
  Total number of failures:   0

Cache 'SQL_PUBLIC_CITY(@c1)':
+--------------------------------------------------------+
| Name(@)                         | SQL_PUBLIC_CITY(@c1) |
| Total entries (Heap / Off-heap) | 0 (0 / 0)            |
| Nodes                           | 3                    |
| Total size Min/Avg/Max          | 0 / 0.00 / 0         |
|   Heap size Min/Avg/Max         | 0 / 0.00 / 0         |
|   Off-heap size Min/Avg/Max     | 0 / 0.00 / 0         |
+--------------------------------------------------------+

Nodes for: SQL_PUBLIC_CITY(@c1)
+==================================================================================================================+
|       Node ID8(@), IP       | CPUs | Heap Used | CPU Load |   Up Time    | Size (Primary / Backup) | Hi/Mi/Rd/Wr |
+==================================================================================================================+
| C58242AF(@n0), 10.1.207.181 | 4    | 2.53 %    | 0.43 %   | 00:41:14.599 | Total: 0 (0 / 0)        | Hi: 0       |
|                             |      |           |          |              |   Heap: 0 (0 / <n/a>)   | Mi: 0       |
|                             |      |           |          |              |   Off-Heap: 0 (0 / 0)   | Rd: 0       |
|                             |      |           |          |              |   Off-Heap Memory: 0    | Wr: 0       |
+-----------------------------+------+-----------+----------+--------------+-------------------------+-------------+
| 7D07604D(@n1), 10.1.207.180 | 4    | 2.40 %    | 0.57 %   | 00:41:14.683 | Total: 0 (0 / 0)        | Hi: 0       |
|                             |      |           |          |              |   Heap: 0 (0 / <n/a>)   | Mi: 0       |
|                             |      |           |          |              |   Off-Heap: 0 (0 / 0)   | Rd: 0       |
|                             |      |           |          |              |   Off-Heap Memory: 0    | Wr: 0       |
+-----------------------------+------+-----------+----------+--------------+-------------------------+-------------+
| 0E95B5BE(@n2), 10.1.207.182 | 4    | 0.99 %    | 0.50 %   | 00:41:14.493 | Total: 0 (0 / 0)        | Hi: 0       |
|                             |      |           |          |              |   Heap: 0 (0 / <n/a>)   | Mi: 0       |
|                             |      |           |          |              |   Off-Heap: 0 (0 / 0)   | Rd: 0       |
|                             |      |           |          |              |   Off-Heap Memory: 0    | Wr: 0       |
+------------------------------------------------------------------------------------------------------------------+
'Hi' - Number of cache hits.
'Mi' - Number of cache misses.
'Rd' - number of cache reads.
'Wr' - Number of cache writes.

Aggregated queries metrics:
  Minimum execution time: 00:00:00.000
  Maximum execution time: 00:00:00.000
  Average execution time: 00:00:00.000
  Total number of executions: 0
  Total number of failures:   0
visor>
```

## Q & A

#### 如何彻底删除 Ignite 集群

A: `~/ignite_uninstall.sh` 脚本将 **kill Ignite 进程，删除程序文件和所有数据文件**

```shell
bash-5.0# ansible all -m shell -a '~/ignite_uninstall.sh'
```
