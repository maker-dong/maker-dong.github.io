# Kubernetes集群安装指北



# 一、前期准备

## 1、虚拟机分配说明

| 地址         | 主机名        | 角色   |
| :----------- | :------------ | :----- |
| 172.16.156.1 | k8s-master-01 | master |
| 172.16.156.2 | k8s-master-02 | master |
| 172.16.156.3 | k8s-master-03 | master |
| 172.16.156.4 | k8s-node-01   | node   |
| 172.16.156.5 | k8s-node-02   | node   |

## 2、各个节点端口占用

- **Master 节点**

| 规则 | 方向    | 端口范围  | 作用                    | 使用者                          |
| :--- | :------ | :-------- | :---------------------- | :------------------------------ |
| TCP  | Inbound | 6443*     | Kubernetes API          | server All                      |
| TCP  | Inbound | 2379-2380 | etcd server             | client API kube-apiserver, etcd |
| TCP  | Inbound | 10250     | Kubelet API             | Self, Control plane             |
| TCP  | Inbound | 10251     | kube-scheduler          | Self                            |
| TCP  | Inbound | 10252     | kube-controller-manager | Sel                             |

- **node 节点**

| 规则 | 方向    | 端口范围    | 作用                | 使用者              |
| :--- | :------ | :---------- | :------------------ | :------------------ |
| TCP  | Inbound | 10250       | Kubelet API         | Self, Control plane |
| TCP  | Inbound | 30000-32767 | NodePort Services** | All                 |

## 3、基础环境设置

Kubernetes 需要一定的环境来保证正常运行，如各个节点时间同步，主机名称解析，关闭防火墙等等。

### 免密设置 

(可以先用ssh跳转一下试试，1节点到其他节点能跳就行，不用每次都设置。)

第一步:在本地机器上使用ssh-keygen产生公钥私钥对

1. ```
   ssh-keygen
   ```

2. 第二步:用ssh-copy-id将公钥复制到远程机器中

$ ssh-copy-id -i ~/.ssh/id_rsa.pub 用户名字@172.16.x.xxx

如果提示ssh-copy-id 不存在，则安装 yum install -y openssh-clients

**注意:** ssh-copy-id **将key写到远程机器的 /root/.ssh/authorized_key.文件中
或者手动复制本地/root/.ssh/id_rsa.pub 追加到目标的/root/.ssh/authorized_key文件中

### 主机名称解析

分布式系统环境中的多主机通信通常基于主机名称进行，这在 IP 地址存在变化的可能 性时为主机提供了固定的访问人口，因此一般需要有专用的 DNS 服务负责解决各节点主机 不过，考虑到此处部署的是测试集群，因此为了降低系复杂度，这里将基于 hosts 的文件进行主机名称解析。

#### **修改hosts**

分别进入不同服务器，进入 /etc/hosts 进行编辑

```
vim /etc/hosts
```

加入下面内容：

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
172.16.156.1   k8s-master-01
172.16.156.2   k8s-master-02
172.16.156.3   k8s-master-03
172.16.156.4   k8s-node-01
172.16.156.5   k8s-node-02
```

#### **修改hostname**

分别进入不同的服务器修改 hostname 名称

```
# 修改 172.16.156.1 服务器
hostnamectl  set-hostname  k8s-master-01
# 修改 172.16.156.2 服务器
hostnamectl  set-hostname  k8s-master-02
# 修改 172.16.156.3 服务器
hostnamectl  set-hostname  k8s-master-03

# 修改 172.16.156.4 服务器
hostnamectl  set-hostname  k8s-node-01
# 修改 172.16.156.5 服务器
hostnamectl  set-hostname  k8s-node-02
```



### 关闭防火墙服务

> 停止并禁用防火墙

```
systemctl stop firewalld
systemctl disable firewalld
```

### 关闭并禁用SELinux

```
# 若当前启用了 SELinux 则需要临时设置其当前状态为 permissive
setenforce 0

