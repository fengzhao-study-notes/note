**�ٷ��ĵ�**��
> http://zookeeper.apache.org/doc/r3.5.5/

**��������zookeeper��Ⱥʵ��**
##### 1������
ϵͳ��Fedora release 30 (Thirty)
Ӧ�ð汾��
- openjdk-1.8.0_212
- zookeeper-3.5.5

##### 2����װ
```shell
# ȷ���Ѿ���װjava����
[root@anatronics ~]# java -version
openjdk version "1.8.0_212"
OpenJDK Runtime Environment (build 1.8.0_212-b04)
OpenJDK 64-Bit Server VM (build 25.212-b04, mixed mode)

# ��ȡ�����ư�
[root@anatronics ~]# wget https://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.5.5/apache-zookeeper-3.5.5-bin.tar.gz

# ��ѹ������������
[root@anatronics ~]# tar xf apache-zookeeper-3.5.5-bin.tar.gz -C /apps/
[root@anatronics apps]# ln -sv apache-zookeeper-3.5.5-bin/ zookeeper/

# ���û���
[root@anatronics apps]# cat /etc/profile.d/zookeeper.sh 
export PATH="$PATH:/apps/zookeeper/bin"
```
##### 3����������ļ�
**�������Ŀ¼**
```shell
[root@anatronics ~]# mkdir /data/zookeeper/zoo{1..3}
```
**�����ļ�**
```shell
[root@anatronics conf]# pwd
/apps/zookeeper/conf

[root@anatronics conf]# cp zoo_sample.cfg zoo1.cfg
[root@anatronics conf]# cp zoo_sample.cfg zoo2.cfg
[root@anatronics conf]# cp zoo_sample.cfg zoo3.cfg

# �ڵ�1�����ļ���Ϣ
[root@anatronics conf]# grep -v "^#" zoo1.cfg 
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/data/zookeeper/zoo1
dataLogDir=/data/zookeeper/log/zoo1
clientPort=2181
server.1=127.0.0.1:2888:3888
server.2=127.0.0.1:2889:3889
server.3=127.0.0.1:2890:3890

# �ڵ�2�����ļ���Ϣ
[root@anatronics conf]# grep -v "^#" zoo2.cfg 
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/data/zookeeper/zoo2
dataLogDir=/data/zookeeper/log/zoo2
clientPort=2181
server.1=127.0.0.1:2888:3888
server.2=127.0.0.1:2889:3889
server.3=127.0.0.1:2890:3890

# �ڵ�3�����ļ���Ϣ
[root@anatronics conf]# grep -v "^#" zoo3.cfg 
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/data/zookeeper/zoo3
dataLogDir=/data/zookeeper/log/zoo3
clientPort=2181
server.1=127.0.0.1:2888:3888
server.2=127.0.0.1:2889:3889
server.3=127.0.0.1:2890:3890
```
**�ڸ��ڵ��ӦdataDir�д���myid�ļ�������������ʶ��**
```shell
[root@anatronics ~]# echo 1 > /data/zookeeper/zoo1/myid 
[root@anatronics ~]# echo 2 > /data/zookeeper/zoo2/myid 
[root@anatronics ~]# echo 3 > /data/zookeeper/zoo3/myid 
```
##### 4����������
**����**
```shell
# �л��������ļ�Ŀ¼��
[root@anatronics conf]# pwd
/apps/zookeeper/conf

[root@anatronics conf]# zkServer.sh start zoo1.cfg
[root@anatronics conf]# zkServer.sh start zoo2.cfg
[root@anatronics conf]# zkServer.sh start zoo3.cfg
```
**�鿴״̬**
```shell
[root@anatronics conf]# zkServer.sh status zoo1.cfg
/usr/bin/java
ZooKeeper JMX enabled by default
Using config: /apps/zookeeper/bin/../conf/zoo1.cfg
Client port found: 2181. Client address: localhost.
Mode: follower
[root@anatronics conf]# zkServer.sh status zoo2.cfg
/usr/bin/java
ZooKeeper JMX enabled by default
Using config: /apps/zookeeper/bin/../conf/zoo2.cfg
Client port found: 2182. Client address: localhost.
Mode: follower
[root@anatronics conf]# zkServer.sh status zoo3.cfg
/usr/bin/java
ZooKeeper JMX enabled by default
Using config: /apps/zookeeper/bin/../conf/zoo3.cfg
Client port found: 2183. Client address: localhost.
Mode: leader
```
**note**

```
#��������ռ��8080�˿�
#�����
#�޸������ű�zkServer.sh���
"-Dzookeeper.admin.enableServer=false"
```

##### 5���򵥲���

**���������й������**
```shell
[root@anatronics conf]# zkCli.sh 
/usr/bin/java
Connecting to localhost:2181
...
[zk: localhost:2181(CONNECTED) 0] 
ZooKeeper -server host:port cmd args
	addauth scheme auth
	close 
	config [-c] [-w] [-s]
	connect host:port
	create [-s] [-e] [-c] [-t ttl] path [data] [acl]
	delete [-v version] path
	deleteall path
	delquota [-n|-b] path
	get [-s] [-w] path
	getAcl [-s] path
	history 
	listquota path
	ls [-s] [-w] [-R] path
	ls2 path [watch]
	printwatches on|off
	quit 
	reconfig [-s] [-v version] [[-file path] | [-members serverID=host:port1:port2;port3[,...]*]] | [-add serverId=host:port1:port2;port3[,...]]* [-remove serverId[,...]*]
	redo cmdno
	removewatches path [-c|-d|-a] [-l]
	rmr path
	set [-s] [-v version] path data
	setAcl [-s] [-v version] [-R] path acl
	setquota -n|-b val path
	stat [-w] path
	sync path
```

**python���ӷ��񲢴�����ֵ**
```python
from kazoo.client import KazooClient
import json
zk = KazooClient(hosts="127.0.0.1:2181, 127.0.0.1:2182, 127.0.0.1:2183")
zk.start()
znode = {
  "hello": "world",
}
znode = json.dumps(znode)
znode = bytes(znode, encoding="utf-8")
zk.set("/test", znode)
```
�л��������У��鿴���
```shell
[zk: localhost:2181(CONNECTED) 6] get /test
{"hello": "world"}
```