### **安装参考官方文档*openstack-ocata InstallGuide***



[TOC]

#### 0、环境：

+ 系统：CentOS7.5

+ node1 192.168.10.101

    node2 192.168.10.102

+ openstack版本：openstack-ocata 

#### 1、准备

配置yum仓库 同步时间  ssh互信

关闭selinux 防火墙

```shell
$ systemctl stop firewalld
```

hosts文件

```shell
$ vim /etc/hosts
192.168.10.101 node1
192.168.10.102 node2
```

yum仓库用的是阿里云的

```
[ali_base]
name=ali_base
baseurl=https://mirrors.aliyun.com/centos/7/os/x86_64/
gpgcheck=0
[extrax]
name=extrax
baseurl=https://mirrors.aliyun.com/centos/7/extras/x86_64/
gpgcheck=0
[updates]
name=updates
baseurl=https://mirrors.aliyun.com/centos/7/updates/x86_64/
gpgcheck=0
[ceph-jewel]
name=ceph_jewel
baseurl=https://mirrors.aliyun.com/centos/7/storage/x86_64/ceph-jewel/
gpgcheck=0
[kvm-common]
name=kvm_common
baseurl=https://mirrors.aliyun.com/centos/7/virt/x86_64/kvm-common/
gpgcheck=0
[openstack]
name=openstack
baseurl=https://mirrors.aliyun.com/centos/7/cloud/x86_64/openstack-ocata/
gpgcheck=0
[epel]
name=epel
baseurl=https://mirrors.aliyun.com/epel/7/x86_64/
gpgcheck=0
```

#### 2、安装python-openstackclient

```shell
$ yum install python-openstackclient -y
```

#### 3、数据库

```shell
#安装并启动数据库
$ yum install mariadb mariadb-server python2-PyMySQL -y
$ systemctl enable mariadb
$ systemctl start mariadb
#初始化数据库
$ mysql_secure_installation 
```

#### 4、安装rabbitmq

```shell
#安装并启动rabbitmq
$ yum install rabbitmq-server -y
$ systemctl enable rabbitmq-server.service 
$ systemctl start rabbitmq-server.service 
#创建用户openstack,并设置权限
$ rabbitmqctl add_user openstack openstack
$ rabbitmqctl set_permissions openstack ".*" ".*" ".*"
```

#### 5、安装memcached

```shell
#安装并启动memcached
$ yum install memcached python-memcached -y
$ vim /etc/sysconfig/memcached 
$ systemctl enable memcached.service 
$ systemctl start memcached.service 
```

#### 6、安装openstack-keystone组件

```shell
yum install openstack-keystone httpd mod_wsgi -y
#数据库常见keystone
lMariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost'  IDENTIFIED BY 'keystone';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
IDENTIFIED BY 'keystone';

$ su -s /bin/sh -c "keystone-manage db_sync" keystone
$ keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
$ keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
$ keystone-manage bootstrap --bootstrap-password admin --bootstrap-admin-url http://192.168.10.101:35357/v3/ --bootstrap-internal-url http://192.168.10.101:5000/v3/ --bootstrap-public-url http://192.168.10.101:5000/v3/ --bootstrap-region-id RegionOne

$ vim /etc/httpd/conf/httpd.conf 

$ ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
$ vim /etc/httpd/conf.d/wsgi-keystone.conf 
$ systemctl enable httpd
$ systemctl start httpd

export OS_USERNAME=admin
export OS_PASSWORD=admin
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://192.168.10.101:35357/v3
export OS_IDENTITY_API_VERSION=3

$ openstack project create --domain default --description "Service Project" service
$ openstack project create --domain default --description "Demo Project" demo
$ openstack user create --domain default --password-prompt demo
$ openstack role create user
$ openstack role add --project demo --user demo user

$ unset OS_AUTH_URL OS_PASSWORD
$ openstack --os-auth-url http://192.168.10.101:35357/v3 --os-project-domain-name default --os-user-domain-name default --os-project-name admin --os-username admin token issue
$ openstack --os-auth-url http://192.168.10.101:35357/v3 --os-project-domain-name default --os-user-domain-name default --os-project-name demo --os-username demo token issue
```

$ vim admin-openrc 

```shell
export OS_USERNAME=admin
export OS_PASSWORD=admin
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://192.168.10.101:35357/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```

$ vim demo-openrc

```shell
export OS_USERNAME=demo
export OS_PASSWORD=demo
export OS_PROJECT_NAME=demo
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://192.168.10.101:35357/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2 
```