# 编辑／etc/sysconfig selinux 文件，以彻底禁用 SELinux
sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config

# 查看selinux状态
getenforce
```

> 如果为permissive，则执行reboot重新启动即可

### 禁用 Swap 设备

kubeadm 默认会预先检当前主机是否禁用了 Swap 设备，并在未用时强制止部署 过程因此，在主机内存资惊充裕的条件下，需要禁用所有的 Swap 设备

```
# 关闭当前已启用的所有 Swap 设备
swapoff -a && sysctl -w vm.swappiness=0
# 编辑 fstab 配置文件，注释掉标识为 Swap 设备的所有行
vi /etc/fstab
```

```
UUID=d644ade9-2d1c-49fb-80a5-0c05258b987e /                       xfs     defaults        0 0
UUID=20475828-7ec0-4056-a9cc-659e1b8d5e5c /boot                   xfs     defaults        0 0
```

### 设置系统参数

> 设置允许路由转发，不对bridge的数据进行处理

创建 /etc/sysctl.d/k8s.conf 文件

```
vim /etc/sysctl.d/k8s.conf
```

加入下面内容:

```
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
```

挂载br_netfilter

```
modprobe br_netfilter
```

生效配置文件

```
sysctl -p /etc/sysctl.d/k8s.conf
```

> sysctl命令：用于运行时配置内核参数

查看是否生成相关文件

```
ls /proc/sys/net/bridge
```

### 资源配置文件

/etc/security/limits.conf 是 Linux 资源使用配置文件，用来限制用户对系统资源的使用

```
echo "* soft nofile 65536" >> /etc/security/limits.conf
echo "* hard nofile 65536" >> /etc/security/limits.conf
echo "* soft nproc 65536"  >> /etc/security/limits.conf
echo "* hard nproc 65536"  >> /etc/security/limits.conf
echo "* soft  memlock  unlimited"  >> /etc/security/limits.conf
echo "* hard memlock  unlimited"  >> /etc/security/limits.conf
```

### 安装依赖包以及相关工具

```
yum install -y epel-release
yum install -y yum-utils device-mapper-persistent-data lvm2 net-tools conntrack-tools wget vim  ntpdate libseccomp libtool-ltdl
```

> 

# 二、安装haproxy

> 此处的haproxy为apiserver提供反向代理，haproxy将所有请求轮询转发到每个master节点上。相对于仅仅使用keepalived主备模式仅单个master节点承载流量，这种方式更加合理、健壮。

## 1、yum安装haproxy

```
yum install -y haproxy
```

## 2、配置haproxy

```
cat > /etc/haproxy/haproxy.cfg << EOF
#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    # to have these messages end up in /var/log/haproxy.log you will
    # need to:
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
    log         127.0.0.1 local2
    
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon
       
    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats
#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000
#---------------------------------------------------------------------
# kubernetes apiserver frontend which proxys to the backends
#---------------------------------------------------------------------
frontend kubernetes-apiserver
    mode                 tcp
    bind                 *:16443
    option               tcplog
    default_backend      kubernetes-apiserver
#---------------------------------------------------------------------
# round robin balancing between the various backends
#---------------------------------------------------------------------
backend kubernetes-apiserver
    mode        tcp
    balance     roundrobin
    server      master01   172.16.156.1:6443 check
    server      master02   172.16.156.2:6443 check
    server      master03   172.16.156.3:6443 check
#---------------------------------------------------------------------
# collection haproxy statistics message
#---------------------------------------------------------------------
listen stats
    bind                 *:1080
    stats auth           admin:awesomePassword
    stats refresh        5s
    stats realm          HAProxy\ Statistics
    stats uri            /admin?stats

