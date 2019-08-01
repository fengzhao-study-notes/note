#### ����
- ϵͳ��CentOS 7.5 
  - 192.168.56.4 tracker storage1 node2
  - 192.168.56.3 storage2  node1
  - ����ǽ��selinux����
- fastDFS v5.11
- nginx 1.14.2
#### ���
FastDFS ��һ����Դ�ĸ����ֲܷ�ʽ�ļ�ϵͳ��DFS���� ������Ҫ���ܰ������ļ��洢���ļ�ͬ�����ļ����ʣ��Լ��������͸���ƽ�⡣��Ҫ����˺������ݴ洢���⣬�ر��ʺ�����С�ļ������鷶Χ��4KB < file_size <500MB��Ϊ��������߷���
FastDFS ϵͳ��������ɫ�����ٷ�����(Tracker Server)���洢������(Storage Server)�Ϳͻ���(Client)��
- Tracker Server
  ���ٷ���������Ҫ�����ȹ������𵽾�������ã�����������е� storage server�� group��ÿ�� storage ������������� Tracker����֪�Լ����� group ����Ϣ��������������������
- Storage Server
  �洢����������Ҫ�ṩ�����ͱ��ݷ����� group Ϊ��λ��ÿ�� group �ڿ����ж�̨ storage server�����ݻ�Ϊ���ݡ�
- Client
  �ͻ��ˣ��ϴ��������ݵķ�������Ҳ���������Լ�����Ŀ�������ڵķ�������

**�ܹ�ͼ**

![1564562604593](assets/1564562604593.png)

**�ϴ�����**

��Tracker�յ��ͻ����ϴ��ļ�������ʱ����Ϊ���ļ�����һ�����Դ洢�ļ���group����ѡ����group���Ҫ�������ͻ��˷���group�е���һ��storage server���������storage server�󣬿ͻ�����storage����д�ļ�����storage����Ϊ�ļ�����һ�����ݴ洢Ŀ¼��Ȼ��Ϊ�ļ�����һ��fileid�����������ϵ���Ϣ�����ļ����洢�ļ���

![img](assets/20180516113258902.png)

**��������**

tracker����������ļ�·�����ļ�ID �����ٶ����ļ���

���磺group1/M00/00/00/wKg4BF1BUsWAZ455AJZEwwk_Q7s446.zip

1.ͨ������tracker�ܹ��ܿ�Ķ�λ���ͻ�����Ҫ���ʵĴ洢����������group1����ѡ����ʵĴ洢�������ṩ�ͻ��˷��ʡ�  

2.�洢���������ݡ��ļ��洢�������·�����͡������ļ�����Ŀ¼�����Ժܿ춨λ���ļ�����Ŀ¼���������ļ����ҵ��ͻ�����Ҫ���ʵ��ļ�

![img](assets/20180516113332205.png)

#### 1����װ��س���

> https://github.com/happyfish100/fastdfs/blob/master/INSTALL
https://github.com/happyfish100

����libfastcommonԴ�룬���밲װ node1 node2���ڵ㶼���в���

