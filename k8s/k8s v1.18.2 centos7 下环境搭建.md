# [k8s v1.18.2 centos7 下环境搭建](https://www.cnblogs.com/360minitao/p/13243280.html)



##  

## 准备

服务器：3台机器——1台主、2台工作节点，可以使用virtualbox 搭建虚拟机

| 主机名   | centos version | ip             | docker version | flannel version | 主机配置  | 备注 |
| -------- | -------------- | -------------- | -------------- | --------------- | --------- | ---- |
| master01 | 7.7.1908       | 192.168.56.101 | 19.03.8        | v0.12.0-amd64   | 2P 2G 20G |      |
| node01   | 7.7.1908       | 192.168.56.102 | 19.03.8        | v0.12.0-amd64   | 2P 2G 20G |      |
| node02   | 7.7.1908       | 192.168.56.103 | 19.03.8        | v0.12.0-amd64   | 2P 2G 20G |      |

k8s版本

| 主机名   | kubelet version | kubeadm version | kubectl version | 备注 |
| -------- | --------------- | --------------- | --------------- | ---- |
| master01 | v1.18.2         | v1.18.2         | v1.18.2         |      |
| node01   | v1.18.2         | v1.18.2         | v1.18.2         |      |
| node01   | v1.18.2         | v1.18.2         | v1.18.2         |      |

## 基本设置：

### 关闭防火墙：

```
systemctl stop firewalld & systemctl disable firewalld
```

### 关闭swap

```bash
#临时关闭
swapoff -a
#永久关闭,重启后生效
vi /etc/fstab
#注释以下代码
/dev/mapper/centos-swap swap ...
```

### 关闭selinux

```bash
#获取状态
getenforce
#暂时关闭
setenforce 0
#永久关闭 需重启
vi /etc/sysconfig/selinux
#修改以下参数，设置为disable
SELINUX=disabled
```

### 修改网络配置

```bash
# 所有机器上都要进行
$ cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
#
$ sysctl --system
```

### 统一时间【如果需要】

```bash
#统一时区，为上海时区
ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
bash -c "echo 'Asia/Shanghai' > /etc/timezone"
#统一使用阿里服务器进行时间更新
yum install -y  ntpdate					#安装ntpdate工具
ntpdate ntp1.aliyun.com					#更新时间
```

### 安装docker:

- 1,删除原有的docker组件

```bash
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

- 2,配置系统docker源

```bash
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
## 注意：此处更换了阿里的源，适用国内用户
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

- 3,查看docker安装列表，选择并安装

```bash
sudo yum list docker-ce --showduplicates
# 此处直接安装最新版本的docker-ce
sudo yum install -y docker-ce

# 注：如果要安装指定的版本可以参考下边的命令
sudo yum install -y docker-ce-3:19.03.8-3.el7.x86_64 
```

- 4.启动docker

```bash
sudo systemctl  enable docker &&  systemctl  start docker
```