```shell
$ . admin-openrc 
$ openstack token issue
$ . demo-openrc 
$ openstack token issue
```

#### 7、Image service glance组件

```shell
MariaDB [(none)]> CREATE DATABASE glance;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' \
IDENTIFIED BY 'glance';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' \
IDENTIFIED BY 'glance';

$ . admin-openrc 
$ openstack user create --domain default --password-prompt glance
$ openstack role add --project service --user glance admin

$ openstack service create --name glance --description "OpenStack Image" image
$ openstack endpoint create --region RegionOne image public http://192.168.10.101:9292
$ openstack endpoint create --region RegionOne image internal http://192.168.10.101:9292
$ openstack endpoint create --region RegionOne image admin http://192.168.10.101:9292

$ yum install -y openstack-glance
$ vim /etc/glance/glance-api.conf 
$ vim /etc/glance/glance-registry.conf 
$ su -s /bin/sh -c "glance-manage db_sync" glance
$ systemctl enable openstack-glance-api.service openstack-glance-registry.service
$ systemctl start openstack-glance-api.service openstack-glance-registry.service

$ . admin-openrc 
$ openstack image create "cirros" --file cirros-0.3.5-x86_64-disk.img --disk-format qcow2 --container-format bare --public
```

#### 8、Compute service neutron组件

```shell
#数据库
MariaDB [(none)] CREATE DATABASE neutron;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' \
IDENTIFIED BY 'neutron';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' \
IDENTIFIED BY 'neutron';

$ openstack user create --domain default --password-prompt nova
$ openstack role add --project service --user nova admin
$ openstack service create --name nova --description "OpenStack Compute" compute
$ openstack endpoint create --region RegionOne compute public http://192.168.10.101:8774/v2.1
$ openstack endpoint create --region RegionOne compute internal http://192.168.10.101:8774/v2.1
$ openstack endpoint create --region RegionOne compute admin http://192.168.10.101:8774/v2.1

$ openstack user create --domain default --password-prompt placement
$ openstack role add --project service --user placement admin
$ openstack service create --name placement --description "Placement API" placement
$ openstack endpoint create --region RegionOne placement public http://192.168.10.101:8778
$ openstack endpoint create --region RegionOne placement internal http://192.168.10.101:8778
$ openstack endpoint create --region RegionOne placement admin http://192.168.10.101:8778

$ yum install openstack-nova-api openstack-nova-conductor openstack-nova-console openstack-nova-novncproxy openstack-nova-scheduler openstack-nova-placement-api
$ vim /etc/nova/nova.conf
$ vim /etc/httpd/conf.d/00-nova-placement-api.conf 
$ systemctl restart httpd
su -s /bin/sh -c "nova-manage api_db sync" nova
su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
su -s /bin/sh -c "nova-manage db sync" nova
nova-manage cell_v2 list_cells
$ systemctl enable openstack-nova-api.service openstack-nova-consoleauth.service openstack-nova-scheduler.service openstack-nova-conductor.service openstack-nova-novncproxy.service
$ systemctl start openstack-nova-api.service openstack-nova-consoleauth.service openstack-nova-scheduler.service openstack-nova-conductor.service openstack-nova-novncproxy.service
```

##### 计算节点

```shell
$ yum install openstack-nova-compute
$ vim /etc/nova/nova.conf 
$ egrep -c 'vmx' /proc/cpuinfo 
$ systemctl enable libvirtd.service openstack-nova-compute.service
$ systemctl start libvirtd.service openstack-nova-compute.service
```

##### 控制节点

```shell
$ . admin-openrc 
$ openstack hypervisor list
$ su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova
```

```shell
$ openstack compute service list
$ openstack catalog list
$ openstack image list
$ nova-status upgrade check
```

#### 9、Netwoking service  neutron组件

##### 控制节点node1

```shell
CREATE DATABASE neutron;
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' \
IDENTIFIED BY 'neutron';
GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' \
IDENTIFIED BY 'neutron';
```

```shell
$ . admin-openrc

$ openstack user create --domain default --password-prompt neutron
$ openstack role add --project service --user neutron admin
$ openstack service create --name neutron --description "OpenStack Networking" network

$ openstack endpoint create --region RegionOne \
network public http://192.168.10.101:9696

$ openstack endpoint create --region RegionOne \
network internal http://192.168.10.101:9696

$ openstack endpoint create --region RegionOne \
network admin http://192.168.10.101:9696
```