```shell
[root@node2 pkgs]# git clone https://github.com/happyfish100/libfastcommon.git
[root@node2 pkgs]# cd libfastcommon/
[root@node2 libfastcommon]# ./make.sh
[root@node2 libfastcommon]# ./make.sh install

# ���������ӣ�fastDFSԴ�������Ҫ
[root@node2 fastdfs-5.11]# ln -s /usr/lib64/libfastcommon.so /usr/local/lib/libfastcommon.so
[root@node2 fastdfs-5.11]# ln -s /usr/lib64/libfastcommon.so /usr/lib/libfastcommon.so
[root@node2 fastdfs-5.11]# ln -s /usr/lib64/libfdfsclient.so /usr/lib/libfdfsclient.so
[root@node2 fastdfs-5.11]# ln -s /usr/lib64/libfdfsclient.so /usr/local/lib/libfdfsclient.so

```
����fastDFSԴ���,���밲װ
```shell
[root@node2 ~]# wget https://github.com/happyfish100/fastdfs/archive/V5.11.tar.gz
[root@node2 ~]# tar xf V5.11.tar.gz
[root@node2 ~]# cd fastdfs-5.11/
[root@node2 fastdfs-5.11]# ./make.sh 
[root@node2 fastdfs-5.11]# ./make.sh install

# ��������������
[root@node2 fastdfs-5.11]# ls /usr/bin/fdfs_*
/usr/bin/fdfs_appender_test   /usr/bin/fdfs_download_file  /usr/bin/fdfs_test1
/usr/bin/fdfs_appender_test1  /usr/bin/fdfs_file_info      /usr/bin/fdfs_trackerd
/usr/bin/fdfs_append_file     /usr/bin/fdfs_monitor        /usr/bin/fdfs_upload_appender
/usr/bin/fdfs_crc32           /usr/bin/fdfs_storaged       /usr/bin/fdfs_upload_file
/usr/bin/fdfs_delete_file     /usr/bin/fdfs_test

# ��������ļ�����
[root@node2 fastdfs-5.11]# cd /etc/fdfs/
[root@node2 fdfs]# ls
client.conf.sample  storage.conf.sample  storage_ids.conf.sample  tracker.conf.sample
```
#### 2�����ò�������ط���
##### 1����������tracker
```shell
# �������ݴ��Ŀ¼
[root@node2 fdfs]# mkdir /data/fastdfs/tracker

[root@node2 fdfs]# cp tracker.conf.sample tracker.conf
# �޸�base_path
[root@node2 fdfs]# vim tracker.conf
base_path=/data/fastdfs/tracker

# ��������
[root@node2 fdfs]# service fdfs_trackerd start

# base_pathĿ¼�������ļ�
[root@node2 fdfs]# tree /data/fastdfs/tracker/
/data/fastdfs/tracker/
������ data
��?? ������ fdfs_trackerd.pid
��?? ������ storage_changelog.dat
������ logs
    ������ trackerd.log
```
##### 2����������storage
�������ڵ㶼���в���
```shell
# �������Ŀ¼
[root@node2 fdfs]# mkdir /data/fastdfs/{storage,file}

[root@node2 fdfs]# cp storage.conf.sample storage.conf
# �޸������ļ�
[root@node2 fdfs]# vim storage.conf
base_path=/data/fastdfs/storage
store_path0=/data/fastdfs/file
tracker_server=192.168.56.4:22122

# ��������
[root@node2 fdfs]# service fdfs_storaged start

# �����˿�2300
[root@node2 fdfs]# ss -tlnp | grep fdfs
LISTEN     0      128          *:22122                    *:*                   users:(("fdfs_trackerd",pid=16891,fd=5))
LISTEN     0      128          *:23000                    *:*                   users:(("fdfs_storaged",pid=23318,fd=5))

# ��������Ŀ¼
[root@node2 fdfs]# ls /data/fastdfs/file/data/
00  0C  18  24  30  3C  48  54  60  6C  78  84  90  9C  A8  B4  C0  CC  D8  E4  F0  FC
01  0D  19  25  31  3D  49  55  61  6D  79  85  91  9D  A9  B5  C1  CD  D9  E5  F1  FD
02  0E  1A  26  32  3E  4A  56  62  6E  7A  86  92  9E  AA  B6  C2  CE  DA  E6  F2  FE
03  0F  1B  27  33  3F  4B  57  63  6F  7B  87  93  9F  AB  B7  C3  CF  DB  E7  F3  FF
04  10  1C  28  34  40  4C  58  64  70  7C  88  94  A0  AC  B8  C4  D0  DC  E8  F4
05  11  1D  29  35  41  4D  59  65  71  7D  89  95  A1  AD  B9  C5  D1  DD  E9  F5
06  12  1E  2A  36  42  4E  5A  66  72  7E  8A  96  A2  AE  BA  C6  D2  DE  EA  F6
07  13  1F  2B  37  43  4F  5B  67  73  7F  8B  97  A3  AF  BB  C7  D3  DF  EB  F7
08  14  20  2C  38  44  50  5C  68  74  80  8C  98  A4  B0  BC  C8  D4  E0  EC  F8
09  15  21  2D  39  45  51  5D  69  75  81  8D  99  A5  B1  BD  C9  D5  E1  ED  F9
0A  16  22  2E  3A  46  52  5E  6A  76  82  8E  9A  A6  B2  BE  CA  D6  E2  EE  FA
0B  17  23  2F  3B  47  53  5F  6B  77  83  8F  9B  A7  B3  BF  CB  D7  E3  EF  FB

# �鿴monitor��Ϣ
[root@node2 fdfs]# fdfs_monitor /etc/fdfs/storage.conf
group count: 1

Group 1:
group name = group1
disk total space = 5110 MB
disk free space = 2111 MB
trunk free space = 0 MB
storage server count = 2
active server count = 2
storage server port = 23000
storage HTTP port = 8888
store path count = 1
subdir count per path = 256
current write server index = 0
current trunk file id = 0

	Storage 1:
		id = 192.168.56.3
		ip_addr = 192.168.56.3 (node1)  ACTIVE
		http domain = 
		version = 5.11
		join time = 2019-08-01 09:23:49
		up time = 2019-08-01 09:23:49
		total storage = 5110 MB
		free storage = 2111 MB
		.....
	Storage 2:
		id = 192.168.56.4
		ip_addr = 192.168.56.4 (node2)  ACTIVE
		http domain = 
		version = 5.11

```
##### 3������client
```shell
[root@node2 fdfs]# mkdir /data/fastdfs/client
[root@node2 fdfs]# vim client.conf
base_path=/data/fastdfs/client
tracker_server=192.168.56.4:22122
```
##### 4���ϴ�����
```shell
# ��node2�ϴ��ļ�
[root@node2 fdfs]# fdfs_upload_file client.conf /root/wordpress-4.9.4-zh_CN.zip 
group1/M00/00/00/wKg4BF1BUsWAZ455AJZEwwk_Q7s446.zip

# �ϴ�֮����λ��
[root@node2 fdfs]# ls /data/fastdfs/file/data/00/00/wKg4BF1BUsWAZ455AJZEwwk_Q7s446.zip 
/data/fastdfs/file/data/00/00/wKg4BF1BUsWAZ455AJZEwwk_Q7s446.zip

# ��node1�ϲ鿴�ϴ����ļ�
[root@node1 fdfs]# ls /data/fastdfs/file/data/00/00/wKg4BF1BUsWAZ455AJZEwwk_Q7s446.zip 
/data/fastdfs/file/data/00/00/wKg4BF1BUsWAZ455AJZEwwk_Q7s446.zip
# �ļ�Ҳ��ͬ����node1�ϣ���Ϊ���ڵ�ͬ����group1
```
���ص��ļ�ID��group���洢Ŀ¼��������Ŀ¼��fileid���ļ���׺�����ɿͻ���ָ������Ҫ���������ļ����ͣ�ƴ�Ӷ��ɡ�
#### 3��nginx+fastdfs-module
##### 1����ȡԴ���
```shell
# ��ȡnginxԴ���
[root@node2 pkgs]# wget http://nginx.org/download/nginx-1.14.2.tar.gz
[root@node2 pkgs]# tar xf nginx-1.14.2.tar.gz 

# ��ȡģ��Դ��
[root@node2 pkgs]# git clone https://github.com/happyfish100/fastdfs-nginx-module.git
```
##### 2�����밲װnginx
```shell
[root@node2 pkgs]# cd nginx-1.14.2/

[root@node2 nginx-1.14.2]# ./configure --prefix=/apps/nginx --add-module=/root/pkgs/fastdfs-nginx-module/src
```
**���뱨����**
```
# ����
/usr/include/fastdfs/fdfs_define.h:15:27: fatal error: common_define.h: No such file or directory
 #include "common_define.h"
# ���
[root@node2 nginx]# vim ../fastdfs-nginx-module/src/config
ngx_module_incs="/usr/include/fastdfs /usr/include/fastcommon/"
CORE_INCS="$CORE_INCS /usr/include/fastdfs /usr/include/fastcommon/"
```
##### 3����������ļ�
###### 1�����������ļ���/etc/fdfs/
```shell
# ����fastdfsԴ����е��ļ���/etc/fdfs
[root@node2 pkgs]# cd fastdfs-5.11/
[root@node2 fastdfs-5.11]# ls
client  conf             fastdfs.spec  init.d   make.sh     README.md   stop.sh  test
common  COPYING-3_0.txt  HISTORY       INSTALL  php_client  restart.sh  storage  tracker
[root@node2 fastdfs-5.11]# cd conf
[root@node2 conf]# ls
anti-steal.jpg  client.conf  http.conf  mime.types  storage.conf  storage_ids.conf  tracker.conf
[root@node2 conf]# cp anti-steal.jpg mime.types http.conf /etc/fdfs/

# ����nginx-fastdfs-module�е������ļ���/etc/fdfs��
[root@node2 ~]# cd pkgs/fastdfs-nginx-module/src/
[root@node2 src]# ls
common.c  common.h  config  mod_fastdfs.conf  ngx_http_fastdfs_module.c
[root@node2 src]# cp mod_fastdfs.conf /etc/fdfs/
```
###### 2���޸������ļ�
1����nginx.conf�������������
```
    server {
        listen 8888;
        server_name 192.168.56.4;
        location /M00 {
            ngx_fastdfs_module;
        }
    }
```
2���޸�/etc/fdfs/http.conf
```shell 
[root@node2 fdfs]# vim http.conf 
http.anti_steal.token_check_fail=/etc/fdfs/anti-steal.jpg
```
3���޸�/etc/fdfs/mod_fastdfs.conf 
```shell
[root@node2 fdfs]# vim mod_fastdfs.conf 
store_path0=/data/fastdfs/file
tracker_server=192.168.56.4:22122
```
##### 4������nginx����
```shell
[root@node2 ~]# /apps/nginx/sbin/nginx 
```
����һ���ϴ����ı��ļ�
```shell
# �ϴ�һ���ı������ļ�
[root@node2 ~]# cat test.txt 
enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.56.4  netmask 255.255.255.0  broadcast 192.168.56.255
        inet6 fe80::c97:3269:2649:c9f3  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:2e:4e:8c  txqueuelen 1000  (Ethernet)
        RX packets 90480585  bytes 8142451739 (7.5 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 88937276  bytes 9049292916 (8.4 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 255341  bytes 13296404 (12.6 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 255341  bytes 13296404 (12.6 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@node2 ~]# fdfs_upload_file /etc/fdfs/client.conf test.txt 
/group1/M00/00/00/wKg4BF1CUxmAFD9-AAADlBZiHr0220.txt

[root@node2 ~]# curl http://192.168.56.4:8888/M00/00/00/wKg4BF1CUxmAFD9-AAADlBZiHr0220.txt
enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.56.4  netmask 255.255.255.0  broadcast 192.168.56.255
        inet6 fe80::c97:3269:2649:c9f3  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:2e:4e:8c  txqueuelen 1000  (Ethernet)
        RX packets 90480585  bytes 8142451739 (7.5 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 88937276  bytes 9049292916 (8.4 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 255341  bytes 13296404 (12.6 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 255341  bytes 13296404 (12.6 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```