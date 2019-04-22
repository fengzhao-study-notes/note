#### ���
  ͨ��xtrabackup���ݣ���������ʵ�飩��������ͬ������ʵ��
#### ����
- ϵͳ CentOS 7.5
- node1 192.168.56.3 ��
- node2 192.168.56.4 ��

#### һ�����ݱ��ݻ�ԭ�׶�
##### �������ݻ�ԭ
###### 1������ ���� node1
```shell
# ȫ����
# xtrabackup --defaults-file=/etc/my.cnf --user=root --password=123456 --backup --target-dir=/data/mysql/base/`date +%Y%m%d_%H%M%S`
���� /data/mysql/base/20190422_082018

�޸����ݿ�1

# ��һ����������
# xtrabackup --defaults-file=/etc/my.cnf --user=root --password=123456 --backup --target-dir=/data/mysql/inc/inc1 --incremental-basedir=/data/mysql/base/20190422_082018
���� /data/mysql/inc/inc1

�޸����ݿ�2

# �ڶ����������� ��inc1�Ļ�����
# xtrabackup --defaults-file=/etc/my.cnf --user=root --password=123456 --backup --target-dir=/data/mysql/inc/inc2 --incremental-basedir=/data/mysql/inc/inc1
���� /data/mysql/inc/inc2
```
###### 2����ԭ׼��
```shell
# ��ԭ׼���׶�
# xtrabackup --prepare --apply-log-only --target-dir=/data/mysql/base/20190422_082018
# xtrabackup --prepare --apply-log-only --target-dir=/data/mysql/base/20190422_082018 --incremental-dir=/data/mysql/inc/inc1
# xtrabackup --prepare --apply-log-only --target-dir=/data/mysql/base/20190422_082018 --incremental-dir=/data/mysql/inc/inc2
```
###### 3���������,�����Ƶ��ӽڵ�
```shell
# tar cvf xtrabackup_base.tar /data/mysql/base/20190422_082018
# scp xtrabackup_base.tar node2:/data/mysql/
```
###### 4����ԭ���� �ӿ�node2
**notice �� ���ݿ����֤�رգ�/var/lib/mysqlĿ¼Ϊ��**
```shell
# ��ԭ����
# tar xf xtrabackup_base.tar
# xtrabackup --user=root --password=123456 --copy-back --target-dir=/data/mysql/20190422_082018

# �޸�Ȩ��
# chown -R mysql.mysql /var/lib/mysql/

# ��������
# systemctl start mariadb

# ��¼��־�㣬����������ʹ��
# cat xtrabackup_binlog_pos_innodb 
./mysql-bin.000002	761

```
#### ��������ͬ�����ƽ׶�
##### 1�����ӿ������ļ�
```shell
# ���������ļ�
# cat /etc/my.cnf
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
innodb_file_per_table=on
skip_name_resolve=on
log-bin=mysql-bin
binlog_format=mixed
server-id       = 1

# �ӿ������ļ�
# cat /etc/my.cnf
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
symbolic-links=0
innodb_file_per_table=on
skip_name_resolve=on
log-bin=mysql-bin
binlog_format=mixed
server-id       = 2
```
##### 2�����ⴴ��slaveͬ��user  node1
```shell
mysql> grant replication slave,reload,super on *.* to repluser@192.168.56.* identified by 'centos';
mysql> FLUSH PRIVILEGES;
```
##### 3���ӿ�����slave
```shell
# ��־��¼λ��
# cat xtrabackup_binlog_pos_innodb 
./mysql-bin.000002	761

# ִ��slave
mysql> change master to master_host='192.168.56.3',master_user='repluser',master_password='centos',master_log_file='mysql-bin.000002',master_log_pos=761;
mysql> start slave;

# �鿴
mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.56.3
                  Master_User: repluser
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000002
          Read_Master_Log_Pos: 761
               Relay_Log_File: mariadb-relay-bin.000002
                Relay_Log_Pos: 529
        Relay_Master_Log_File: mysql-bin.000002
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
```