- 5.更换镜像仓库源 [阿里云docker仓库](https://dev.aliyun.com/search.html)

```bash
# 进入阿里云帐号，依次进入：控制台——容器镜像服务(可以搜索到)——镜像中心——镜像加速器。镜像加速器中获取到加速器地址: "https://xxxxxxx.mirror.aliyuncs.com"

#linux下默认文件为/etc/docker/daemon.json，添加下列仓库
$ sudo vi /etc/docker/daemon.json

{
"registry-mirrors": ["https://xxxxxxx.mirror.aliyuncs.com"]
}

# 重启docker使其生效
$ sudo systemctl  restart docker 
```

### 安装K8S组件：

- 1,更新K8S源

```bash
$ cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
        http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

- 2, kubelet、kubeadm、kubectl

```bash
$ yum install -y kubelet kubeadm kubectl
# 启动kubelet 服务
$  systemctl  enable kubelet &&  systemctl  start kubelet
```

### host配置

```bash
#分别在服务器上修改hostname,使用hostnamectl命令，或者直接修改/etc/hostname 文件
hostnamectl set-hostname  master01
hostnamectl set-hostname  node01
hostnamectl set-hostname  node02

#每台机器都执行
# cat >> /etc/hosts << EOF
192.168.56.101    master01
192.168.56.102    node01
192.168.56.103    node02
EOF

# check
# cat /etc/hosts
```

### 验证MAC地址确保唯一

```bash
# enp0s3 网卡设备名称，根据自己实际情况进行改动
# cat /sys/class/net/enp0s3/address
# cat /sys/class/dmi/id/product_uuid
```

## 配置master服务器

### 配置k8s初始化文件

[k8s config 命令参考](https://kubernetes.io/zh/docs/reference/setup-tools/kubeadm/kubeadm-config/)

master节点下 生成默认配置文件

```shell
$ kubeadm config print init-defaults > init-kubeadm.conf
```

修改init-kubeadm.conf 主要参数

```bash
#imageRepository: k8s.gcr.io 更换k8s镜像仓库
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
# kubernetesVersion: v1.18.0  修改为版本 v1.18.2
kubernetesVersion: v1.18.2
# localAPIEndpointc，advertiseAddress为master-ip ，port默认不修改
localAPIEndpoint:
  advertiseAddress: 192.168.56.101  #此处为master的IP
  bindPort: 6443
# 配置子网络
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
  podSubnet: 10.244.0.0/16			#添加pod子网络,使用的是flannel网络
```

### 拉取下载k8s组件

```bash
#查看安装时需要的镜像文件列表
$ kubeadm  config images list --config init-kubeadm.conf

k8s.gcr.io/kube-apiserver:v1.18.2
k8s.gcr.io/kube-controller-manager:v1.18.2
k8s.gcr.io/kube-scheduler:v1.18.2
k8s.gcr.io/kube-proxy:v1.18.2
k8s.gcr.io/pause:3.2
k8s.gcr.io/etcd:3.4.3-0
k8s.gcr.io/coredns:1.6.7
#下载，根据配置文件进行镜像下载，如果此处下载比较缓慢可以考虑更换配置文件中的imageRepository参数
$ kubeadm config images pull --config init-kubeadm.conf
```

### 初始化k8s

```bash
# init 
#如果kubeadm init过了，此时需要加个参数来忽略到这些：--ignore-preflight-errors=all
$ kubeadm init --config init-kubeadm.conf

# 启动后可以根据提示执行下列命令，并记录john token
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 此处要记录下 join语句，如果join token忘记，则需要执行下边命令重新生成
$ kubeadm token create --print-join-command
kubeadm join 192.168.56.101:6443 --token abcdeff.zlo1o2t9i8e53whd     --discovery-token-ca-cert-hash sha256:3613c3bc855b5d9e1555b99ebd21f48873d20353aeb8c2c64e73d8c7597d43f9


#--------------------以下为均一些状态查看命令，可以不执行--------------------
# 查看启动状态
[root@master01 kube]# kubectl get nodes
NAME       STATUS     ROLES    AGE   VERSION
master01   NotReady   master   16m   v1.18.2
[root@master01 kube]# kubectl get cs
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok                  
etcd-0               Healthy   {"health":"true"}  
# 查看具体初始化情况
$ kubectl get pods -n kube-system -o wide
NAME                               READY   STATUS    RESTARTS   AGE
coredns-546565776c-6kbxw           1/1     Running   2          16h
coredns-546565776c-bmfgz           1/1     Running   2          16h
etcd-master01                      1/1     Running   9          16h
kube-apiserver-master01            1/1     Running   9          16h
kube-controller-manager-master01   1/1     Running   9          16h
kube-flannel-ds-amd64-8qphl        1/1     Running   3          14h
kube-flannel-ds-amd64-c8hrq        1/1     Running   3          13h
kube-flannel-ds-amd64-lcmhs        1/1     Running   2          16h
kube-proxy-6wv5w                   1/1     Running   3          13h
kube-proxy-f4x2t                   1/1     Running   4          16h
kube-proxy-m464b                   1/1     Running   13         14h
kube-scheduler-master01            1/1     Running   10         16h
# 查看 kube-flannel-ds-amd64-c8hrq 状态
$ kubectl describe pod kube-flannel-ds-amd64-c8hrq -n kube-system
```

### flannel网络配置

```bash
 #下载 配置文件
$ wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
#下载
sed -i 's/quay.io/quay-mirror.qiniu.com/' kube-flannel.yml 修改镜像地址(原因quay.io镜像访问不了)

$ kubectl apply -f kube-flannel.yml


# 问题解决：
# 此处如果没法下载配置文件kube-flannel.yml，raw.githubusercontent.com访问拒绝
# 通过IPAddress.com 查询真实IP,输入raw.githubusercontent.com查询到真实IP地址(例如:199.232.4.133)
# 添加到hosts中，
$ sudo vi /etc/hosts 
199.232.4.133 raw.githubusercontent.com
```

## 配置node

node节点 初始化内容

```bash
#拷贝 master机器上 $HOME/.kube/config 到node节点上
scp $HOME/.kube/config root@node01:~/
scp $HOME/.kube/config root@node02:~/
#分别在node01和node02上执行下边命令
#不然执行kubectl 会报错
#error: no configuration has been provided, try setting KUBERNETES_MASTER environment variable
mkdir -p $HOME/.kube
sudo mv $HOME/config $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 节点注册方法1：直接使用命令节点注册【推荐】    

```bash
#直接使用指令加入
$ kubeadm join 192.168.56.101:6443 --token abcdeff.zlo1o2t9i8e53whd     --discovery-token-ca-cert-hash sha256:3613c3bc855b5d9e1555b99ebd21f48873d20353aeb8c2c64e73d8c7597d43f9
# token加入语句忘记了可以在master上使用下边命令进行生成
$ kubeadm token create --print-join-command
```

### 节点注册方法2：使用 join-config.yml 进行节点注册

```bash
# node 节点上生成 join-config.yml，并对主要的参数做修改
$ kubeadm config print join-defaults > join-config.yaml

# join-config.yaml 内容
 apiVersion: kubeadm.k8s.io/v1beta2
caCertPath: /etc/kubernetes/pki/ca.crt
discovery:
  bootstrapToken:
    apiServerEndpoint: kube-apiserver:6443 # 修改为master机器ip:port，192.168.56.101:6443
    token: abcdef.0123456789abcdef		   # 修改为真实token
    unsafeSkipCAVerification: true
  timeout: 5m0s
  tlsBootstrapToken: abcdef.0123456789abcdef # 修改为真实token
kind: JoinConfiguration
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: node01								# 根据实际节点名进行修改
  taints: null

#使用配置文件加入
$ kubeadm join  --config=join-config.yaml 
```

### 添加flannel网络

```bash
 #下载 配置文件
$ wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
#或者拷贝 master机器上的文件
scp root@master01:~/workspace/kube/kube-flannel.yml .
#下载
$ kubectl apply -f kube-flannel.yml

# 查看节点添加状况
# kubectl  get nodes 
NAME       STATUS   ROLES    AGE   VERSION
master01   Ready    master   16h   v1.18.2
node01     Ready    <none>   12h   v1.18.2
node02     Ready    <none>   12h   v1.18.2
```

## 常见问题

### node节点NotReady

```bash
# 查看pod 状态
$ kubectl get pods -n kube-system -0 wide
NAME                             READY   STATUS     RESTARTS   AGE     IP               NODE     NOMINATED NODE   READINESS GATES
kube-flannel-ds-amd64-gtlwv      1/1     Running    4          4h23m   192.168.56.101   master01   <none>           <none>
kube-flannel-ds-amd64-m78z2      0/1     Init:0/1   0          3h13m   192.168.56.103   node02    <none>           <none>

# Init:0/1 错误，查看详情
$ kubectl describe pod kube-flannel-ds-amd64-m78z2 -n kube-system

# 一般都是flannel的docker镜像没有下载成功
# 可以在node2上，执行docker images 查看是否有flannel镜像，如果没有手动下载
```

### node节点删除重新加入

```bash
# -----重新加入注册
# 删除指定node
$ kubectl delete node node01
# 删除对应的配置文件以及证书
$ rm -rf /etc/kubernetes/kubelet.conf /etc/kubernetes/pki/ca.crt &&  systemctl restart docker kubelet
# 重新执行注册语句
```

[节点注册语句](https://www.cnblogs.com/climbsnail/p/12821799.html#jump)

## 附：

- [raw.githubusercontent.com 无法访问解决](https://www.ioiox.com/archives/62.html)
- 原文<https://www.cnblogs.com/climbsnail/p/12821799.html>

## 准备

服务器：3台机器——1台主、2台工作节点，可以使用virtualbox 搭建虚拟机

| 主机名   | centos version | ip             | docker version | flannel version | 主机配置  | 备注 |
| -------- | -------------- | -------------- | -------------- | --------------- | --------- | ---- |
| master01 | 7.7.1908       | 192.168.56.101 | 19.03.8        | v0.12.0-amd64   | 2P 2G 20G |      |
| node01   | 7.7.1908       | 192.168.56.102 | 19.03.8        | v0.12.0-amd64   | 2P 2G 20G |      |
| node02   | 7.7.1908       | 192.168.56.103 | 19.03.8        | v0.12.0-amd64   | 2P 2G 20G |      |

k8s版本

| 主机名   | kubelet version | kubeadm version | kubectl version | 备注 |
| -------- | --------------- | --------------- | --------------- | ---- |
| master01 | v1.18.2         | v1.18.2         | v1.18.2         |      |
| node01   | v1.18.2         | v1.18.2         | v1.18.2         |      |
| node01   | v1.18.2         | v1.18.2         | v1.18.2         |      |

## 基本设置：

### 关闭防火墙：

```
systemctl stop firewalld & systemctl disable firewalld
```

### 关闭swap

```bash
#临时关闭
swapoff -a
#永久关闭,重启后生效
vi /etc/fstab
#注释以下代码
/dev/mapper/centos-swap swap ...
```

### 关闭selinux

```bash
#获取状态
getenforce
#暂时关闭
setenforce 0
#永久关闭 需重启
vi /etc/sysconfig/selinux
#修改以下参数，设置为disable
SELINUX=disabled
```

### 修改网络配置

```bash
# 所有机器上都要进行
$ cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
#
$ sysctl --system
```

### 统一时间【如果需要】

```bash
#统一时区，为上海时区
ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
bash -c "echo 'Asia/Shanghai' > /etc/timezone"
#统一使用阿里服务器进行时间更新
yum install -y  ntpdate					#安装ntpdate工具
ntpdate ntp1.aliyun.com					#更新时间
```

### 安装docker:

- 1,删除原有的docker组件

```bash
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

- 2,配置系统docker源

```bash
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
## 注意：此处更换了阿里的源，适用国内用户
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

- 3,查看docker安装列表，选择并安装

```bash
sudo yum list docker-ce --showduplicates
# 此处直接安装最新版本的docker-ce
sudo yum install -y docker-ce

# 注：如果要安装指定的版本可以参考下边的命令
sudo yum install -y docker-ce-3:19.03.8-3.el7.x86_64 
```

- 4.启动docker

```bash
sudo systemctl  enable docker &&  systemctl  start docker
```

- 5.更换镜像仓库源 [阿里云docker仓库](https://dev.aliyun.com/search.html)

```bash
# 进入阿里云帐号，依次进入：控制台——容器镜像服务(可以搜索到)——镜像中心——镜像加速器。镜像加速器中获取到加速器地址: "https://xxxxxxx.mirror.aliyuncs.com"

#linux下默认文件为/etc/docker/daemon.json，添加下列仓库
$ sudo vi /etc/docker/daemon.json

{
"registry-mirrors": ["https://xxxxxxx.mirror.aliyuncs.com"]
}

# 重启docker使其生效
$ sudo systemctl  restart docker 
```

### 安装K8S组件：

- 1,更新K8S源

```bash
$ cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
        http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

- 2, kubelet、kubeadm、kubectl

```bash
$ yum install -y kubelet kubeadm kubectl
# 启动kubelet 服务
$  systemctl  enable kubelet &&  systemctl  start kubelet
```

### host配置

```bash
#分别在服务器上修改hostname,使用hostnamectl命令，或者直接修改/etc/hostname 文件
hostnamectl set-hostname  master01
hostnamectl set-hostname  node01
hostnamectl set-hostname  node02

#每台机器都执行
# cat >> /etc/hosts << EOF
192.168.56.101    master01
192.168.56.102    node01
192.168.56.103    node02
EOF

# check
# cat /etc/hosts
```

### 验证MAC地址确保唯一

```bash
# enp0s3 网卡设备名称，根据自己实际情况进行改动
# cat /sys/class/net/enp0s3/address
# cat /sys/class/dmi/id/product_uuid
```

## 配置master服务器

### 配置k8s初始化文件

[k8s config 命令参考](https://kubernetes.io/zh/docs/reference/setup-tools/kubeadm/kubeadm-config/)

master节点下 生成默认配置文件

```shell
$ kubeadm config print init-defaults > init-kubeadm.conf
```

修改init-kubeadm.conf 主要参数

```bash
#imageRepository: k8s.gcr.io 更换k8s镜像仓库
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
# kubernetesVersion: v1.18.0  修改为版本 v1.18.2
kubernetesVersion: v1.18.2
# localAPIEndpointc，advertiseAddress为master-ip ，port默认不修改
localAPIEndpoint:
  advertiseAddress: 192.168.56.101  #此处为master的IP
  bindPort: 6443
# 配置子网络
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
  podSubnet: 10.244.0.0/16			#添加pod子网络,使用的是flannel网络
```

### 拉取下载k8s组件

```bash
#查看安装时需要的镜像文件列表
$ kubeadm  config images list --config init-kubeadm.conf

k8s.gcr.io/kube-apiserver:v1.18.2
k8s.gcr.io/kube-controller-manager:v1.18.2
k8s.gcr.io/kube-scheduler:v1.18.2
k8s.gcr.io/kube-proxy:v1.18.2
k8s.gcr.io/pause:3.2
k8s.gcr.io/etcd:3.4.3-0
k8s.gcr.io/coredns:1.6.7
#下载，根据配置文件进行镜像下载，如果此处下载比较缓慢可以考虑更换配置文件中的imageRepository参数
$ kubeadm config images pull --config init-kubeadm.conf
```

### 初始化k8s

```bash
# init 
#如果kubeadm init过了，此时需要加个参数来忽略到这些：--ignore-preflight-errors=all
$ kubeadm init --config init-kubeadm.conf

# 启动后可以根据提示执行下列命令，并记录john token
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 此处要记录下 join语句，如果join token忘记，则需要执行下边命令重新生成
$ kubeadm token create --print-join-command
kubeadm join 192.168.56.101:6443 --token abcdeff.zlo1o2t9i8e53whd     --discovery-token-ca-cert-hash sha256:3613c3bc855b5d9e1555b99ebd21f48873d20353aeb8c2c64e73d8c7597d43f9


#--------------------以下为均一些状态查看命令，可以不执行--------------------
# 查看启动状态
[root@master01 kube]# kubectl get nodes
NAME       STATUS     ROLES    AGE   VERSION
master01   NotReady   master   16m   v1.18.2
[root@master01 kube]# kubectl get cs
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok                  
etcd-0               Healthy   {"health":"true"}  
# 查看具体初始化情况
$ kubectl get pods -n kube-system -0 wide
NAME                               READY   STATUS    RESTARTS   AGE
coredns-546565776c-6kbxw           1/1     Running   2          16h
coredns-546565776c-bmfgz           1/1     Running   2          16h
etcd-master01                      1/1     Running   9          16h
kube-apiserver-master01            1/1     Running   9          16h
kube-controller-manager-master01   1/1     Running   9          16h
kube-flannel-ds-amd64-8qphl        1/1     Running   3          14h
kube-flannel-ds-amd64-c8hrq        1/1     Running   3          13h
kube-flannel-ds-amd64-lcmhs        1/1     Running   2          16h
kube-proxy-6wv5w                   1/1     Running   3          13h
kube-proxy-f4x2t                   1/1     Running   4          16h
kube-proxy-m464b                   1/1     Running   13         14h
kube-scheduler-master01            1/1     Running   10         16h
# 查看 kube-flannel-ds-amd64-c8hrq 状态
$ kubectl describe pod kube-flannel-ds-amd64-c8hrq -n kube-system
```

### flannel网络配置

```bash
 #下载 配置文件
$ wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
#下载
$ kubectl apply -f kube-flannel.yml


# 问题解决：
# 此处如果没法下载配置文件kube-flannel.yml，raw.githubusercontent.com访问拒绝
# 通过IPAddress.com 查询真实IP,输入raw.githubusercontent.com查询到真实IP地址(例如:199.232.4.133)
# 添加到hosts中，
$ sudo vi /etc/hosts 
199.232.4.133 raw.githubusercontent.com
```

## 配置node

node节点 初始化内容

```bash
#拷贝 master机器上 $HOME/.kube/config 到node节点上
scp $HOME/.kube/config root@node01:~/
scp $HOME/.kube/config root@node02:~/
#分别在node01和node02上执行下边命令
#不然执行kubectl 会报错
#error: no configuration has been provided, try setting KUBERNETES_MASTER environment variable
mkdir -p $HOME/.kube
sudo mv $HOME/config $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 节点注册方法1：直接使用命令节点注册【推荐】    

```bash
#直接使用指令加入
$ kubeadm join 192.168.56.101:6443 --token abcdeff.zlo1o2t9i8e53whd     --discovery-token-ca-cert-hash sha256:3613c3bc855b5d9e1555b99ebd21f48873d20353aeb8c2c64e73d8c7597d43f9
# token加入语句忘记了可以在master上使用下边命令进行生成
$ kubeadm token create --print-join-command
```

### 节点注册方法2：使用 join-config.yml 进行节点注册

```bash
# node 节点上生成 join-config.yml，并对主要的参数做修改
$ kubeadm config print join-defaults > join-config.yaml

# join-config.yaml 内容
 apiVersion: kubeadm.k8s.io/v1beta2
caCertPath: /etc/kubernetes/pki/ca.crt
discovery:
  bootstrapToken:
    apiServerEndpoint: kube-apiserver:6443 # 修改为master机器ip:port，192.168.56.101:6443
    token: abcdef.0123456789abcdef		   # 修改为真实token
    unsafeSkipCAVerification: true
  timeout: 5m0s
  tlsBootstrapToken: abcdef.0123456789abcdef # 修改为真实token
kind: JoinConfiguration
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: node01								# 根据实际节点名进行修改
  taints: null

#使用配置文件加入
$ kubeadm join  --config=join-config.yaml 
```

### 添加flannel网络

```bash
 #下载 配置文件
$ wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
#或者拷贝 master机器上的文件
scp root@master01:~/workspace/kube/kube-flannel.yml .
#下载
$ kubectl apply -f kube-flannel.yml

# 查看节点添加状况
# kubectl  get nodes 
NAME       STATUS   ROLES    AGE   VERSION
master01   Ready    master   16h   v1.18.2
node01     Ready    <none>   12h   v1.18.2
node02     Ready    <none>   12h   v1.18.2
```

## 常见问题

### node节点NotReady

```bash
# 查看pod 状态
$ kubectl get pods -n kube-system -0 wide
NAME                             READY   STATUS     RESTARTS   AGE     IP               NODE     NOMINATED NODE   READINESS GATES
kube-flannel-ds-amd64-gtlwv      1/1     Running    4          4h23m   192.168.56.101   master01   <none>           <none>
kube-flannel-ds-amd64-m78z2      0/1     Init:0/1   0          3h13m   192.168.56.103   node02    <none>           <none>

# Init:0/1 错误，查看详情
$ kubectl describe pod kube-flannel-ds-amd64-m78z2 -n kube-system

# 一般都是flannel的docker镜像没有下载成功
# 可以在node2上，执行docker images 查看是否有flannel镜像，如果没有手动下载
```

### node节点删除重新加入

```bash
# -----重新加入注册
# 删除指定node
$ kubectl delete node node01
# 删除对应的配置文件以及证书
$ rm -rf /etc/kubernetes/kubelet.conf /etc/kubernetes/pki/ca.crt &&  systemctl restart docker kubelet
# 重新执行注册语句
```

[节点注册语句](https://www.cnblogs.com/climbsnail/p/12821799.html#jump)

## 附：

- [raw.githubusercontent.com 无法访问解决](https://www.ioiox.com/archives/62.html)