```shell
#安装相关包
$ yum install openstack-neutron openstack-neutron-ml2 \
openstack-neutron-linuxbridge ebtables -y
```

##### 两种网络选一种

###### Networking Option 1: Provider networks

```shell
$ vim /etc/neutron/neutron.conf

[database]
# ...
connection = mysql+pymysql://neutron:neutron@192.168.10.101/neutron

[DEFAULT]
# ...
core_plugin = ml2
service_plugins = 
allow_overlapping_ips = true
transport_url = rabbit://openstack:openstack@192.168.10.101
auth_strategy = keystone
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true

[keystone_authtoken]
# ...
auth_uri = http://192.168.10.101:5000
auth_url = http://192.168.10.101:35357
memcached_servers = 192.168.10.101:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = neutron

[nova]
# ...
auth_url = http://192.168.10.101:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = nova

[oslo_concurrency]
# ...
lock_path = /var/lib/neutron/tmp

$ vim /etc/neutron/plugins/ml2/ml2_conf.ini

[ml2]
# ...
type_drivers = flat,vlan
tenant_network_types = 
mechanism_drivers = linuxbridge
extension_drivers = port_security

[ml2_type_flat]
# ...
flat_networks = provider

[securitygroup]
# ...
## enable_ipset = true

$ vim /etc/neutron/plugins/ml2/linuxbridge_agent.ini
[linux_bridge]
physical_interface_mappings = provider:ens33

[vxlan]
enable_vxlan = false

[securitygroup]
# ...
enable_security_group = true
## firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver

$ vim /etc/neutron/dhcp_agent.ini
[DEFAULT]
# ...
interface_driver = linuxbridge
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = true
```

###### Networking Option 2: Self-service networks

```shell
$ vim /etc/neutron/neutron.conf
[database]
# ...
connection = mysql+pymysql://neutron:neutron@192.168.10.101/neutron

[DEFAULT]
# ...
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = true
transport_url = rabbit://openstack:openstack@192.168.10.101
auth_strategy = keystone
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true
[keystone_authtoken]
# ...
auth_uri = http://192.168.10.101:5000
auth_url = http://192.168.10.101:35357
memcached_servers = 192.168.10.101:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = neutron

[nova]
# ...
auth_url = http://192.168.10.101:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = nova

[oslo_concurrency]
# ...
## lock_path = /var/lib/neutron/tmp


$ vim /etc/neutron/plugins/ml2/ml2_conf.ini
[ml2]
# ...
type_drivers = flat,vlan,vxlan
tenant_network_types = vxlan
mechanism_drivers = linuxbridge,l2population
extension_drivers = port_security
[ml2_type_flat]
# ...
flat_networks = provider

[ml2_type_vxlan]
# ...
vni_ranges = 1:1000

[securitygroup]
# ...
## enable_ipset = true

$ vim /etc/neutron/plugins/ml2/linuxbridge_agent.ini
[linux_bridge]
physical_interface_mappings = provider:ens33

[vxlan]
enable_vxlan = true
local_ip = 192.168.10.101
l2_population = true

[securitygroup]
# ...
enable_security_group = true
## firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver

#vim /etc/neutron/l3_agent.in
[DEFAULT]
# ...
## interface_driver = linuxbridge

$ vim /etc/neutron/dhcp_agent.ini
[DEFAULT]
# ...
interface_driver = linuxbridge
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
## enable_isolated_metadata = true

$ vim /etc/neutron/metadata_agent.ini
[DEFAULT]
# ...
nova_metadata_ip = 192.168.10.101
## metadata_proxy_shared_secret = neutron

$ vim etc/nova/nova.conf
[neutron]
# ...
url = http://192.168.10.101:9696
auth_url = http://192.168.10.101:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = neutron
service_metadata_proxy = true

## metadata_proxy_shared_secret = neutron

$ ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini

$ su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
--config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron

$ systemctl restart openstack-nova-api.service

$ systemctl enable neutron-server.service \
neutron-linuxbridge-agent.service neutron-dhcp-agent.service \
neutron-metadata-agent.service

$ systemctl start neutron-server.service \
neutron-linuxbridge-agent.service neutron-dhcp-agent.service \
neutron-metadata-agent.service

以下是option2需要开启的
$ systemctl enable neutron-l3-agent.service
$ systemctl start neutron-l3-agent.service
```

##### 计算节点node2

