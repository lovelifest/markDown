### [Docker环境安装](http://www.macrozheng.com/#/deploy/mall_deploy_docker?id=docker环境安装)

### 环境查看

- 系统内核是3.0以上 

```shell
[root@rabbitmq001 ~]# uname -r
3.10.0-957.el7.x86_64
```

- 系统版本

```shell
[root@rabbitmq001 ~]# cat /etc/os-release
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"

```



### 安装yum-utils：

```shell
yum install -y yum-utils device-mapper-persistent-data lvm2 
```

### 为yum源添加docker仓库位置：

```shell
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo 
```

### **添加国内yum源**

```
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo 
```

### 更新yum软件包索引

```shell
yum makecache fast
```

### 安装docker：

```
yum install docker-ce 
```

### 启动docker：

```
systemctl start docker 
```

### 开机启动docker

```
sudo systemctl enable docker 
```



## 镜像加速

由于国内网络问题，需要配置加速器来加速。

 需要修改配置文件，Docker 使用 /etc/docker/daemon.json来配置daemon。

```shell
vi /etc/docker/daemon.json 
```

在配置文件中加入

```json
{
   "registry-mirrors": ["http://hub-mirror.c.163.com"] 
} 
```

```shell
systemctl restart docker 
```



### docker run hello-world

```shell
[root@rabbitmq001 ~]# docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete 
Digest: sha256:d58e752213a51785838f9eed2b7a498ffa1cb3aa7f946dda11af39286c3db9a9
Status: Downloaded newer image for hello-world:latest
WARNING: IPv4 forwarding is disabled. Networking will not work.

Hello from Docker!
```

### docker images

```shell
[root@rabbitmq001 ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              bf756fb1ae65        6 months ago        13.3kB
```

## 卸载

```shell
yum remove docker-ce
##删除目录
rm -rf /var/lib/docker
## /var/lib/docker docker默认工作路径
```

