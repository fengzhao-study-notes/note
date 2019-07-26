Jenkins��װ����

> https://jenkins.io/zh/doc/book/installing/

##### ������

- ϵͳ��CentOS7.5
  - jenkins 192.168.1.201
  - tomcat  192.168.1.51
- Java��1.8.0-212
- jenkins�� 2.176.2-1.1
- Tomcat��8.5.43

##### 1����װjava ��tomcat

```shell
# ��̨��������װjava
[root@lab ~]# yum install -y java
[root@lab ~]# java -version
openjdk version "1.8.0_212"
OpenJDK Runtime Environment (build 1.8.0_212-b04)
OpenJDK 64-Bit Server VM (build 25.212-b04, mixed mode)

# tomcat��������
[root@anatronics app]# pwd
/app
[root@anatronics app]# wget http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-8/v8.5.43/bin/apache-tomcat-8.5.43.tar.gz
[root@anatronics app]# tar xf apache-tomcat-8.5.43.tar.gz
[root@anatronics app]# ln -s apache-tomcat-8.5.43/ tomcat/

# ����tomcat manger page
[root@anatronics app]# cd tomcat/    
[root@anatronics tomcat]# vim conf/tomcat-users.xml 
<role rolename="manager-gui"/>
<role rolename="admin"/>
<role rolename="manager"/>
<role rolename="manager-script"/>
<user username="jenkins" password="jenkins" roles="manager-gui,admin,manager,manager-script"/>
[root@anatronics tomcat]# vim webapps/manager/META-INF/context.xml 
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="^.*$" />
```

##### 2����ȡjenkins��,����װ

```shell
[root@lab ~]# wget https://pkg.jenkins.io/redhat-stable/jenkins-2.176.2-1.1.noarch.rpm
[root@lab ~]# yum install ./jenkins-2.176.2-1.1.noarch.rpm
[root@lab ~]# rpm -ql jenkins
/etc/init.d/jenkins
/etc/logrotate.d/jenkins
/etc/sysconfig/jenkins
/usr/lib/jenkins
/usr/lib/jenkins/jenkins.war
/usr/sbin/rcjenkins
/var/cache/jenkins
/var/lib/jenkins
/var/log/jenkins
```
##### 3����������

```shell
[root@lab ~]# systemctl start jenkins
[root@lab ~]# ss -tlnp|grep java
LISTEN     0      50          :::8080                    :::*                   users:(("java",pid=18520,fd=166))
```

##### 4����ʼ������

���������192.168.1.201:8080

![1563435439439](assets/1563435439439.png)

������ʾ�鿴���룺

```shell
[root@lab ~]# cat /var/lib/jenkins/secrets/initialAdminPassword
658a430f0dea4c15860abc04c9365b8c
```

�������룬��װ�����֮�����ù���Ա�û�

![1563436017811](assets/1563436017811.png)

����ҳ��

![1563436134877](assets/1563436134877.png)

Deploy to container Plugin

Publish Over SSH

##### 5������һ��maven demo

��װdeploy to contaner���

1���½�һ��maven��Ŀ

![1563499376432](assets/1563499376432.png)

2��ѡ��Դ����Դgit

> https://github.com/bingyue/easy-springmvc-maven

![1563499410244](assets/1563499410244.png)

![1563499541366](assets/1563499541366.png)

3�����������

![1563499596700](assets/1563499596700.png)



4��ѡ�������������鿴����̨��Ϣ

![1563499126703](assets/1563499126703.png)

5����֤����tomcat��������

�鿴tomcatĿ¼

```shell
[root@anatronics webapps]# pwd
/app/tomcat/webapps
[root@anatronics webapps]# ls
app  app.war  docs  examples  host-manager  manager  ROOT
```

���������http://192.168.1.51:8080/app

![1563499250501](assets/1563499250501.png)

![1563499301810](assets/1563499301810.png)

<<<<<<< HEAD
=======
**����git��tag��ȡԴ��**

1����װgit parameter���

2���½���Ŀ��ѡ���������������

![1563933371598](assets/1563933371598.png)

3��׼��githubԴ���tag,��������ȥ

```shell
[root@anatronics]# git clone https://github.com/weijuwei/easy-springmvc-maven.git

# ��һ�θ����ύ������tag����push
[root@anatronics easy-springmvc-maven]# git add .
[root@anatronics easy-springmvc-maven]# git commit -m "update user pass"
[root@anatronics easy-springmvc-maven]# git tag -a "1.0.0"
[root@anatronics easy-springmvc-maven]# git push origin master
[root@anatronics easy-springmvc-maven]# git push --tags 

# �ڶ��θ����ύ������tag����push
[root@anatronics easy-springmvc-maven]# git add .
[root@anatronics easy-springmvc-maven]# git commit -m "update user pass"
[root@anatronics easy-springmvc-maven]# git tag -a "1.0.1"
[root@anatronics easy-springmvc-maven]# git push origin master
[root@anatronics easy-springmvc-maven]# git push --tags 

# �鿴tag
[root@anatronics easy-springmvc-maven]# git tag
1.0.0
1.0.1
```

4��ѡ��Bulid with Parameters

![1563933749095](assets/1563933749095.png)
>>>>>>> 566881d6ce561b13e4ea50f3068f213a5624089d
