> https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/
##### �򵥽���
��keepalived + haproxy + kubeadm��ʽ������̨master���ؾ���߿��á�
##### ����
ϵͳ��Ubuntu 18.04
docker: 19.03.5
k8s��v1.17.0
������
- node01: 192.168.20.101 master
- node02: 192.168.20.102 master
- node03: 192.168.20.103 node
- VIP: 192.168.20.20
������ssh���ţ�ʱ��ͬ����hosts����
##### keepalived
**��װ**
```
root@node02:~# apt install -y keepalived
```
**�����ļ�**
node02�������
```shell
root@node02:~# cat /etc/keepalived/keepalived.conf 
! Configuration File for keepalived

global_defs {
   notification_email {
     acassen
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}

vrrp_script check_haproxy {
    script "killall -0 haproxy"
    interval 3
    weight -2
    fall 10
    rise 2
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    garp_master_delay 10
    smtp_alert
    virtual_router_id 51
    priority 120
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.20.20
    }
    track_script {
        check_haproxy
    }
}
```
node01�������
```shell
root@node01:~# cat /etc/keepalived/keepalived.conf 
! Configuration File for keepalived

global_defs {
   notification_email {
     acassen
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}

vrrp_script check_haproxy {
    script "killall -0 haproxy"
    interval 3
    weight -2
    fall 10
    rise 2
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    garp_master_delay 10
    smtp_alert
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.20.20
    }
    track_script {
        check_haproxy
    }
}
```
**�����鿴**

```shell
# systemctl start keepalived.service 
root@node02:~# ip a show eth0 
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:66:9f:cc brd ff:ff:ff:ff:ff:ff
    inet 192.168.20.102/24 brd 192.168.20.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet 192.168.20.20/32 scope global eth0
       valid_lft forever preferred_lft forever
```
##### haproxy
��̨��������һ��
```
# haproxy��������
listen stats
    bind                 0.0.0.0:1080
	mode http
    stats refresh        5s
    stats uri            /status
    stats auth admin:admin
    stats admin if TRUE

frontend kubernetes-apiserver
   mode  tcp
   bind  0.0.0.0:8443
   option   tcplog
   default_backend     kubernetes-apiserver
backend kubernetes-apiserver
    balance     roundrobin
    mode        tcp
    server  node01 192.168.20.101:6443 check inter 5000 fall 2 rise 2 weight 1
    server  node02 192.168.20.102:6443 check inter 5000 fall 2 rise 2 weight 1
```
����
```shell
systemctl start haproxy
```
##### ����docker��k8s

��̨���������в���
###### ���׼��
1���Ƴ��ɰ汾docker

```shell
apt-get autoremove docker
apt-get remove docker  docker-engine docker-ce docker.io -y
```
2�����docker��kubernetes aptԴ
```shell
apt -y install apt-transport-https ca-certificates curl software-properties-common

curl -fsSL https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add -
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | apt-key add -

cat <<EOF >/etc/apt/sources.list.d/docker-k8s.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable
EOF
```
3����װ��ذ�
```shell
apt update
apt install kubelet kubeadm kubectl docker-ce
```

4���ر�swap
```
swapoff -a
```
5��������ȡ

��ʹ�ùٷ�����Դ��ʹ�õ��ǰ����Ƶ�
```
kubeadm config images pull --image-repository=registry.aliyuncs.com/google_containers
```
###### master��װ
����VIP����������ִ�в�������ǰ��node02
����kubelet
```shell
systemctl enable kubelet
systemctl start kubelet
```
**��ʼ��master**
**--control-plane-endpoint ΪVIP**
```shell
root@node02:~# kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.20.102 --control-plane-endpoint 192.168.20.20:8443 --image-repository=registry.aliyuncs.com/google_containers

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join 192.168.20.20:8443 --token 3fjt0d.1vzgmr16npj6knq7 \
    --discovery-token-ca-cert-hash sha256:d6a4160d51817929617898c69300d109b2150f5cc3f5f693a71fa556b88ca10b \
    --control-plane 

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.20.20:8443 --token 3fjt0d.1vzgmr16npj6knq7 \
    --discovery-token-ca-cert-hash sha256:d6a4160d51817929617898c69300d109b2150f5cc3f5f693a71fa556b88ca10b 
```
����������ʾ���в���
```shell
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
�鿴�ڵ���Ϣ
```shell
root@node02:~# kubectl get node
NAME     STATUS     ROLES    AGE   VERSION
node02   NotReady   master   11m   v1.17.0
```
**��װ������**
```shell
root@node02:~#kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```
�ٴβ鿴�ڵ��Ծ���
```shell
root@node01:~# kubectl get node 
NAME     STATUS   ROLES    AGE   VERSION
node02   Ready    master   13m   v1.17.0
```