```shell
$ yum install openstack-neutron-linuxbridge ebtables ipset

$ vim /etc/neutron/neutron.conf

[DEFAULT]
# ...
transport_url = rabbit://openstack:RABBIT_PASS@controller
[DEFAULT]
# ...
auth_strategy = keystone
[keystone_authtoken]
auth_uri = http://192.168.10.101:5000
auth_url = http://192.168.10.101:35357
memcached_servers = 192.168.10.101:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = neutron

[oslo_concurrency]
# ...
## lock_path = /var/lib/neutron/tmp

$ scp node1:/etc/neutron/plugins/ml2/linuxbridge_agent.ini /etc/neutron/plugins/ml2/

或者

$ vim /etc/neutron/plugins/ml2/linuxbridge_agent.ini
[linux_bridge]
physical_interface_mappings = provider:ens33

[vxlan]
enable_vxlan = true
local_ip = 192.168.10.101
l2_population = true

[securitygroup]
# ...
enable_security_group = true
## firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver

$ vim /etc/nova/nova.conf
[neutron]
# ...
url = http://192.168.10.101:9696
auth_url = http://192.168.10.101:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = neutron

#启动服务
$ systemctl restart openstack-nova-compute.service
$ systemctl enable neutron-linuxbridge-agent.service
$ systemctl start neutron-linuxbridge-agent.service

#验证操作在node1
$ . admin-openrc
$ openstack extension list --network
$ openstack network agent list
```



#### 10、启动一个虚拟机实例

##### 1、创建网络

创建一个provider networking

```shell
$ . admin-openrc
$  openstack network create --share --external \
--provider-physical-network provider \
--provider-network-type flat provider
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2018-12-12T14:29:21Z                 |
| description               |                                      |
| dns_domain                | None                                 |
| id                        | ee35d042-b197-4f60-bdc3-f5164ee18a8c |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| is_default                | None                                 |
| mtu                       | 1500                                 |
| name                      | provider                             |
| port_security_enabled     | True                                 |
| project_id                | 95e2347d6e3a489c90fc525a2864a4ac     |
| provider:network_type     | flat                                 |
| provider:physical_network | provider                             |
| provider:segmentation_id  | None                                 |
| qos_policy_id             | None                                 |
| revision_number           | 4                                    |
| router:external           | External                             |
| segments                  | None                                 |
| shared                    | True                                 |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| updated_at                | 2018-12-12T14:29:22Z                 |
+---------------------------+--------------------------------------+

$  openstack subnet create --network provider \
--allocation-pool start=192.168.10.30,end=192.168.10.40 \
--dns-nameserver 119.29.29.29 --gateway 192.168.10.2 \
--subnet-range 192.168.10.0/24 provider
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| allocation_pools  | 192.168.10.30-192.168.10.40          |
| cidr              | 192.168.10.0/24                      |
| created_at        | 2018-12-12T14:33:50Z                 |
| description       |                                      |
| dns_nameservers   | 8.8.4.4                              |
| enable_dhcp       | True                                 |
| gateway_ip        | 192.168.10.2                         |
| host_routes       |                                      |
| id                | b4271080-8e85-4258-a890-216eb02c2f6d |
| ip_version        | 4                                    |
| ipv6_address_mode | None                                 |
| ipv6_ra_mode      | None                                 |
| name              | provider                             |
| network_id        | ee35d042-b197-4f60-bdc3-f5164ee18a8c |
| project_id        | 95e2347d6e3a489c90fc525a2864a4ac     |
| revision_number   | 2                                    |
| segment_id        | None                                 |
| service_types     |                                      |
| subnetpool_id     | None                                 |
| updated_at        | 2018-12-12T14:33:50Z                 |
+-------------------+--------------------------------------+

$ openstack network list
+-------------------------------------+----------+--------------------------------------+
| ID                                  | Name     | Subnets                              |
+-------------------------------------+----------+--------------------------------------+
| ee35d042-b197-4f60-bdc3-f5164ee18a8 | provider | b4271080-8e85-4258-a890-216eb02c2f6d |
| c                                   |          |                                      |
+-------------------------------------+----------+--------------------------------------+
```

##### 2、创建一个flavor（默认创建实例最小内存是512M）

