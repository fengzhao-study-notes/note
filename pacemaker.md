> https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/high_availability_add-on_reference/index

#### 1������

- ����ϵͳ��CentOS7.5
  - node1: 192.168.56.3
  - node2: 192.168.56.4
- ϵͳ����
  
  - ���������ڵ�hosts
  
    ```shell
    [root@node1 ~]# cat /etc/hosts
    192.168.56.3 node1 node1.xl.com
    192.168.56.4 node2 node2.xl.com
    ```
  
  - ʱ��ͬ��
  
  - �ڵ��ssh����
  
  - selinux��firewall����
#### 2����װ��ذ�
���ڵ�һ������
##### 1������yumԴ
```shell
[root@node1 ~]# cat /etc/yum.repos.d/base.repo 
[base-aliyun]
name=aliyun base
baseurl=https://mirrors.aliyun.com/centos/7/os/x86_64/
gpgcheck=0

[updates]
name=aliyun updates
baseurl=https://mirrors.aliyun.com/centos/7/updates/x86_64/
gpgcheck=0

[extras]
name=aliyun extrax
baseurl=https://mirrors.aliyun.com/centos/7/extras/x86_64/
gpgcheck=0
```
##### 2����װ��ذ�
```shell
[root@node1 ~]# yum install pcs pacemaker fence-agents-all -y
```
#### 3�����ò�������ط���
##### 1������hacluster�û�����
���ڵ㶼����
```shell
# 123456
[root@node1 ~]# passwd hacluster 
Changing password for user hacluster.
New password: 
BAD PASSWORD: The password is shorter than 8 characters
Retype new password: 
passwd: all authentication tokens updated successfully.
```
##### 2������pcsd�������ýڵ�����֤
```shell
# ���ڵ㶼Ҫ��������
[root@node1 ~]# systemctl start pcsd
[root@node1 ~]# systemctl enable pcsd

# ���ýڵ����֤������һ̨���ü��ɣ�
[root@node1 ~]# pcs cluster auth node1 node2
Username: hacluster
Password: 
node1: Authorized
node2: Authorized
```
##### 3��������Ⱥ������
**������һ���ڵ��������**
```shell
# �½�һ����Ϊmycluster�ļ�Ⱥ������node1 node2��ӵ�����
[root@node1 ~]# pcs cluster setup --name mycluster node1 node2
Destroying cluster on nodes: node1, node2...
node1: Stopping Cluster (pacemaker)...
node2: Stopping Cluster (pacemaker)...
node1: Successfully destroyed cluster
node2: Successfully destroyed cluster

Sending 'pacemaker_remote authkey' to 'node1', 'node2'
node1: successful distribution of the file 'pacemaker_remote authkey'
node2: successful distribution of the file 'pacemaker_remote authkey'
Sending cluster config files to the nodes...
node1: Succeeded
node2: Succeeded

Synchronizing pcsd certificates on nodes node1, node2...
node1: Success
node2: Success
Restarting pcsd on the nodes in order to reload the certificates...
node1: Success
node2: Success

# ������Ⱥ������������
[root@node1 ~]# pcs cluster start --all
node1: Starting Cluster (corosync)...
node2: Starting Cluster (corosync)...
node1: Starting Cluster (pacemaker)...
node2: Starting Cluster (pacemaker)...
[root@node1 ~]# pcs cluster enable --all

# ����stonith�ͺ����ٲ�
[root@node1 ~]# pcs property set stonith-enabled=false
# ����ᱨ��WARNING: no stonith devices and stonith-enabled is not false

[root@node1 ~]# pcs property set no-quorum-policy=ignore
```
#### 4����Ⱥ��Դ����
##### ��Դ����

- ocf:  ocf��ʽ�������ű���/usr/lib/ocf/resource.d/
- lsb:  lsb�Ľű�һ����/etc/rc.d/init.d/

�鿴�����Դ���õĲ���

```shell
# �鿴�����Դ���õĲ���
[root@node1 ~]# pcs resource describe -h

Usage: pcs resource describe...
    describe [<standard>:[<provider>:]]<type> [--full]
        Show options for the specified resource. If --full is specified, all
        options including advanced ones are shown.
```

##### 1�������Դ

���һ��VirtualIP��Դ  ocf���͵�   ������keepalived��ip��

```shell
[root@node1 ~]# pcs resource create VirtualIP ocf:heartbeat:IPaddr2 ip=192.168.56.200 cidr_netmask=32 nic=enp0s3 op monitor interval=30s
```
�鿴��Դ״��

```shell
[root@node1 ~]# pcs resource 
 VirtualIP	(ocf::heartbeat:IPaddr2):	Started node1
```

##### 2���鿴״̬

**�鿴״̬,ip��node1����Ч**

```shell
[root@node1 ~]# pcs status
Cluster name: mycluster
Stack: corosync
Current DC: node2 (version 1.1.19-8.el7_6.4-c3c624ea3d) - partition with quorum
Last updated: Fri Jul 26 15:49:17 2019
Last change: Fri Jul 26 15:48:15 2019 by root via cibadmin on node1

2 nodes configured
1 resource configured

Online: [ node1 node2 ]

Full list of resources:

 VirtualIP	(ocf::heartbeat:IPaddr2):	Started node1

Daemon Status:
  corosync: active/disabled
  pacemaker: active/disabled
  pcsd: active/disabled
```
**��node1�ϲ鿴ip**
```shell
[root@node1 ~]# ip a
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:c4:56:a3 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.3/24 brd 192.168.56.255 scope global noprefixroute enp0s3
       valid_lft forever preferred_lft forever
    inet 192.168.56.200/32 brd 192.168.56.255 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::c97:3269:2649:c9f3/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```