EOF
```

> haproxy配置在其他master节点上(192.168.2.12和192.168.2.13)相同

## 3、启动并检测haproxy

```
# 设置开机启动
systemctl enable haproxy
# 开启haproxy
systemctl start haproxy
# 查看启动状态
systemctl status haproxy
```

## 4、检测haproxy端口

```
ss -lnt | grep -E "16443|1080"
```

显示：

```
[root@k8s-master-01 ~]# ss -lnt | grep -E "16443|1080"
LISTEN     0      3000         *:1080                     *:*                  
LISTEN     0      3000         *:16443                    *:*   
```

### 确认一下iptables

确认一下iptables filter表中FOWARD链的默认策略(pllicy)为ACCEPT。

```
iptables -nvL
```

显示：

```
Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 DOCKER-USER  all  --  *      *       0.0.0.0/0            0.0.0.0/0
    0     0 DOCKER-ISOLATION-STAGE-1  all  --  *      *       0.0.0.0/0            0.0.0.0/0
    0     0 ACCEPT     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
    0     0 DOCKER     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0
    0     0 ACCEPT     all  --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0
    0     0 ACCEPT     all  --  docker0 docker0  0.0.0.0/0            0.0.0.0/0
```

> Docker从1.13版本开始调整了默认的防火墙规则，禁用了iptables filter表中FOWARD链，这样会引起Kubernetes集群中跨Node的Pod无法通信。但这里通过安装docker 1806，发现默认策略又改回了ACCEPT，这个不知道是从哪个版本改回的，因为我们线上版本使用的1706还是需要手动调整这个策略的。

# 三、安装kubeadm、kubelet

## 1、配置可用的国内yum源用于安装：

```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

## 2、安装kubelet

- 需要在每台机器上都安装以下的软件包：

- - **kubeadm**: 用来初始化集群的指令。
  - **kubelet**: 在集群中的每个节点上用来启动 pod 和 container 等。
  - **kubectl**: 用来与集群通信的命令行工具。

### 查看kubelet版本列表

```
yum list kubelet --showduplicates | sort -r
```

### 安装kubelet

```
yum install -y kubelet-1.18.6-0.x86_64
```

### 检查 Docker 与 kubelet 所使用的 cgroup 驱动是否相同

出于稳定性考虑，官方建议两者均使用 systemd 驱动。

```
sed -i "/^KUBELET_EXTRA_ARGS=/cKUBELET_EXTRA_ARGS=\"--cgroup-driver=systemd\"" /etc/sysconfig/kubelet

[ $(systemctl is-enabled kubelet.service) != enabled ] && systemctl enable kubelet
```

### 启动kubelet并设置开机启动

```
systemctl enable kubelet
systemctl start kubelet
```

### 检查状态

检查状态,发现是failed状态

```
[root@k8s-master-01 ~]# systemctl status kubelet.service 
● kubelet.service - kubelet: The Kubernetes Node Agent
   Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; vendor preset: disabled)
  Drop-In: /usr/lib/systemd/system/kubelet.service.d
           └─10-kubeadm.conf
   Active: activating (auto-restart) (Result: exit-code) since 二 2021-07-06 10:09:45 CST; 2s ago
     Docs: https://kubernetes.io/docs/
  Process: 24785 ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS (code=exited, status=255)
 Main PID: 24785 (code=exited, status=255)

7月 06 10:09:45 k8s-master-01 systemd[1]: Unit kubelet.service entered failed state.
7月 06 10:09:45 k8s-master-01 systemd[1]: kubelet.service failed.

```

正常，kubelet会10秒重启一次，等初始化master节点后即可正常

```
systemctl status kubelet
```

## 3、安装kubeadm

> 负责初始化集群

### 查看kubeadm版本列表

```
yum list kubeadm --showduplicates | sort -r
```

### 安装kubeadm

```
yum install -y kubeadm-1.18.6-0.x86_64
```

> 安装 kubeadm 时候会默认安装 kubectl ，所以不需要单独安装kubectl

## 4、重启服务器