```shell
$ openstack flavor create --id 0 --vcpus 1 --ram 64 --disk 1 m1.nano

Policy doesn't allow os_compute_api:os-flavor-manage to be performed. (HTTP 403) (Request-ID: req-c0aed97b-5641-442b-988f-c4ce07052f1d)
报错解决：
 在/etc/nova/policy.json添加：
      {
        "os_compute_api:os-flavor-manage": ""
      }
  其它/etc/openstack-dashboard/nova_policy.json:    "os_compute_api:os-flavor-manage:discoverable": "@",
      /etc/openstack-dashboard/nova_policy.json:"os_compute_api:os-flavor-manage": "",    
          
$ openstack flavor create --id 0 --vcpus 1 --ram 64 --disk 1 m1.nano

+----------------------------+---------+
| Field                      | Value   |
+----------------------------+---------+
| OS-FLV-DISABLED:disabled   | False   |
| OS-FLV-EXT-DATA:ephemeral  | 0       |
| disk                       | 1       |
| id                         | 0       |
| name                       | m1.nano |
| os-flavor-access:is_public | True    |
| properties                 |         |
| ram                        | 64      |
| rxtx_factor                | 1.0     |
| swap                       |         |
| vcpus                      | 1       |
+----------------------------+---------+
```

##### 3、生成密钥对Generate a key pair

```shell
$ . demo-openrc

##### Generate a key pair and add a public key:

$ ssh-keygen -q -N ""
$ openstack keypair create --public-key ~/.ssh/id_rsa.pub mykey
+-------------+-------------------------------------------------+
| Field       | Value                                           |
+-------------+-------------------------------------------------+
| fingerprint | 00:f0:a6:c9:fe:36:c0:f9:4c:ed:6c:eb:a9:a9:bf:96 |
| name        | mykey                                           |
| user_id     | 9f9ac7b9ffbc4e1588f15aac0a742f38                |
+-------------+-------------------------------------------------+
Verify addition of the key pair:
$ openstack keypair list
+-------+-------------------------------------------------+
| Name  | Fingerprint                                     |
+-------+-------------------------------------------------+
| mykey | 00:f0:a6:c9:fe:36:c0:f9:4c:ed:6c:eb:a9:a9:bf:96 |
+-------+-------------------------------------------------+
```

##### 4、添加安全组规则Add security group rules

```shell
By default, the default security group applies to all instances and includes firewall rules that deny remote access to instances.

Add rules to the default security group:
– Permit ICMP (ping):
$ openstack security group rule create --proto icmp default
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| created_at        | 2018-12-12T14:58:12Z                 |
| description       |                                      |
| direction         | ingress                              |
| ether_type        | IPv4                                 |
| id                | ae599ecd-9011-4bb9-bcdd-affc1982fdf5 |
| name              | None                                 |
| port_range_max    | None                                 |
| port_range_min    | None                                 |
| project_id        | 5b942ba22b584d64960e25df68b7ed28     |
| protocol          | icmp                                 |
| remote_group_id   | None                                 |
| remote_ip_prefix  | 0.0.0.0/0                            |
| revision_number   | 1                                    |
| security_group_id | 13e9343f-e100-4b82-80dc-5c0a1ead8d21 |
| updated_at        | 2018-12-12T14:58:12Z                 |
+-------------------+--------------------------------------+
– Permit secure shell (SSH) access:
$ openstack security group rule create --proto tcp --dst-port 22 default
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| created_at        | 2018-12-12T14:59:47Z                 |
| description       |                                      |
| direction         | ingress                              |
| ether_type        | IPv4                                 |
| id                | a68452b6-7f26-49df-98b6-5d6b279551fd |
| name              | None                                 |
| port_range_max    | 22                                   |
| port_range_min    | 22                                   |
| project_id        | 5b942ba22b584d64960e25df68b7ed28     |
| protocol          | tcp                                  |
| remote_group_id   | None                                 |
| remote_ip_prefix  | 0.0.0.0/0                            |
| revision_number   | 1                                    |
| security_group_id | 13e9343f-e100-4b82-80dc-5c0a1ead8d21 |
| updated_at        | 2018-12-12T14:59:47Z                 |
+-------------------+--------------------------------------+

$ openstack security group list
+--------------------------------------+---------+------------------------+----------------------------------+
| ID                                   | Name    | Description            | Project                          |
+--------------------------------------+---------+------------------------+----------------------------------+
| 13e9343f-e100-4b82-80dc-5c0a1ead8d21 | default | Default security group | 5b942ba22b584d64960e25df68b7ed28 |
+--------------------------------------+---------+------------------------+----------------------------------+
```

##### 5、创建运行一个虚拟机实例