**��node1��ģ��ڵ���ϲ鿴IP�Ƿ��Ư��**
```shell
# ģ��node1����
[root@node1 ~]# pcs node standby node1

# �鿴��Ⱥ״̬��node1����standby״̬��ip��Դ��node2����Ч
[root@node1 ~]# pcs status
Cluster name: mycluster
Stack: corosync
Current DC: node2 (version 1.1.19-8.el7_6.4-c3c624ea3d) - partition with quorum
Last updated: Fri Jul 26 15:53:28 2019
Last change: Fri Jul 26 15:53:22 2019 by root via cibadmin on node1

2 nodes configured
1 resource configured

Node node1: standby
Online: [ node2 ]

Full list of resources:

 VirtualIP	(ocf::heartbeat:IPaddr2):	Started node2

Daemon Status:
  corosync: active/disabled
  pacemaker: active/disabled
  pcsd: active/disabled

# node2�ϲ鿴IP
[root@node2 ~]# ip a
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:2e:4e:8c brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.4/24 brd 192.168.56.255 scope global noprefixroute enp0s3
       valid_lft forever preferred_lft forever
    inet 192.168.56.200/32 brd 192.168.56.255 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::667c:84e6:3555:33a5/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
    inet6 fe80::c97:3269:2649:c9f3/64 scope link tentative noprefixroute dadfailed 
       valid_lft forever preferred_lft forever
```
##### 3��������Դ�ڵ����ȼ�
**node1�ָ��󣬲鿴��Դ����node2����Ч**
```shell
[root@node1 ~]# pcs status nodes
Pacemaker Nodes:
 Online: node1 node2
 Standby:
 Maintenance:
 Offline:
Pacemaker Remote Nodes:
 Online:
 Standby:
 Maintenance:
 Offline:
[root@node1 ~]# pcs status resources 
 VirtualIP	(ocf::heartbeat:IPaddr2):	Started node2
```
**������Դѡ��ڵ�����ȼ�**

```shell
# ������ԴVirtualIP�Ľڵ������ԣ�����ѡ��node1������ڵ�online�������
[root@node1 ~]# pcs constraint location VirtualIP prefers node1

# �鿴��Դ�ķֲ����
[root@node1 ~]# pcs status resources 
 VirtualIP	(ocf::heartbeat:IPaddr2):	Started node1
```

##### 4������Դ������

**������Դ���С����󡱣�ʹ��ʼ�ձ�����ͬһ���ڵ���Ч��**

**1�����һ��nginx����Դ**

```shell
[root@node1 ~]# pcs resource create Nginx ocf:heartbeat:nginx op monitor interval=3s timeout=30s op start interval=0 timeout=60s op stop interval=0 timeout=60s
```

**2���鿴��Դ״̬������Դ�ֱ��ڲ�ͬ�Ľڵ�**

```shell
# �鿴����Դ״̬��nginx��node2��������Ч
[root@node1 ~]# pcs status resources 
 VirtualIP	(ocf::heartbeat:IPaddr2):	Started node1
 Nginx	(ocf::heartbeat:nginx):	Started node2
```

**3��������Դ����**

###### ��ʽһ��������Դ��

��VirtualIP��Nginx����Դ��ӵ�һ������

```shell
# ������Դ��ӵ�myweb����
[root@node1 ~]# pcs resource group add myweb VirtualIP
[root@node1 ~]# pcs resource group add myweb Nginx

# �ٴβ鿴��Դ�ֲ����
[root@node1 ~]# pcs status resources 
 Resource Group: myweb
     VirtualIP	(ocf::heartbeat:IPaddr2):	Started node1
     Nginx	(ocf::heartbeat:nginx):	Started node1
```

###### ��ʽ����������ԴԼ��constraint

ɾ��֮ǰ����Դ�飬�鿴״̬������Դ�ֲ��ڲ�ͬ�ڵ�

```shell
[root@node1 ~]# pcs resource group remove myweb

[root@node1 ~]# pcs status resources 
 VirtualIP	(ocf::heartbeat:IPaddr2):	Started node1
 Nginx	(ocf::heartbeat:nginx):	Started node2
```

**������Դ��constraint����������Դ������˳��**

```shell
[root@node1 ~]# pcs constraint colocation add VirtualIP Nginx INFINITY

# ������Դ������˳��
[root@node1 ~]# pcs constraint order start VirtualIP then start Nginx
Adding VirtualIP Nginx (kind: Mandatory) (Options: first-action=start then-action=start)

[root@node1 ~]# pcs status resources 
 VirtualIP	(ocf::heartbeat:IPaddr2):	Started node1
 Nginx	(ocf::heartbeat:nginx):	Started node1
```

**�鿴����Լ����Ϣ**

```shell
[root@node1 ~]# pcs constraint 
Location Constraints:
  Resource: VirtualIP
    Enabled on: node1 (score:INFINITY)
Ordering Constraints:
  start VirtualIP then start Nginx (kind:Mandatory)
Colocation Constraints:
  VirtualIP with Nginx (score:INFINITY)
```