###### ����master�ڵ�
����֮ǰmaster�ڵ��ϵ����key��֤���ļ�������Ҫ����master�Ľڵ���
```shell
root@node02:~# cat k8s_master_ca.sh 
#!/bin/bash

if [ $# -eq 0 ];
then
    echo "usage: $0 [host|ip] "
    exit
fi

for HOST in $*
do
	#ssh $HOST "mkdir /tmp/testest"

	ssh $HOST "mkdir -p /etc/kubernetes/pki/etcd/"
	scp /etc/kubernetes/pki/{ca.*,sa.*,front-proxy-ca.*} $HOST:/etc/kubernetes/pki/
	scp /etc/kubernetes/admin.conf $HOST:/etc/kubernetes/
	scp /etc/kubernetes/pki/etcd/ca.* $HOST:/etc/kubernetes/pki/etcd/

	echo "$HOST transport OK......"
	echo "---------------------"
done
```
ִ������
```shell
systemctl enable kubelet
systemctl start kubelet

kubeadm join 192.168.20.20:8443 --token 3fjt0d.1vzgmr16npj6knq7 \
    --discovery-token-ca-cert-hash sha256:d6a4160d51817929617898c69300d109b2150f5cc3f5f693a71fa556b88ca10b \
    --control-plane 
```
�鿴���
```shell
root@node02:~# kubectl get node
NAME     STATUS   ROLES    AGE     VERSION
node01   Ready    master   26s     v1.17.0
node02   Ready    master   5m15s   v1.17.0shell
```

######   ����node�ڵ�

```shell
root@node03:~# apt install docker-ce kubelet -y
root@node03:~# systemctl enable docker
root@node03:~# systemctl enable kubelet
root@node03:~# systemctl start docker
root@node03:~# systemctl start kubelet

root@node03:~# kubeadm join 192.168.20.20:8443 --token te1kgf.z0a9eocjhbw94spc     --discovery-token-ca-cert-hash sha256:d6a4160d51817929617898c69300d109b2150f5cc3f5f693a71fa556b88ca10b
```

```shell
root@node01:~/k8s# kubectl get node
NAME     STATUS   ROLES    AGE     VERSION
node01   Ready    master   3d20h   v1.17.0
node02   Ready    master   3d20h   v1.17.0
node03   Ready    <none>   3h24m   v1.17.0
```

###### master�ڵ����pod���ȷ���

```shell
root@node02:~# kubectl taint nodes --all node-role.kubernetes.io/master-
node/node01 untainted
node/node02 untainted
```

**master�ڵ�ָ���������**

```shell
kubectl taint node localhost.localdomain node-role.kubernetes.io/master="":NoSchedule
```

###### **�޸�·��ת��ģʽipvs���iptables**

���нڵ㶼ִ��

1����������ں�ģ��

```shell
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
```

2���޸��ں˲���

```shell
root@node03:~# cat >> /etc/sysctl.conf << EOF
> net.ipv4.ip_forward = 1
> net.bridge.bridge-nf-call-iptables = 1
> net.bridge.bridge-nf-call-ip6tables = 1
> EOF
root@node03:~# sysctl -p
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
```

3��master�ڵ�ִ��

�޸�configMap�е�kube-proxy�е�mode�޸�Ϊipvs

```shell
root@node01:~/k8s# kubectl get cm kube-proxy -n kube-system
NAME         DATA   AGE
kube-proxy   2      3d20h
root@node01:~/k8s# kubectl edit cm kube-proxy -n kube-system
 mode: "ipvs"
```

4��ɾ��kube-system���ƿռ���kubec-proxy-xxxxPod,�������������µ�Pod

```shell
root@node02:/etc/kubernetes/manifests# kubectl get pod -n kube-system |grep kube-proxy 
kube-proxy-j977j                 1/1     Running   1          3d20h
kube-proxy-pvkhr                 1/1     Running   1          3d20h
kube-proxy-tdfs6                 1/1     Running   1          166m
root@node02:/etc/kubernetes/manifests# kubectl get pod -n kube-system |grep kube-proxy |awk '{system("kubectl delete pod "$1" -n kube-system")}'
pod "kube-proxy-j977j" deleted
pod "kube-proxy-pvkhr" deleted
pod "kube-proxy-tdfs6" deleted
root@node02:/etc/kubernetes/manifests# kubectl get pod -n kube-system |grep kube-proxy 
kube-proxy-c8ddm                 1/1     Running   0          12s
kube-proxy-s5pf9                 1/1     Running   0          10s
kube-proxy-zq4qf                 1/1     Running   0          4s
```

5����װipvsadm�鿴���ɵ�ipvs����

```shell
root@node02:~# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.20.102:32254 rr
  -> 10.244.2.8:80                Masq    1      0          0         
TCP  10.96.0.1:443 rr
  -> 192.168.20.101:6443          Masq    1      0          0         
  -> 192.168.20.102:6443          Masq    1      0          0         
TCP  10.96.0.10:53 rr
  -> 10.244.0.4:53                Masq    1      0          0         
  -> 10.244.0.5:53                Masq    1      0          0         
TCP  10.96.0.10:9153 rr
  -> 10.244.0.4:9153              Masq    1      0          0         
  -> 10.244.0.5:9153              Masq    1      0          0         
TCP  10.96.221.70:80 rr
  -> 10.244.2.8:80                Masq    1      0          0         
TCP  10.244.0.0:32254 rr
  -> 10.244.2.8:80                Masq    1      0          0         
TCP  10.244.0.1:32254 rr
  -> 10.244.2.8:80                Masq    1      0          0         
```

##### �鿴haproxy+keepalived״̬��Ϣ

������д�http://192.168.20.20:1080/status�鿴���������Ϣ