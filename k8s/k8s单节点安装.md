## 阅读目录

- [说明](https://www.cnblogs.com/hellxz/p/use-kubeadm-init-kubernetes-cluster.html#%E8%AF%B4%E6%98%8E)
- [环境准备](https://www.cnblogs.com/hellxz/p/use-kubeadm-init-kubernetes-cluster.html#%E7%8E%AF%E5%A2%83%E5%87%86%E5%A4%87)
- [部署docker](https://www.cnblogs.com/hellxz/p/use-kubeadm-init-kubernetes-cluster.html#%E9%83%A8%E7%BD%B2docker)
- [部署kubernetes集群](https://www.cnblogs.com/hellxz/p/use-kubeadm-init-kubernetes-cluster.html#%E9%83%A8%E7%BD%B2kubernetes%E9%9B%86%E7%BE%A4)

## 说明

本文系搭建kubernetes v1.18.5 集群笔记，使用三台虚拟机作为 CentOS 测试机，安装kubeadm、kubelet、kubectl均使用yum安装，网络组件选用的是 flannel

行文中难免出现错误，如果读者有高见，请评论与我交流

如需转载请注明原始出处 <https://www.cnblogs.com/hellxz/p/use-kubeadm-init-kubernetes-cluster.html>

## 环境准备

部署集群没有特殊说明均使用root用户执行命令

### 硬件信息

| ip             | hostname    | mem  | disk | explain          |
| -------------- | ----------- | ---- | ---- | ---------------- |
| 192.168.87.145 | kube-master | 4 GB | 20GB | k8s 控制平台节点 |
| 192.168.87.146 | kube-node1  | 4 GB | 20GB | k8s 执行节点1    |
| 192.168.87.147 | kube-node2  | 4 GB | 20GB | k8s 执行节点2    |

### 软件信息

| software   | version                              |
| ---------- | ------------------------------------ |
| CentOS     | CentOS Linux release 7.7.1908 (Core) |
| Kubernetes | v1.18.5                              |
| Docker     | 19.03.12                             |

### 保证环境正确性

| purpose              | commands                                                     |
| -------------------- | ------------------------------------------------------------ |
| 保证集群各节点互通   | `ping -c 3 <ip>`                                             |
| 保证MAC地址唯一      | `ip link` 或 `ifconfig -a`                                   |
| 保证集群内主机名唯一 | 查询 `hostnamectl status`，修改 `hostnamectl set-hostname <hostname>` |
| 保证系统产品uuid唯一 | `dmidecode -s system-uuid` 或 `sudo cat /sys/class/dmi/id/product_uuid` |

> 修改MAC地址参考命令：
>
> ```bash
> ifconfig eth0 down
> ifconfig eth0 hw ether 00:0C:18:EF:FF:ED
> ifconfig eth0 up
> ```
>
> 如product_uuid不唯一，请考虑重装CentOS系统

### 确保端口开放正常

kube-master节点端口检查：

| Protocol | Direction | Port Range | Purpose                 |
| -------- | --------- | ---------- | ----------------------- |
| TCP      | Inbound   | 6443*      | kube-api-server         |
| TCP      | Inbound   | 2379-2380  | etcd API                |
| TCP      | Inbound   | 10250      | Kubelet API             |
| TCP      | Inbound   | 10251      | kube-scheduler          |
| TCP      | Inbound   | 10252      | kube-controller-manager |

kube-node*节点端口检查：

| Protocol | Direction | Port Range  | Purpose           |
| -------- | --------- | ----------- | ----------------- |
| TCP      | Inbound   | 10250       | Kubelet API       |
| TCP      | Inbound   | 30000-32767 | NodePort Services |

> 如果你对主机的防火墙配置不是很自信，可以关掉防火墙：
>
> ```bash
> systemctl disable --now firewalld
> ```
>
> 或者 清除iptables规则 （慎用）
>
> ```bash
> iptables -F
> ```

### 配置主机互信

分别在**各节点**配置hosts映射：

```bash
cat >> /etc/hosts <<EOF
192.168.87.145 kube-master
192.168.87.146 kube-node1
192.168.87.147 kube-node2
EOF
```

**kube-master**生成ssh密钥，分发公钥到各节点：

```bash
#生成ssh密钥，直接一路回车
ssh-keygen -t rsa
#复制刚刚生成的密钥到各节点可信列表中，需分别输入各主机密码
ssh-copy-id root@kube-master
ssh-copy-id root@kube-node1
ssh-copy-id root@kube-node2
```

### 禁用swap

swap仅当内存不够时会使用硬盘块充当额外内存，硬盘的io较内存差距极大，禁用swap以提高性能

各节点均需执行：

```bash
swapoff -a
sed -i 's/.*swap.*/#&/' /etc/fstab
```

### 关闭 SELinux

关闭 SELinux，否则 kubelet 挂载目录时可能报错 `Permission denied`，可以设置为`permissive`或`disabled`，`permissive` 会提示warn信息

各节点均需执行：

```bash
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
```

### 设置系统时区、同步时间

```bash
timedatectl set-timezone Asia/Shanghai
systemctl enable --now chronyd
```

查看同步状态：

```bash
timedatectl status
```

输出：

```text
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

- `System clock synchronized: yes`，表示时钟已同步；
- `NTP service: active`，表示开启了时钟同步服务；

```bash
# 将当前的 UTC 时间写入硬件时钟
timedatectl set-local-rtc 0
# 重启依赖于系统时间的服务
systemctl restart rsyslog && systemctl restart crond
```

## 部署docker

所有节点均需安装部署docker

### 添加docker yum源

```bash
#安装必要依赖
yum install -y yum-utils device-mapper-persistent-data lvm2
#添加aliyun docker-ce yum源
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
#重建yum缓存
yum makecache fast
```

### 安装docker

```bash
#查看可用docker版本
yum list docker-ce.x86_64 --showduplicates | sort -r
```

![img](https://img2020.cnblogs.com/blog/1149398/202007/1149398-20200703165421246-1856100496.png)

```bash
#安装指定版本docker
yum install -y docker-ce-19.03.12-3.el7
```

> 这里以安装19.03.12版本举例，注意版本号不包含`:`与之前的数字

![img](https://img2020.cnblogs.com/blog/1149398/202007/1149398-20200703165912677-1453268192.png)

### 确保网络模块开机自动加载

```bash
lsmod | grep overlay
lsmod | grep br_netfilter
```

若上面命令无返回值输出或提示文件不存在，需执行以下命令：

```bash
cat > /etc/modules-load.d/docker.conf <<EOF
overlay
br_netfilter
EOF
modprobe overlay
modprobe br_netfilter
```

### 使桥接流量对iptables可见

各节点均需执行：

```bash
cat > /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

验证是否生效，均返回 `1` 即正确

```bash
sysctl -n net.bridge.bridge-nf-call-iptables
sysctl -n net.bridge.bridge-nf-call-ip6tables
```

### 配置docker

```bash
mkdir /etc/docker
#修改cgroup驱动为systemd[k8s官方推荐]、限制容器日志量、修改存储类型，最后的docker家目录可修改
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ],
  "registry-mirrors": ["https://7uuu3esz.mirror.aliyuncs.com"],
  "data-root": "/data/docker"
}
EOF
#添加开机自启，立即启动
systemctl enable --now docker
```

### 验证docker是否正常

```bash
#查看docker信息，判断是否与配置一致
docker info
#hello-docker测试
docker run --rm hello-world
#删除测试image
docker rmi hello-world
```

![img](https://img2020.cnblogs.com/blog/1149398/202007/1149398-20200703172430369-1826136717.png)

### 添加用户到docker组

非root用户，无需sudo即可使用docker命令

```bash
#添加用户到docker组
usermod -aG docker <USERNAME>
#当前会话立即更新docker组
newgrp docker
```

## 部署kubernetes集群

未特殊说明，各节点均需执行如下步骤

### 添加kubernetes源

```bash
cat > /etc/yum.repos.d/kubernetes.repo <<EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
#重建yum缓存，输入y添加证书认证
yum makecache fast
```

### 安装kubeadm、kubelet、kubectl

各节点均需安装kubeadm、kubelet，kubectl仅kube-master节点需安装（作为worker节点，kubectl无法使用，可以不装）

```bash
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
systemctl enable --now kubelet
```

### 配置自动补全命令

```bash
#安装bash自动补全插件
yum install bash-completion -y
#设置kubectl与kubeadm命令补全，下次login生效
kubectl completion bash >/etc/bash_completion.d/kubectl
kubeadm completion bash > /etc/bash_completion.d/kubeadm
```

### 预拉取kubernetes镜像

由于国内网络因素，kubernetes镜像需要从mirrors站点或通过dockerhub用户推送的镜像拉取

```bash
#查看指定k8s版本需要哪些镜像
kubeadm config images list --kubernetes-version v1.18.5
```

![img](https://img2020.cnblogs.com/blog/1149398/202007/1149398-20200703181519490-1098488138.png)

> 另因阿里云的镜像暂时还没更新到v1.18.5版本，所以通过dockerhub上拉取，目前阿里云最新同步版本是v1.18.3，想通过v1.18.3版本拉取镜像请参考 <<https://www.cnblogs.com/hellxz/p/13204093.html>

在 `/root/k8s` 目录下，新建脚本`get-k8s-images.sh`，内容如下：

```bash
#!/bin/bash
# Script For Quick Pull K8S Docker Images
# by Hellxz Zhang <hellxz001@foxmail.com>

KUBE_VERSION=v1.18.5
PAUSE_VERSION=3.2
CORE_DNS_VERSION=1.6.7
ETCD_VERSION=3.4.3-0

# pull kubernetes images from hub.docker.com
docker pull kubeimage/kube-proxy-amd64:$KUBE_VERSION
docker pull kubeimage/kube-controller-manager-amd64:$KUBE_VERSION
docker pull kubeimage/kube-apiserver-amd64:$KUBE_VERSION
docker pull kubeimage/kube-scheduler-amd64:$KUBE_VERSION
# pull aliyuncs mirror docker images
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause:$PAUSE_VERSION
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:$CORE_DNS_VERSION
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:$ETCD_VERSION

# retag to k8s.gcr.io prefix
docker tag kubeimage/kube-proxy-amd64:$KUBE_VERSION  k8s.gcr.io/kube-proxy:$KUBE_VERSION
docker tag kubeimage/kube-controller-manager-amd64:$KUBE_VERSION k8s.gcr.io/kube-controller-manager:$KUBE_VERSION
docker tag kubeimage/kube-apiserver-amd64:$KUBE_VERSION k8s.gcr.io/kube-apiserver:$KUBE_VERSION
docker tag kubeimage/kube-scheduler-amd64:$KUBE_VERSION k8s.gcr.io/kube-scheduler:$KUBE_VERSION
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/pause:$PAUSE_VERSION k8s.gcr.io/pause:$PAUSE_VERSION
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:$CORE_DNS_VERSION k8s.gcr.io/coredns:$CORE_DNS_VERSION
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:$ETCD_VERSION k8s.gcr.io/etcd:$ETCD_VERSION

# untag origin tag, the images won't be delete.
docker rmi kubeimage/kube-proxy-amd64:$KUBE_VERSION
docker rmi kubeimage/kube-controller-manager-amd64:$KUBE_VERSION
docker rmi kubeimage/kube-apiserver-amd64:$KUBE_VERSION
docker rmi kubeimage/kube-scheduler-amd64:$KUBE_VERSION
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/pause:$PAUSE_VERSION
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:$CORE_DNS_VERSION
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:$ETCD_VERSION
```

脚本添加可执行权限，执行脚本拉取镜像：

```bash
chmod +x get-k8s-images.sh
./get-k8s-images.sh
```

拉取完成，执行 `docker images` 查看镜像

![img](https://img2020.cnblogs.com/blog/1149398/202007/1149398-20200703183516645-448394415.png)

### 初始化kube-master

仅 kube-master 节点需要执行此步骤

**修改kubelet配置默认cgroup driver**

```bash
cat > /var/lib/kubelet/config.yaml <<EOF
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
cgroupDriver: systemd
EOF
systemctl restart kubelet
```

**生成kubeadm初始化配置文件** [可选] 仅当需自定义初始化配置时用

```bash
kubeadm config print init-defaults > init.default.yaml
```

**测试环境是否正常**（WARNING是正常的）

```bash
kubeadm init phase preflight [--config kubeadm-init.yaml]
```

![img](https://img2020.cnblogs.com/blog/1149398/202007/1149398-20200703184500141-1054895820.png)

> 上图提示Warning是正常的，校验不了k8s信息是因为连不上被ban的网站，最后一个提示是因我本地未关闭防火墙，请我看清楚必要放行的端口号是否畅通

**初始化master** 10.244.0.0/16是flannel固定使用的IP段，设置取决于网络组件要求

```bash
kubeadm init --pod-network-cidr=10.244.0.0/16 --kubernetes-version=v1.18.5 [--config kubeadm-init.yaml]
```

输出如下：

```bash
[root@kube-master k8s]# kubeadm init --pod-network-cidr=10.244.0.0/16 --kubernetes-version=v1.18.5
W0703 18:49:19.076654   16469 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
[init] Using Kubernetes version: v1.18.5
[preflight] Running pre-flight checks
	[WARNING Firewalld]: firewalld is active, please ensure ports [6443 10250] are open or your cluster may not function correctly
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kube-master kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.87.145]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [kube-master localhost] and IPs [192.168.87.145 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [kube-master localhost] and IPs [192.168.87.145 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
W0703 18:49:23.039913   16469 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[control-plane] Creating static Pod manifest for "kube-scheduler"
W0703 18:49:23.040907   16469 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 21.505101 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.18" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node kube-master as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node kube-master as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: 2b7cfv.6bhz4z3a3vzyg498
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.87.145:6443 --token 2b7cfv.6bhz4z3a3vzyg498 \
    --discovery-token-ca-cert-hash sha256:79bd63d82634f9953cc9d6b5a923fa87c973f0c3fd9ed7270167052dd834c026
```

**为日常使用集群的用户添加kubectl使用权限**

```bash
su hellxz
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/admin.conf
sudo chown $(id -u):$(id -g) $HOME/.kube/admin.conf
echo "export KUBECONFIG=$HOME/.kube/admin.conf" >> ~/.bashrc
exit
```

![img](https://img2020.cnblogs.com/blog/1149398/202007/1149398-20200706154421749-109893170.png)

**配置master认证**

```bash
echo 'export KUBECONFIG=/etc/kubernetes/admin.conf' >> /etc/profile
. /etc/profile
```

> 如果不配置这个，会提示如下输出：
>
> ```bash
> The connection to the server localhost:8080 was refused - did you specify the right host or port?
> ```
>
> 此时master节点已经初始化成功，但是还未完装网络组件，还无法与其他节点通讯

**安装网络组件，以flannel为例**

```bash
cd ~/k8s
yum install -y wget
#下载flannel最新配置文件
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl apply -f kube-flannel.yml
```

![img](https://img2020.cnblogs.com/blog/1149398/202007/1149398-20200703203611610-489015457.png)

**查看kube-master节点状态**

```bash
kubectl get nodes
```

![img](https://img2020.cnblogs.com/blog/1149398/202007/1149398-20200703203656234-1367531429.png)

> 如果STATUS提示`NotReady`，可以通过 `kubectl describe node kube-master` 查看具体的描述信息，性能差的服务器到达Ready状态时间会长些

**备份镜像供其他节点使用**

在kube-master节点将镜像备份出来，便于后续传输给其他node节点，当然有镜像仓库更好

```bash
docker save k8s.gcr.io/kube-proxy:v1.18.5 \
            k8s.gcr.io/kube-apiserver:v1.18.5 \
            k8s.gcr.io/kube-controller-manager:v1.18.5 \
            k8s.gcr.io/kube-scheduler:v1.18.5 \
            k8s.gcr.io/pause:3.2 \
            k8s.gcr.io/coredns:1.6.7 \
            k8s.gcr.io/etcd:3.4.3-0 > k8s-imagesV1.18.5.tar
```

![img](https://img2020.cnblogs.com/blog/1149398/202007/1149398-20200703191337838-770260216.png)

### 初始化kube-node*节点并加入集群

**拷贝镜像到node节点**，以kube-node1举例，node2不再累述

```bash
#此时命令在kube-node*节点上执行
mkdir ~/k8s
scp root@kube-master:/root/k8s/k8s-imagesV1.18.5.tar ~/k8s
```

**获取加入kubernetes命令**，未忘可不选

刚才在初始化kube-master节点时，有在最后输出其加入集群的命令，假如我没记下来，那怎么办呢？

访问kube-master输入创建新token命令，同时输出加入集群的命令：

```bash
kubeadm token create --print-join-command
```

![img](https://img2020.cnblogs.com/blog/1149398/202007/1149398-20200703204353429-1001595838.png)

**在kube-node\*节点上执行加入集群命令**

```bash
kubeadm join 192.168.87.145:6443 --token jdyzyq.icwlpkm36kgs6nqh     --discovery-token-ca-cert-hash sha256:24f9b05fa10307ef6fff4132e0ec3c8b54917d4ff440b36108908aca588d8be7 
```

![img](https://img2020.cnblogs.com/blog/1149398/202007/1149398-20200703205930708-817131399.png)

### 查看集群节点状态

```bash
kubectl get nodes
```

![img](https://img2020.cnblogs.com/blog/1149398/202007/1149398-20200703210122591-836716446.png)

**参考**

> - 《Kubernetes权威指南》第4版
> - 官方文档 <https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/>