为了防止发生某些未知错误，这里我们重启下服务器，方便进行后续操作

```
reboot
```

# 四、初始化第一个kubernetes master节点

## 1、创建kubeadm配置的yaml文件

`cat /root/18/kubeadm.conf`

```yaml
apiServer:
  extraArgs:
    authorization-mode: Node,RBAC
    enable-admission-plugins: LimitRanger
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint: 172.16.156.1:16443
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: k8s.gcr.io
kind: ClusterConfiguration
kubernetesVersion: v1.18.6
networking:
  dnsDomain: cluster.local
  podSubnet: 192.168.0.0/16
  serviceSubnet: 10.96.0.0/12
scheduler: {}
```

### 列出初始化时所需要的镜像

`kubeadm config images list --config /root/18/kubeadm.conf`

显示如下：

	k8s.gcr.io/kube-apiserver:v1.18.6
	k8s.gcr.io/kube-controller-manager:v1.18.6
	k8s.gcr.io/kube-scheduler:v1.18.6
	k8s.gcr.io/kube-proxy:v1.18.6
	k8s.gcr.io/pause:3.2
	k8s.gcr.io/etcd:3.4.3-0
	k8s.gcr.io/coredns:1.6.7

镜像都放在了master1的/root/18下，需要把下面的镜像拷贝到其他机器，先load进去，然后再tag一下


```
docker tag registry.aliyuncs.com/google_containers/kube-apiserver:v1.18.6  k8s.gcr.io/kube-apiserver:v1.18.6

docker tag  registry.aliyuncs.com/google_containers/kube-controller-manager:v1.18.6   k8s.gcr.io/kube-controller-manager:v1.18.6

docker tag registry.aliyuncs.com/google_containers/kube-scheduler:v1.18.6   k8s.gcr.io/kube-scheduler:v1.18.6

docker tag registry.aliyuncs.com/google_containers/kube-proxy:v1.18.6  k8s.gcr.io/kube-proxy:v1.18.6

docker tag registry.aliyuncs.com/google_containers/pause:3.2  k8s.gcr.io/pause:3.2

docker tag  registry.aliyuncs.com/google_containers/etcd:3.4.3-0  k8s.gcr.io/etcd:3.4.3-0

docker tag registry.aliyuncs.com/google_containers/coredns:1.6.7  k8s.gcr.io/coredns:1.6.7
```

## 2、初始化第一个master节点

```
kubeadm init --config /root/18/kubeadm.conf --upload-certs
```

日志：

```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join 172.16.156.1:16443 --token k31zx5.6o6cevc1fufmp1dd \
    --discovery-token-ca-cert-hash sha256:6c1ba3aa102bc5df6a46daef7d0274b858a03e843ff69560263eb77ec6a61cab \
    --control-plane --certificate-key 8fb9d8b1e367d089514d973f3916c76d146da8746e6ab93ecc31d74f370d4fd9

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.16.156.1:16443 --token k31zx5.6o6cevc1fufmp1dd \
    --discovery-token-ca-cert-hash sha256:6c1ba3aa102bc5df6a46daef7d0274b858a03e843ff69560263eb77ec6a61cab

```

来让节点加入集群

## 3、配置kubectl环境变量

配置环境变量

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
### 3.1、初始化第二、三个master节点从上边复制去掉斜线 再执行

```
kubeadm join 172.16.156.1:16443 --token k31zx5.6o6cevc1fufmp1dd 
 --discovery-token-ca-cert-hash sha256:6c1ba3aa102bc5df6a46daef7d0274b858a03e843ff69560263eb77ec6a61cab  --control-plane --certificate-key 8fb9d8b1e367d089514d973f3916c76d146da8746e6ab93ecc31d74f370d4fd9
```

### 3.2、初始化两个node节点 从上边复制最后的join命令 去掉斜线 再执行