```shell
$ openstack server create --flavor m1.nano --image cirros \
--nic net-id=PROVIDER_NET_ID --security-group default \
--key-name mykey first-instance

PROVIDER_NET_ID可以从openstack network list获得
--image 从openstack image list获得
--security-group从openstack security group list获得
```

6、创建实例

```shell
$ openstack server create --flavor m2.nano --image cirros  --nic net-id=151145fa-d611-492f-b0b9-8d308875d245 --security-group default  --key-name mykey first-vm
+-----------------------------+-----------------------------------------------+
| Field                       | Value                                         |
+-----------------------------+-----------------------------------------------+
| OS-DCF:diskConfig           | MANUAL                                        |
| OS-EXT-AZ:availability_zone |                                               |
| OS-EXT-STS:power_state      | NOSTATE                                       |
| OS-EXT-STS:task_state       | scheduling                                    |
| OS-EXT-STS:vm_state         | building                                      |
| OS-SRV-USG:launched_at      | None                                          |
| OS-SRV-USG:terminated_at    | None                                          |
| accessIPv4                  |                                               |
| accessIPv6                  |                                               |
| addresses                   |                                               |
| adminPass                   | AzsHHG9Rig8x                                  |
| config_drive                |                                               |
| created                     | 2018-12-13T00:48:52Z                          |
| flavor                      | m2.nano (1)                                   |
| hostId                      |                                               |
| id                          | 040a8515-9bc6-419c-89af-ba638ff39191          |
| image                       | cirros (44744539-3d3d-4a74-b475-a28132ea48b8) |
| key_name                    | mykey                                         |
| name                        | first-vm                                      |
| progress                    | 0                                             |
| project_id                  | 5b942ba22b584d64960e25df68b7ed28              |
| properties                  |                                               |
| security_groups             | name='default'                                |
| status                      | BUILD                                         |
| updated                     | 2018-12-13T00:48:52Z                          |
| user_id                     | 9f9ac7b9ffbc4e1588f15aac0a742f38              |
| volumes_attached            |                                               |
+-----------------------------+-----------------------------------------------+
```

7、查看实例状态

```shell
$ openstack server list

+--------------------------------------+----------+--------+------------------------+------------+
| ID                                   | Name     | Status | Networks               | Image Name |
+--------------------------------------+----------+--------+------------------------+------------+
| 040a8515-9bc6-419c-89af-ba638ff39191 | first-vm | ACTIVE | provider=192.168.10.39 | cirros     |
+--------------------------------------+----------+--------+------------------------+------------+

虚拟控制台访问实例Access the instance using the virtual console
Obtain a Virtual Network Computing (VNC) session URL for your instance and access it from a web
browser:

$ openstack console url show first-vm

+-------+-------------------------------------------------------------------------------------+
| Field | Value                                                                               |
+-------+-------------------------------------------------------------------------------------+
| type  | novnc                                                                               |
| url   | http://192.168.10.101:6080/vnc_auto.html?token=70d0aa7e-f237-48ba-b2fa-0b60915ff963 |
+-------+-------------------------------------------------------------------------------------+
```

#### 11、安装Dashboard组件

```shell
$ yum install openstack-dashboard

$ vim /etc/openstack-dashboard/local_settings

##Configure the dashboard to use OpenStack services on the controller node:
OPENSTACK_HOST = "192.168.10.101"

##Allow your hosts to access the dashboard:
ALLOWED_HOSTS = ['*', ]

##Configure the memcached session storage service:
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
CACHES = {
'default': {
'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
'LOCATION': '192.168.10.101:11211',

##Enable the Identity API version 3:
OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST

##Enable support for domains:
OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True

##Configure API versions:
OPENSTACK_API_VERSIONS = {
"identity": 3,
"image": 2,
"volume": 2,
}

##Configure Default as the default domain for users that you create via the dashboard:
OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "Default"

##Configure user as the default role for users that you create via the dashboard:
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"

##If you chose networking option 1, disable support for layer-3 networking services:
OPENSTACK_NEUTRON_NETWORK = {
...
'enable_router': False,
'enable_quotas': False,
'enable_distributed_router': False,
'enable_ha_router': False,
'enable_lb': False,
'enable_firewall': False,
'enable_vpn': False,
'enable_fip_topology_check': False,
}

##Optionally, configure the time zone:
TIME_ZONE = "Asia/Shanghai"

$ systemctl restart httpd.service memcached.service


浏览器访问 http://192.168.10.101/dashboard/
```


