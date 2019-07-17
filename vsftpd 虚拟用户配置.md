#### ����vsfptd�����û�
##### ����
- ϵͳ��CentOS7.5
- firewall��selinux���ѽ�ֹ
- vsftpd: version 3.0.2
##### 1�������û������ļ�
```shell
[root@node1 vsftpd]# cat vusers.txt 
luoshaojun
lsj2019
leo
leo2019

[root@node1 vsftpd]# db_load -T -t hash -f vusers.txt vusers.db
[root@node1 vsftpd]# chmod 600 vusers.db
```
##### 2�����vuser�û���FTPĿ¼
```shell
[root@node1 vsftpd]# mkdir /ftp
[root@node1 vsftpd]# useradd -d /ftp -s /sbin/nologin vuser

[root@node1 vsftpd]# chmod +rx /ftp
[root@node1 vsftpd]# chmod -w /ftp
[root@node1 vsftpd]# mkdir /ftp/upload
[root@node1 vsftpd]# setfacl -m u:vuser:rwx /ftp/upload
```
##### 3������pam�ļ�
```shell
[root@node1 vusers.d]# vim /etc/pam.d/vsftpd.pam 
auth required pam_userdb.so db=/etc/vsftpd/vusers
account required pam_userdb.so db=/etc/vsftpd/vusers
```
##### 4����vsftpd.conf�п�������ӳ���û�,��ָ��pam�ļ�
```shell
[root@node1 vsftpd]# vim vsftpd.conf
guest_enable=YES
guest_username=vuser
pam_service_name=vsftpd.pam
```
##### 5��Ϊ�������û��������Ե������ļ�
```shell
[root@node1 vsftpd]# mkdir vusers.d

[root@node1 vsftpd]# vim vsftpd.conf
user_config_dir=/etc/vsftpd/vusers.d/

# luoshaojun�û����ϴ�������ɾ�������ǡ�������
[root@node1 vsftpd]# cat vusers.d/luoshaojun 
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=NO
write_enable=YES

# leo�û�ֻ������
[root@node1 vsftpd]# cat vusers.d/leo 
anon_world_readable_only=NO
local_root=/ftp
```
##### 6����������
```shell
[root@node1 vsftpd]# systemctl enable vsftpd
[root@node1 vsftpd]# systemctl start vsftpd
```