```
kubeadm join 172.16.156.1:16443 --token k31zx5.6o6cevc1fufmp1dd \
    --discovery-token-ca-cert-hash sha256:6c1ba3aa102bc5df6a46daef7d0274b858a03e843ff69560263eb77ec6a61cab
```

## 4、查看组件状态

```
kubectl get cs
```

显示：

```
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health": "true"}
```

> controller-manager/scheduler状态为unhealthy也不影响，若非要解决，可参考：
>
> https://www.cnblogs.com/wuliping/p/13780147.html

查看pod状态

```
kubectl get pods --namespace=kube-system
```

显示：

```
[root@k8s-master-01 .kube]# kubectl get pods --namespace=kube-system
NAME                                    READY   STATUS              RESTARTS   AGE
coredns-66bff467f8-qnqjq                0/1     ContainerCreating   0          13m
coredns-66bff467f8-rlmz6                0/1     ContainerCreating   0          13m
etcd-k8s-master-01                      1/1     Running             0          13m
etcd-k8s-master-02                      1/1     Running             0          5m24s
etcd-k8s-master-03                      1/1     Running             0          5m14s
kube-apiserver-k8s-master-01            1/1     Running             0          13m
kube-apiserver-k8s-master-02            1/1     Running             0          5m26s
kube-apiserver-k8s-master-03            1/1     Running             0          4m19s
kube-controller-manager-k8s-master-01   1/1     Running             0          2m13s
kube-controller-manager-k8s-master-02   1/1     Running             0          5m26s
kube-controller-manager-k8s-master-03   1/1     Running             0          4m11s
kube-proxy-2ffp2                        1/1     Running             0          5m26s
kube-proxy-8hlzc                        1/1     Running             0          5m21s
kube-proxy-pphcn                        1/1     Running             0          13m
kube-scheduler-k8s-master-01            1/1     Running             0          2m33s
kube-scheduler-k8s-master-02            1/1     Running             0          5m25s
kube-scheduler-k8s-master-03            1/1     Running             0          4m2s

```

可以看到coredns没有启动，这是由于还没有配置网络插件，接下来配置下后再重新查看启动状态

# 五、安装网络插件

## Install Calico

calico文件已经放在了master1  /root/18下，直接执行

```
kubectl apply -f /root/18/calico.yaml
```

没有报错即可

## 查看集群状态

```
kubectl get nodes
```

```
[root@k8s-master-01 ~]# kubectl get nodes
NAME            STATUS   ROLES    AGE   VERSION
k8s-master-01   Ready    master   28m   v1.18.6
k8s-master-02   Ready    master   20m   v1.18.6
k8s-master-03   Ready    master   20m   v1.18.6
k8s-node-01     Ready    <none>   12m   v1.18.6
k8s-node-02     Ready    <none>   12m   v1.18.6

```


# 六、安装一个nginx进行集群内访问

在k8s-mastre-01上的/root/18/目录下直接执行

```
kubectl apply -f /root/18/nginx.yaml
```

## 查看默认命名空间下的所有资源

```
kubectl get all
```

```
[root@k8s-master-01 ~]# kubectl get all
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-674bd4fbf8-9t8wl   1/1     Running   0          7m30s

NAME                             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/kubernetes               ClusterIP   10.96.0.1       <none>        443/TCP        29m
service/nginx-service-nodeport   NodePort    10.101.155.96   <none>        80:30970/TCP   7m30s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1/1     1            1           7m30s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-674bd4fbf8   1         1         1       7m30s

```

## 测试

通过kubectl get pod 查看，通过curl 进行测试，能够返回200表示部署成功

```
1、kubectl get pod
2、kubectl get svc
3、curl 172.16.156.1:80
```


# 七、集群重置

- 在k8s-master-01上执行删除其他四个节点：

```
kubectl drain <node name> --delete-local-data --force --ignore-daemonsets
kubectl delete node <node name>
```

- 然后在所有的节点执行

```
kubeadm reset
```