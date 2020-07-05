### 数据卷

#### 	概念

​		数据如果在容器中，如果删除容器，数据就会丢失，就是数据持久化，删除容器的时候保留数据在宿主机上

#### 	安装Mysql

```shell
[root@rabbitmq001 ~]# docker pull mysql:5.7
5.7: Pulling from library/mysql
8559a31e96f4: Already exists 
d51ce1c2e575: Pull complete 
c2344adc4858: Pull complete 
fcf3ceff18fc: Pull complete 
16da0c38dc5b: Pull complete 
b905d1797e97: Pull complete 
4b50d1c6b05c: Pull complete 
d85174a87144: Pull complete 
a4ad33703fa8: Pull complete 
f7a5433ce20d: Pull complete 
3dcd2a278b4a: Pull complete 
Digest: sha256:32f9d9a069f7a735e28fd44ea944d53c61f990ba71460c5c183e610854ca4854
Status: Downloaded newer image for mysql:5.7
docker.io/library/mysql:5.7
[root@rabbitmq001 ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@rabbitmq001 ~]# docker images
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
tomcat02              1.0                 560f2d2b7746        19 minutes ago      652MB
centos                latest              831691599b88        2 weeks ago         215MB
redis                 latest              235592615444        3 weeks ago         104MB
tomcat                latest              2eb5a120304e        3 weeks ago         647MB
nginx                 latest              2622e6cca7eb        3 weeks ago         132MB
mysql                 5.7                 9cfcce23593a        3 weeks ago         448MB
portainer/portainer   latest              cd645f5a4769        4 weeks ago         79.1MB
elasticsearch         7.7.0               7ec4f35ab452        7 weeks ago         757MB
elasticsearch         latest              5acf0e8da90b        21 months ago       486MB
[root@rabbitmq001 ~]# docker run -d -p 3310:3306 -v /usr/local/mysql/conf
"docker run" requires at least 1 argument.
See 'docker run --help'.

Usage:  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

Run a command in a new container
[root@rabbitmq001 ~]# docker run -d -p 3310:3306 -v /usr/local/mysql/conf:/etc/mysql/conf.d -v /usr/local/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7
6d349deea5d95ecdea5c0d6ce7a2e2082c650e928564996f1b4a28242d4e1214
[root@rabbitmq001 ~]# 

```

#### 具名挂载和匿名挂载

```shell
# 匿名挂载 -v 容器路径
[root@rabbitmq001 data]# docker run -d -P --name nginx01 -v /etc/nginx nginx
137ac21f8910ebb625318058d9cbb7073a5b705596be0eeab7152235266e3107
[root@rabbitmq001 data]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
137ac21f8910        nginx               "/docker-entrypoint.…"   4 seconds ago       Up 4 seconds        0.0.0.0:32768->80/tcp               nginx01
6d349deea5d9        mysql:5.7           "docker-entrypoint.s…"   9 minutes ago       Up 9 minutes        33060/tcp, 0.0.0.0:3310->3306/tcp   mysql01
[root@rabbitmq001 data]# docker volume ls
DRIVER              VOLUME NAME
local               323f582eb387f8bb9f80701e408db9fdb999950d366cbb64974aed38225c896c
local               160612e9e9bab8fb17883b2be963d63f364b2aa1ace1591e11d99bc80a95af75
local               d4b0452c4d45a3eeab10dfe0f0e8b2d32105b7f8762be146ff782cab91b8b23c
local               ec0d1c7c8e6ea8635644ff86f380eccdaf343d3f32f0e5df7ed0e7801ff038f6
# 具名挂在 -v volume名字：容器路径
[root@rabbitmq001 data]# docker run -d -P --name nginx02 -v jumingnginx:/etc/nginx nginx
39aa1a3fae0d5007ae58cb867d339f2fe6a1c5810e62865fe32bbf4c2ea2ba1f
[root@rabbitmq001 data]# docker volume ls
DRIVER              VOLUME NAME
local               323f582eb387f8bb9f80701e408db9fdb999950d366cbb64974aed38225c896c
local               160612e9e9bab8fb17883b2be963d63f364b2aa1ace1591e11d99bc80a95af75
local               d4b0452c4d45a3eeab10dfe0f0e8b2d32105b7f8762be146ff782cab91b8b23c
local               ec0d1c7c8e6ea8635644ff86f380eccdaf343d3f32f0e5df7ed0e7801ff038f6
local               jumingnginx
[root@rabbitmq001 data]# docker volume inspect jumingnginx
[
    {
        "CreatedAt": "2020-07-05T14:24:18+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/jumingnginx/_data",
        "Name": "jumingnginx",
        "Options": null,
        "Scope": "local"
    }
]
[root@rabbitmq001 data]# 
```

#### 拓展

```shell
#ro 	表示read only 只读
#rw		表示read write 可读可写
docker run -d -P --name nginx02 -v jumingnginx:/etc/nginx:ro nginx
docker run -d -P --name nginx02 -v jumingnginx:/etc/nginx:rw nginx
```



### Dockerfile

#### 	概念

​		Dockerfile用来构建docker镜像的文件

#### 	步骤

- ###### 编写一个dockerfile文件

- ###### docker build 构成一个镜像

- ###### docker run 运行镜像

- ###### docker push发布镜像	（DockerHub、阿里云镜像仓库）				

#### Dockerfile文件样例

```
FROM centos

VOLUME ["volume01","volume02"]

CMD echo "------end-------"

CMD /bin/bash
```

```shell
[root@rabbitmq001 centos]# docker build -f Dockerfile -t st/centos:1.0 .
Sending build context to Docker daemon  2.048kB
Step 1/4 : FROM centos
 ---> 831691599b88
Step 2/4 : VOLUME ["volume01","volume02"]
 ---> Running in f73e9c04203f
Removing intermediate container f73e9c04203f
 ---> ff0478349234
Step 3/4 : CMD echo "------end-------"
 ---> Running in c614c2c68879
Removing intermediate container c614c2c68879
 ---> 909f0df068cc
Step 4/4 : CMD /bin/bash
 ---> Running in feaac6634cc6
Removing intermediate container feaac6634cc6
 ---> 0f69e5530335
Successfully built 0f69e5530335
Successfully tagged st/centos:1.0
[root@rabbitmq001 centos]# docker images
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
st/centos             1.0                 0f69e5530335        8 seconds ago       215MB
tomcat02              1.0                 560f2d2b7746        About an hour ago   652MB
centos                latest              831691599b88        2 weeks ago         215MB
redis                 latest              235592615444        3 weeks ago         104MB
tomcat                latest              2eb5a120304e        3 weeks ago         647MB
nginx                 latest              2622e6cca7eb        3 weeks ago         132MB
mysql                 5.7                 9cfcce23593a        3 weeks ago         448MB
portainer/portainer   latest              cd645f5a4769        4 weeks ago         79.1MB
elasticsearch         7.7.0               7ec4f35ab452        7 weeks ago         757MB
elasticsearch         latest              5acf0e8da90b        21 months ago       486MB
[root@rabbitmq001 centos]# docker run -it -d st/centos:1.0
4df4a0266d31df6324cc7303673b78734031fb15b4b25b1eb200c22068066d18
[root@rabbitmq001 centos]# docker exec -it 4df4a0266 /bin/bash
#这里能看到挂在的两个数据卷
[root@4df4a0266d31 /]# ls 
bin  etc   lib	  lost+found  mnt  proc  run   srv  tmp  var	   volume02
dev  home  lib64  media       opt  root  sbin  sys  usr  volume01
[root@4df4a0266d31 /]# exit
exit
[root@rabbitmq001 centos]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
4df4a0266d31        st/centos:1.0       "/bin/sh -c /bin/bash"   35 seconds ago      Up 34 seconds                                           crazy_nobel
39aa1a3fae0d        nginx               "/docker-entrypoint.…"   29 minutes ago      Up 29 minutes       0.0.0.0:32769->80/tcp               nginx02
137ac21f8910        nginx               "/docker-entrypoint.…"   33 minutes ago      Up 33 minutes       0.0.0.0:32768->80/tcp               nginx01
6d349deea5d9        mysql:5.7           "docker-entrypoint.s…"   42 minutes ago      Up 42 minutes       33060/tcp, 0.0.0.0:3310->3306/tcp   mysql01
[root@rabbitmq001 centos]# docker inspect 4df4a0266d31
[
    {
        "Id": "4df4a0266d31df6324cc7303673b78734031fb15b4b25b1eb200c22068066d18",
        "Created": "2020-07-05T06:52:46.857068721Z",
        "Path": "/bin/sh",
        "Args": [
            "-c",
            "/bin/bash"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 106809,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2020-07-05T06:52:47.135379394Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:0f69e5530335e78d158ce066fdc1b77f6182cacbf0b5e790e652bb7adb32bbb5",
        "ResolvConfPath": "/var/lib/docker/containers/4df4a0266d31df6324cc7303673b78734031fb15b4b25b1eb200c22068066d18/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/4df4a0266d31df6324cc7303673b78734031fb15b4b25b1eb200c22068066d18/hostname",
        "HostsPath": "/var/lib/docker/containers/4df4a0266d31df6324cc7303673b78734031fb15b4b25b1eb200c22068066d18/hosts",
        "LogPath": "/var/lib/docker/containers/4df4a0266d31df6324cc7303673b78734031fb15b4b25b1eb200c22068066d18/4df4a0266d31df6324cc7303673b78734031fb15b4b25b1eb200c22068066d18-json.log",
        "Name": "/crazy_nobel",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "Capabilities": null,
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/9360602c0eddd4f7e132456283ae78bf0cb2d805b587d87fe868143be2ef5e73-init/diff:/var/lib/docker/overlay2/af13b024aeb86a6af03c115bc5c4777017ef4bb33cd84945504f02a0d5463d52/diff",
                "MergedDir": "/var/lib/docker/overlay2/9360602c0eddd4f7e132456283ae78bf0cb2d805b587d87fe868143be2ef5e73/merged",
                "UpperDir": "/var/lib/docker/overlay2/9360602c0eddd4f7e132456283ae78bf0cb2d805b587d87fe868143be2ef5e73/diff",
                "WorkDir": "/var/lib/docker/overlay2/9360602c0eddd4f7e132456283ae78bf0cb2d805b587d87fe868143be2ef5e73/work"
            },
            "Name": "overlay2"
        },
        # 两个匿名挂在的位置
        "Mounts": [
            {
                "Type": "volume",
                "Name": "e61e5a01ceb0d535310fe0354361b2aea851e0f160c3107d9fc2011f07ed60b7",
                "Source": "/var/lib/docker/volumes/e61e5a01ceb0d535310fe0354361b2aea851e0f160c3107d9fc2011f07ed60b7/_data",
                "Destination": "volume02",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            },
            {
                "Type": "volume",
                "Name": "b0f80adf6620c12969ff114c0e8c61607e81afc294860ab1e1f8df8438bb930a",
                "Source": "/var/lib/docker/volumes/b0f80adf6620c12969ff114c0e8c61607e81afc294860ab1e1f8df8438bb930a/_data",
                "Destination": "volume01",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
        "Config": {
            "Hostname": "4df4a0266d31",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": true,
            "OpenStdin": true,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "/bin/bash"
            ],
            "Image": "st/centos:1.0",
            "Volumes": {
                "volume01": {},
                "volume02": {}
            },
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "org.label-schema.build-date": "20200611",
                "org.label-schema.license": "GPLv2",
                "org.label-schema.name": "CentOS Base Image",
                "org.label-schema.schema-version": "1.0",
                "org.label-schema.vendor": "CentOS"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "4a2f624d9d88c2f5c96fbec5ce702e6694042c641715839e9bc97af12eb342cf",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/4a2f624d9d88",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "a37ae0498057a52956c6f6e31225fbf9dd9b6373700651357ded39b22acecaf4",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.5",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:05",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "55d03b18d7fcdf89234231dc6b0566e458a4edb012d031b935d07df895396d14",
                    "EndpointID": "a37ae0498057a52956c6f6e31225fbf9dd9b6373700651357ded39b22acecaf4",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.5",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:05",
                    "DriverOpts": null
                }
            }
        }
    }
]
[root@rabbitmq001 centos]# 

```

#### --volumes-from命令

#### 练习：多个centos公用数据卷

```shell
[root@rabbitmq001 centos]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
4df4a0266d31        st/centos:1.0       "/bin/sh -c /bin/bash"   16 minutes ago      Up 16 minutes                                           crazy_nobel
39aa1a3fae0d        nginx               "/docker-entrypoint.…"   44 minutes ago      Up 44 minutes       0.0.0.0:32769->80/tcp               nginx02
137ac21f8910        nginx               "/docker-entrypoint.…"   49 minutes ago      Up 49 minutes       0.0.0.0:32768->80/tcp               nginx01
6d349deea5d9        mysql:5.7           "docker-entrypoint.s…"   58 minutes ago      Up 58 minutes       33060/tcp, 0.0.0.0:3310->3306/tcp   mysql01
[root@rabbitmq001 centos]# docker run -it -d --name centos01  st/centos:1.0
a296f36065438283e5f1ad84f1737f7d5ba6a15fffae07baf86a5aadb109135c
#centos02公用centos01的数据卷
[root@rabbitmq001 centos]# docker run -it -d --name centos02 --volumes-from centos01  st/centos:1.0
9923f23db556e66f68c1d15f22573bfe3c43e8aa86dba163f91f67217127add7
[root@rabbitmq001 centos]# docker run -it -d --name centos03 --volumes-from centos01  st/centos:1.0
ad6ae952f37e168edc2654dddba5cda845a9d8836ec94212353dfc9b817f6388
[root@rabbitmq001 centos]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
ad6ae952f37e        st/centos:1.0       "/bin/sh -c /bin/bash"   7 seconds ago       Up 7 seconds                                            centos03
9923f23db556        st/centos:1.0       "/bin/sh -c /bin/bash"   14 seconds ago      Up 13 seconds                                           centos02
a296f3606543        st/centos:1.0       "/bin/sh -c /bin/bash"   2 minutes ago       Up 2 minutes                                            centos01
4df4a0266d31        st/centos:1.0       "/bin/sh -c /bin/bash"   20 minutes ago      Up 20 minutes                                           crazy_nobel
39aa1a3fae0d        nginx               "/docker-entrypoint.…"   48 minutes ago      Up 48 minutes       0.0.0.0:32769->80/tcp               nginx02
137ac21f8910        nginx               "/docker-entrypoint.…"   52 minutes ago      Up 52 minutes       0.0.0.0:32768->80/tcp               nginx01
6d349deea5d9        mysql:5.7           "docker-entrypoint.s…"   About an hour ago   Up About an hour    33060/tcp, 0.0.0.0:3310->3306/tcp   mysql01
[root@rabbitmq001 centos]# docker exec -it a296f3606543 /bin/bash
[root@a296f3606543 /]# ls
bin  etc   lib	  lost+found  mnt  proc  run   srv  tmp  var	   volume02
dev  home  lib64  media       opt  root  sbin  sys  usr  volume01
[root@a296f3606543 /]# cd volume01
[root@a296f3606543 volume01]# touch a.txt
[root@a296f3606543 volume01]# ls
a.txt
[root@a296f3606543 volume01]# exit
exit
[root@rabbitmq001 centos]# docker exec -it 9923f23db556 /bin/bash
[root@9923f23db556 /]# cd /volume01/
[root@9923f23db556 volume01]# ls
a.txt

```



#### 练习：多个mysql同步数据

```shell
[root@rabbitmq001 ~]# docker run -d -p 3310:3306 -v /usr/local/mysql/conf:/etc/mysql/conf.d -v /usr/local/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7

# 其他的mysql同步mysql01数据就可以了
[root@rabbitmq001 ~]# docker run -d -p 3310:3306  -e MYSQL_ROOT_PASSWORD=123456 --name mysql02 --volumes-from mysql01 mysql:5.7
```

#### dockerfile构建过程

1. 每个保留关键字（指令）都必须是大写字母

2. 指令是顺序执行

3. #表示注释

4. 每一个指令都回创建一个镜像层，并提交

#### Dockerfile的指令

```shell
FROM 			# 基础镜像
MAINTAINER		# 镜像是谁写的，姓名+邮箱
RUN				# 镜像构建时候需要运行的命令
ADD				# 增加一些本地的压缩包，像tomcat压缩包等等
WORKDIR			# 镜像的工作目录
VOLUME			# 数据卷
EXPOSE			# 暴露端口
CMD				# 指定容器启动时候要运行的命令，只有最后一个会生效，可被替代
ENTRYPOINT		# 指定容器启动时候要运行的命令，可以追加命令
ONBUILD			# 当构建一个被继承Dockerfile 这个时候就会运行ONBUILD 的指令(触发指令)
COPY			# 类似ADD命令，将文件拷贝到镜像中
ENV				# 构建的时候设置环境变量
```

#### 练习：构建一个带vim和ifconfig的centos镜像

```shell
[root@rabbitmq001 centos]# touch Dockerfile
[root@rabbitmq001 centos]# vim Dockerfile
[root@rabbitmq001 centos]# cat Dockerfile 
FROM centos

MAINTAINER songtao<314436024@qq.com>

ENV MYPATH /usr/local

WORKDIR $MYPATH

RUN yum -y install vim

RUN yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH

CMD echo "-----end--------"

CMD /bin/bash

[root@rabbitmq001 centos]# docker build -f Dockerfile -t mycentos:0.1 .
Sending build context to Docker daemon  2.048kB
Step 1/10 : FROM centos
 ---> 831691599b88
Step 2/10 : MAINTAINER songtao<314436024@qq.com>
 ---> Running in 886e460555e5
Removing intermediate container 886e460555e5
 ---> 41d7f600e908
Step 3/10 : ENV MYPATH /usr/local
 ---> Running in 3c0d7f665430
Removing intermediate container 3c0d7f665430
 ---> 813e274c97a0
Step 4/10 : WORKDIR $MYPATH
 ---> Running in 7cd9ef234b31
Removing intermediate container 7cd9ef234b31
 ---> 4787bc20c855
Step 5/10 : RUN yum -y install vim
 ---> Running in aee5ecdaa34a
CentOS-8 - AppStream                            3.6 MB/s | 5.8 MB     00:01    
CentOS-8 - Base                                 189 kB/s | 2.2 MB     00:12    
CentOS-8 - Extras                               977  B/s | 6.7 kB     00:07    
Dependencies resolved.
================================================================================
 Package             Arch        Version                   Repository      Size
================================================================================
Installing:
 vim-enhanced        x86_64      2:8.0.1763-13.el8         AppStream      1.4 M
Installing dependencies:
 gpm-libs            x86_64      1.20.7-15.el8             AppStream       39 k
 vim-common          x86_64      2:8.0.1763-13.el8         AppStream      6.3 M
 vim-filesystem      noarch      2:8.0.1763-13.el8         AppStream       48 k
 which               x86_64      2.21-12.el8               BaseOS          49 k

Transaction Summary
================================================================================
Install  5 Packages

Total download size: 7.8 M
Installed size: 31 M
Downloading Packages:
(1/5): gpm-libs-1.20.7-15.el8.x86_64.rpm        350 kB/s |  39 kB     00:00    
(2/5): vim-filesystem-8.0.1763-13.el8.noarch.rp 934 kB/s |  48 kB     00:00    
(3/5): which-2.21-12.el8.x86_64.rpm             143 kB/s |  49 kB     00:00    
(4/5): vim-enhanced-8.0.1763-13.el8.x86_64.rpm  2.5 MB/s | 1.4 MB     00:00    
(5/5): vim-common-8.0.1763-13.el8.x86_64.rpm    6.5 MB/s | 6.3 MB     00:00    
--------------------------------------------------------------------------------
Total                                           512 kB/s | 7.8 MB     00:15     
warning: /var/cache/dnf/AppStream-02e86d1c976ab532/packages/gpm-libs-1.20.7-15.el8.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID 8483c65d: NOKEY
CentOS-8 - AppStream                            200 kB/s | 1.6 kB     00:00    
Importing GPG key 0x8483C65D:
 Userid     : "CentOS (CentOS Official Signing Key) <security@centos.org>"
 Fingerprint: 99DB 70FA E1D7 CE22 7FB6 4882 05B5 55B3 8483 C65D
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
Key imported successfully
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1 
  Installing       : which-2.21-12.el8.x86_64                               1/5 
  Installing       : vim-filesystem-2:8.0.1763-13.el8.noarch                2/5 
  Installing       : vim-common-2:8.0.1763-13.el8.x86_64                    3/5 
  Installing       : gpm-libs-1.20.7-15.el8.x86_64                          4/5 
  Running scriptlet: gpm-libs-1.20.7-15.el8.x86_64                          4/5 
  Installing       : vim-enhanced-2:8.0.1763-13.el8.x86_64                  5/5 
  Running scriptlet: vim-enhanced-2:8.0.1763-13.el8.x86_64                  5/5 
  Running scriptlet: vim-common-2:8.0.1763-13.el8.x86_64                    5/5 
  Verifying        : gpm-libs-1.20.7-15.el8.x86_64                          1/5 
  Verifying        : vim-common-2:8.0.1763-13.el8.x86_64                    2/5 
  Verifying        : vim-enhanced-2:8.0.1763-13.el8.x86_64                  3/5 
  Verifying        : vim-filesystem-2:8.0.1763-13.el8.noarch                4/5 
  Verifying        : which-2.21-12.el8.x86_64                               5/5 

Installed:
  gpm-libs-1.20.7-15.el8.x86_64         vim-common-2:8.0.1763-13.el8.x86_64    
  vim-enhanced-2:8.0.1763-13.el8.x86_64 vim-filesystem-2:8.0.1763-13.el8.noarch
  which-2.21-12.el8.x86_64             

Complete!
Removing intermediate container aee5ecdaa34a
 ---> 3d238ba1c2ea
Step 6/10 : RUN yum -y install net-tools
 ---> Running in 32d4991c9d43
Last metadata expiration check: 0:00:21 ago on Sun Jul  5 07:59:14 2020.
Dependencies resolved.
================================================================================
 Package         Architecture Version                        Repository    Size
================================================================================
Installing:
 net-tools       x86_64       2.0-0.51.20160912git.el8       BaseOS       323 k

Transaction Summary
================================================================================
Install  1 Package

Total download size: 323 k
Installed size: 1.0 M
Downloading Packages:
net-tools-2.0-0.51.20160912git.el8.x86_64.rpm   603 kB/s | 323 kB     00:00    
--------------------------------------------------------------------------------
Total                                            66 kB/s | 323 kB     00:04     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1 
  Installing       : net-tools-2.0-0.51.20160912git.el8.x86_64              1/1 
  Running scriptlet: net-tools-2.0-0.51.20160912git.el8.x86_64              1/1 
  Verifying        : net-tools-2.0-0.51.20160912git.el8.x86_64              1/1 

Installed:
  net-tools-2.0-0.51.20160912git.el8.x86_64                                     

Complete!
Removing intermediate container 32d4991c9d43
 ---> 175d8d3ac983
Step 7/10 : EXPOSE 80
 ---> Running in 4a42c810c4a4
Removing intermediate container 4a42c810c4a4
 ---> 5ea9a6b3cf78
Step 8/10 : CMD echo $MYPATH
 ---> Running in 5e65922721bf
Removing intermediate container 5e65922721bf
 ---> fa0f8eec41dd
Step 9/10 : CMD echo "-----end--------"
 ---> Running in 0a12623e175d
Removing intermediate container 0a12623e175d
 ---> 9751e3111bf6
Step 10/10 : CMD /bin/bash
 ---> Running in 8e0f78437948
Removing intermediate container 8e0f78437948
 ---> 499fa11010cd
Successfully built 499fa11010cd
Successfully tagged mycentos:0.1
[root@rabbitmq001 centos]# docker images
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
mycentos              0.1                 499fa11010cd        32 seconds ago      295MB
st/centos             1.0                 0f69e5530335        About an hour ago   215MB
tomcat02              1.0                 560f2d2b7746        2 hours ago         652MB
centos                latest              831691599b88        2 weeks ago         215MB
redis                 latest              235592615444        3 weeks ago         104MB
tomcat                latest              2eb5a120304e        3 weeks ago         647MB
nginx                 latest              2622e6cca7eb        3 weeks ago         132MB
mysql                 5.7                 9cfcce23593a        3 weeks ago         448MB
portainer/portainer   latest              cd645f5a4769        4 weeks ago         79.1MB
elasticsearch         7.7.0               7ec4f35ab452        7 weeks ago         757MB
elasticsearch         latest              5acf0e8da90b        21 months ago       486MB
[root@rabbitmq001 centos]# docker run -it 499fa11010cd
[root@78d0fa8cd296 local]# pwd
/usr/local
[root@5ba222f0adfe local]# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.3  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:03  txqueuelen 0  (Ethernet)
        RX packets 7  bytes 586 (586.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0


```

#### docker history

```shell
# 查看镜像的构建历史
[root@rabbitmq001 centos]# docker history tomcat
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
2eb5a120304e        3 weeks ago         /bin/sh -c #(nop)  CMD ["catalina.sh" "run"]    0B                  
<missing>           3 weeks ago         /bin/sh -c #(nop)  EXPOSE 8080                  0B                  
<missing>           3 weeks ago         /bin/sh -c set -e  && nativeLines="$(catalin…   0B                  
<missing>           3 weeks ago         /bin/sh -c set -eux;   savedAptMark="$(apt-m…   19.9MB              
<missing>           3 weeks ago         /bin/sh -c #(nop)  ENV TOMCAT_SHA512=75e16a0…   0B                  
<missing>           3 weeks ago         /bin/sh -c #(nop)  ENV TOMCAT_VERSION=9.0.36    0B                  
<missing>           3 weeks ago         /bin/sh -c #(nop)  ENV TOMCAT_MAJOR=9           0B                  
<missing>           3 weeks ago         /bin/sh -c #(nop)  ENV GPG_KEYS=05AB33110949…   0B                  
<missing>           3 weeks ago         /bin/sh -c #(nop)  ENV LD_LIBRARY_PATH=/usr/…   0B                  
<missing>           3 weeks ago         /bin/sh -c #(nop)  ENV TOMCAT_NATIVE_LIBDIR=…   0B                  
<missing>           3 weeks ago         /bin/sh -c #(nop) WORKDIR /usr/local/tomcat     0B                  
<missing>           3 weeks ago         /bin/sh -c mkdir -p "$CATALINA_HOME"            0B                  
<missing>           3 weeks ago         /bin/sh -c #(nop)  ENV PATH=/usr/local/tomca…   0B                  
<missing>           3 weeks ago         /bin/sh -c #(nop)  ENV CATALINA_HOME=/usr/lo…   0B                  
<missing>           3 weeks ago         /bin/sh -c #(nop)  CMD ["jshell"]               0B                  
<missing>           3 weeks ago         /bin/sh -c set -eux;   dpkgArch="$(dpkg --pr…   323MB               
<missing>           3 weeks ago         /bin/sh -c #(nop)  ENV JAVA_URL_VERSION=11.0…   0B                  
<missing>           3 weeks ago         /bin/sh -c #(nop)  ENV JAVA_BASE_URL=https:/…   0B                  
<missing>           3 weeks ago         /bin/sh -c #(nop)  ENV JAVA_VERSION=11.0.7      0B                  
<missing>           3 weeks ago         /bin/sh -c { echo '#/bin/sh'; echo 'echo "$J…   27B                 
<missing>           3 weeks ago         /bin/sh -c #(nop)  ENV PATH=/usr/local/openj…   0B                  
<missing>           3 weeks ago         /bin/sh -c #(nop)  ENV JAVA_HOME=/usr/local/…   0B                  
<missing>           3 weeks ago         /bin/sh -c #(nop)  ENV LANG=C.UTF-8             0B                  
<missing>           3 weeks ago         /bin/sh -c set -eux;  apt-get update;  apt-g…   11.1MB              
<missing>           3 weeks ago         /bin/sh -c apt-get update && apt-get install…   146MB               
<missing>           3 weeks ago         /bin/sh -c set -ex;  if ! command -v gpg > /…   17.5MB              
<missing>           3 weeks ago         /bin/sh -c apt-get update && apt-get install…   16.5MB              
<missing>           3 weeks ago         /bin/sh -c #(nop)  CMD ["bash"]                 0B                  
<missing>           3 weeks ago         /bin/sh -c #(nop) ADD file:1ab357efe422cfed5…   114MB               
[root@rabbitmq001 centos]# 

```

#### CMD和ENTRYPOINT的区别

```shell
[root@rabbitmq001 centos]# touch Dockerfile-cmd-test
[root@rabbitmq001 centos]# touch Dockerfile-entrypoint-test
[root@rabbitmq001 centos]# vim Dockerfile-cmd-test 
[root@rabbitmq001 centos]# vim Dockerfile-entrypoint-test 
[root@rabbitmq001 centos]# cat Dockerfile-cmd-test 
FROM centos

CMD ["ls","-a"]
[root@rabbitmq001 centos]# cat Dockerfile-entrypoint-test 
FROM centos
ENTRYPOINT ["ls","-a"]
[root@rabbitmq001 centos]# 

[root@rabbitmq001 centos]# docker build -f Dockerfile-cmd-test -t centos-cmd-test .
Sending build context to Docker daemon  4.096kB
Step 1/2 : FROM centos
 ---> 831691599b88
Step 2/2 : CMD ["ls","-a"]
 ---> Running in 820b7d442361
Removing intermediate container 820b7d442361
 ---> e5df2f531199
Successfully built e5df2f531199
Successfully tagged centos-cmd-test:latest
[root@rabbitmq001 centos]# docker build -f Dockerfile-entrypoint-test -t centos-entrypoint-test .
Sending build context to Docker daemon  4.096kB
Step 1/2 : FROM centos
 ---> 831691599b88
Step 2/2 : ENTRYPOINT ["ls","-a"]
 ---> Running in d976ee2d2e4f
Removing intermediate container d976ee2d2e4f
 ---> d5ccc2ef5e4d
Successfully built d5ccc2ef5e4d
Successfully tagged centos-entrypoint-test:latest
# cmd 在启动容器时候会替换ls -a 为 -l命令，单独执行-l报错
[root@rabbitmq001 centos]# docker run -it e5df2f531199 -l
docker: Error response from daemon: OCI runtime create failed: container_linux.go:349: starting container process caused "exec: \"-l\": executable file not found in $PATH": unknown.
#entrypoint 在执行命令为拼接 即为ls -al
[root@rabbitmq001 centos]# docker run -it d5ccc2ef5e4d -l
total 0
drwxr-xr-x.   1 root root   6 Jul  5 08:16 .
drwxr-xr-x.   1 root root   6 Jul  5 08:16 ..
-rwxr-xr-x.   1 root root   0 Jul  5 08:16 .dockerenv
lrwxrwxrwx.   1 root root   7 May 11  2019 bin -> usr/bin
drwxr-xr-x.   5 root root 360 Jul  5 08:16 dev
drwxr-xr-x.   1 root root  66 Jul  5 08:16 etc
drwxr-xr-x.   2 root root   6 May 11  2019 home
lrwxrwxrwx.   1 root root   7 May 11  2019 lib -> usr/lib
lrwxrwxrwx.   1 root root   9 May 11  2019 lib64 -> usr/lib64
drwx------.   2 root root   6 Jun 11 02:35 lost+found
drwxr-xr-x.   2 root root   6 May 11  2019 media
drwxr-xr-x.   2 root root   6 May 11  2019 mnt
drwxr-xr-x.   2 root root   6 May 11  2019 opt
dr-xr-xr-x. 125 root root   0 Jul  5 08:16 proc
dr-xr-x---.   2 root root 162 Jun 11 02:35 root
drwxr-xr-x.  11 root root 163 Jun 11 02:35 run
lrwxrwxrwx.   1 root root   8 May 11  2019 sbin -> usr/sbin
drwxr-xr-x.   2 root root   6 May 11  2019 srv
dr-xr-xr-x.  13 root root   0 Jun 30 14:37 sys
drwxrwxrwt.   7 root root 145 Jun 11 02:35 tmp
drwxr-xr-x.  12 root root 144 Jun 11 02:35 usr
drwxr-xr-x.  20 root root 262 Jun 11 02:35 var
[root@rabbitmq001 centos]# 

```

#### 练习：部署tomcat

```shell
[root@rabbitmq001 tomcat]# cat Dockerfile 
FROM centos

MAINTAINER songtao<314436024@qq.com>

ADD jdk-8u192-linux-x64.tar.gz /usr/local/
ADD apache-tomcat-8.5.56.tar.gz /usr/local/

RUN yum -y install vim

ENV MYPATH /usr/local
WORKDIR $MYPATH

ENV JAVA_HOME /usr/local/jdk1.8.0_192
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-8.5.56
ENV CATALINA_BASH /usr/local/apache-tomcat-8.5.56
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin

EXPOSE 8080

CMD /usr/local/apache-tomcat-8.5.56/bin/startup.sh && tail -f /usr/local/apache-tomcat-8.5.56/logs/catalina.out
[root@rabbitmq001 tomcat]# docker build  -f Dockerfile -t songtaotomcat .
Sending build context to Docker daemon  217.3MB
Step 1/14 : FROM centos
 ---> 831691599b88
Step 2/14 : MAINTAINER songtao<314436024@qq.com>
 ---> Using cache
 ---> 41d7f600e908
Step 3/14 : ADD jdk-8u192-linux-x64.tar.gz /usr/local/
 ---> e1878ff773bb
Step 4/14 : ADD apache-tomcat-8.5.56.tar.gz /usr/local/
 ---> 050c1f3928a2
Step 5/14 : RUN yum -y install vim
 ---> Running in 5d3a84b1a836
CentOS-8 - AppStream                            2.7 MB/s | 5.8 MB     00:02    
CentOS-8 - Base                                  71 kB/s | 2.2 MB     00:32    
CentOS-8 - Extras                               8.8 kB/s | 6.7 kB     00:00    
Dependencies resolved.
================================================================================
 Package             Arch        Version                   Repository      Size
================================================================================
Installing:
 vim-enhanced        x86_64      2:8.0.1763-13.el8         AppStream      1.4 M
Installing dependencies:
 gpm-libs            x86_64      1.20.7-15.el8             AppStream       39 k
 vim-common          x86_64      2:8.0.1763-13.el8         AppStream      6.3 M
 vim-filesystem      noarch      2:8.0.1763-13.el8         AppStream       48 k
 which               x86_64      2.21-12.el8               BaseOS          49 k

Transaction Summary
================================================================================
Install  5 Packages

Total download size: 7.8 M
Installed size: 31 M
Downloading Packages:
(1/5): gpm-libs-1.20.7-15.el8.x86_64.rpm        381 kB/s |  39 kB     00:00    
(2/5): vim-filesystem-8.0.1763-13.el8.noarch.rp 1.5 MB/s |  48 kB     00:00    
(3/5): which-2.21-12.el8.x86_64.rpm             426 kB/s |  49 kB     00:00    
(4/5): vim-enhanced-8.0.1763-13.el8.x86_64.rpm  3.5 MB/s | 1.4 MB     00:00    
(5/5): vim-common-8.0.1763-13.el8.x86_64.rpm    7.4 MB/s | 6.3 MB     00:00    
--------------------------------------------------------------------------------
Total                                           2.5 MB/s | 7.8 MB     00:03     
CentOS-8 - AppStream                            1.6 MB/s | 1.6 kB     00:00    
warning: /var/cache/dnf/AppStream-02e86d1c976ab532/packages/gpm-libs-1.20.7-15.el8.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID 8483c65d: NOKEY
Importing GPG key 0x8483C65D:
 Userid     : "CentOS (CentOS Official Signing Key) <security@centos.org>"
 Fingerprint: 99DB 70FA E1D7 CE22 7FB6 4882 05B5 55B3 8483 C65D
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
Key imported successfully
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1 
  Installing       : which-2.21-12.el8.x86_64                               1/5 
  Installing       : vim-filesystem-2:8.0.1763-13.el8.noarch                2/5 
  Installing       : vim-common-2:8.0.1763-13.el8.x86_64                    3/5 
  Installing       : gpm-libs-1.20.7-15.el8.x86_64                          4/5 
  Running scriptlet: gpm-libs-1.20.7-15.el8.x86_64                          4/5 
  Installing       : vim-enhanced-2:8.0.1763-13.el8.x86_64                  5/5 
  Running scriptlet: vim-enhanced-2:8.0.1763-13.el8.x86_64                  5/5 
  Running scriptlet: vim-common-2:8.0.1763-13.el8.x86_64                    5/5 
  Verifying        : gpm-libs-1.20.7-15.el8.x86_64                          1/5 
  Verifying        : vim-common-2:8.0.1763-13.el8.x86_64                    2/5 
  Verifying        : vim-enhanced-2:8.0.1763-13.el8.x86_64                  3/5 
  Verifying        : vim-filesystem-2:8.0.1763-13.el8.noarch                4/5 
  Verifying        : which-2.21-12.el8.x86_64                               5/5 

Installed:
  gpm-libs-1.20.7-15.el8.x86_64         vim-common-2:8.0.1763-13.el8.x86_64    
  vim-enhanced-2:8.0.1763-13.el8.x86_64 vim-filesystem-2:8.0.1763-13.el8.noarch
  which-2.21-12.el8.x86_64             

Complete!
Removing intermediate container 5d3a84b1a836
 ---> 6a9802ec9670
Step 6/14 : ENV MYPATH /usr/local
 ---> Running in cb0b504d117b
Removing intermediate container cb0b504d117b
 ---> f90526bf9c36
Step 7/14 : WORKDIR $MYPATH
 ---> Running in 45b7155f8f4e
Removing intermediate container 45b7155f8f4e
 ---> 1a9c72a39df7
Step 8/14 : ENV JAVA_HOME /usr/local/jdk1.8.0_192
 ---> Running in bcdbf0db2351
Removing intermediate container bcdbf0db2351
 ---> 0a831d62d2b9
Step 9/14 : ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
 ---> Running in 283fa99b5006
Removing intermediate container 283fa99b5006
 ---> 65100a8e2211
Step 10/14 : ENV CATALINA_HOME /usr/local/apache-tomcat-8.5.56
 ---> Running in 54787bb1d290
Removing intermediate container 54787bb1d290
 ---> 03b7cb92eb07
Step 11/14 : ENV CATALINA_BASH /usr/local/apache-tomcat-8.5.56
 ---> Running in 7fe15cefe2ef
Removing intermediate container 7fe15cefe2ef
 ---> fd498cf34160
Step 12/14 : ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin
 ---> Running in 79d43112dd88
Removing intermediate container 79d43112dd88
 ---> a1cbb5878095
Step 13/14 : EXPOSE 8080
 ---> Running in 522dc2b7ca7a
Removing intermediate container 522dc2b7ca7a
 ---> 0531e9a1d4e7
Step 14/14 : CMD /usr/local/apache-tomcat-8.5.56/bin/startup.sh && tail -f /usr/local/apache-tomcat-8.5.56/logs/catalina.out
 ---> Running in b3ceeea81d40
Removing intermediate container b3ceeea81d40
 ---> 2fd852d8de55
Successfully built 2fd852d8de55
Successfully tagged songtaotomcat:latest
[root@rabbitmq001 tomcat]# docker run -d -p 9090:8080 --name songtaotomcat001 -v /usr/local/tomcat/test:/usr/local/apache-tomcat-8.5.56/webapps/test -v /usr/local/tomcat/tomcatlog/:/usr/local/apache-tomcat-8.5.56/logs 2fd852d8de55
189388d752d7a864f0592f8cf79befa6857012e147bab7ad68c6ab76c3dd1b37
[root@rabbitmq001 tomcat]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
189388d752d7        2fd852d8de55        "/bin/sh -c '/usr/lo…"   5 seconds ago       Up 4 seconds        0.0.0.0:9090->8080/tcp              songtaotomcat001
4df4a0266d31        st/centos:1.0       "/bin/sh -c /bin/bash"   2 hours ago         Up 2 hours                                              crazy_nobel
6d349deea5d9        mysql:5.7           "docker-entrypoint.s…"   3 hours ago         Up 3 hours          33060/tcp, 0.0.0.0:3310->3306/tcp   mysql01


```

#### 发布自己的镜像到DockerHub上

```
docker login -u 
```

```
docker push 镜像名
```

发布到阿里云镜像服务器

1. 登陆阿里云
2. 产品→镜像和容器服务
3. 创建namespace
4. 提交到阿里云

### docker网络

#### 	理解docker0

1. 我们每启动一个docker容器，docker就会给docker容器分配一个ip，我们只要安装了docker,就会有一个网卡docker0,
2. 桥接模式，使用的计数是veth-pair技术
3. 每个容器的网卡都是一对对的
4. veth-pair 就是一对虚拟设备接口，成对出现的，一端连着协议，一端彼此相连，正因为有这个特性，veth-pair就充当这桥梁的作用，链接各种虚拟网络设备

```shell
[root@rabbitmq001 test]# ip addr
# 本机回环地址
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
# 虚拟机网卡地址
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:50:56:3a:cd:d4 brd ff:ff:ff:ff:ff:ff
    inet 192.168.95.10/24 brd 192.168.95.255 scope global noprefixroute dynamic ens33
       valid_lft 1527sec preferred_lft 1527sec
    inet6 fe80::2249:6aed:1006:87b6/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
# docker 网卡地址
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:e2:c2:af:ca brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:e2ff:fec2:afca/64 scope link 
       valid_lft forever preferred_lft forever
[root@rabbitmq001 test]# 

```

```shell
# 新启动一个容器以后
[root@rabbitmq001 test]# docker run -it -P --name tomcat01 tomcat
Unable to find image 'tomcat:latest' locally
latest: Pulling from library/tomcat
e9afc4f90ab0: Pull complete 
989e6b19a265: Pull complete 
af14b6c2f878: Pull complete 
5573c4b30949: Pull complete 
fb1a405f128d: Pull complete 
612a9f566fdc: Pull complete 
cf63ebed1142: Pull complete 
fbb20561cd50: Pull complete 
e99c920870d7: Pull complete 
b7f793f2be47: Pull complete 
Digest: sha256:81c2a95e5b1b5867229d75255abe54928d505deb81c8ff8949b61fde1a5d30a1
Status: Downloaded newer image for tomcat:latest
Using CATALINA_BASE:   /usr/local/tomcat
Using CATALINA_HOME:   /usr/local/tomcat
Using CATALINA_TMPDIR: /usr/local/tomcat/temp
Using JRE_HOME:        /usr/local/openjdk-11
Using CLASSPATH:       /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar
NOTE: Picked up JDK_JAVA_OPTIONS:  --add-opens=java.base/java.lang=ALL-UNNAMED --add-opens=java.base/java.io=ALL-UNNAMED --add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED
05-Jul-2020 10:17:53.882 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server version name:   Apache Tomcat/9.0.36
05-Jul-2020 10:17:53.890 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server built:          Jun 3 2020 17:07:09 UTC
05-Jul-2020 10:17:53.890 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server version number: 9.0.36.0
05-Jul-2020 10:17:53.890 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log OS Name:               Linux
05-Jul-2020 10:17:53.890 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log OS Version:            3.10.0-957.el7.x86_64
05-Jul-2020 10:17:53.890 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Architecture:          amd64
05-Jul-2020 10:17:53.890 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Java Home:             /usr/local/openjdk-11
05-Jul-2020 10:17:53.890 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log JVM Version:           11.0.7+10
05-Jul-2020 10:17:53.890 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log JVM Vendor:            Oracle Corporation
05-Jul-2020 10:17:53.890 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log CATALINA_BASE:         /usr/local/tomcat
05-Jul-2020 10:17:53.890 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log CATALINA_HOME:         /usr/local/tomcat
05-Jul-2020 10:17:53.913 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: --add-opens=java.base/java.lang=ALL-UNNAMED
05-Jul-2020 10:17:53.913 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: --add-opens=java.base/java.io=ALL-UNNAMED
05-Jul-2020 10:17:53.913 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: --add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED
05-Jul-2020 10:17:53.913 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.util.logging.config.file=/usr/local/tomcat/conf/logging.properties
05-Jul-2020 10:17:53.913 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager
05-Jul-2020 10:17:53.913 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djdk.tls.ephemeralDHKeySize=2048
05-Jul-2020 10:17:53.913 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.protocol.handler.pkgs=org.apache.catalina.webresources
05-Jul-2020 10:17:53.913 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dorg.apache.catalina.security.SecurityListener.UMASK=0027
05-Jul-2020 10:17:53.913 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dignore.endorsed.dirs=
05-Jul-2020 10:17:53.913 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dcatalina.base=/usr/local/tomcat
05-Jul-2020 10:17:53.914 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dcatalina.home=/usr/local/tomcat
05-Jul-2020 10:17:53.914 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.io.tmpdir=/usr/local/tomcat/temp
05-Jul-2020 10:17:53.914 INFO [main] org.apache.catalina.core.AprLifecycleListener.lifecycleEvent Loaded Apache Tomcat Native library [1.2.24] using APR version [1.6.5].
05-Jul-2020 10:17:53.914 INFO [main] org.apache.catalina.core.AprLifecycleListener.lifecycleEvent APR capabilities: IPv6 [true], sendfile [true], accept filters [false], random [true].
05-Jul-2020 10:17:53.914 INFO [main] org.apache.catalina.core.AprLifecycleListener.lifecycleEvent APR/OpenSSL configuration: useAprConnector [false], useOpenSSL [true]
05-Jul-2020 10:17:53.924 INFO [main] org.apache.catalina.core.AprLifecycleListener.initializeSSL OpenSSL successfully initialized [OpenSSL 1.1.1d  10 Sep 2019]
05-Jul-2020 10:17:54.641 INFO [main] org.apache.coyote.AbstractProtocol.init Initializing ProtocolHandler ["http-nio-8080"]
05-Jul-2020 10:17:54.713 INFO [main] org.apache.catalina.startup.Catalina.load Server initialization in [1,260] milliseconds
05-Jul-2020 10:17:54.836 INFO [main] org.apache.catalina.core.StandardService.startInternal Starting service [Catalina]
05-Jul-2020 10:17:54.836 INFO [main] org.apache.catalina.core.StandardEngine.startInternal Starting Servlet engine: [Apache Tomcat/9.0.36]
05-Jul-2020 10:17:54.862 INFO [main] org.apache.coyote.AbstractProtocol.start Starting ProtocolHandler ["http-nio-8080"]
05-Jul-2020 10:17:54.919 INFO [main] org.apache.catalina.startup.Catalina.start Server startup in [205] milliseconds
[root@rabbitmq001 test]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                     NAMES
a0bb01b9c91c        tomcat              "catalina.sh run"   50 seconds ago      Up 49 seconds       0.0.0.0:32770->8080/tcp   tomcat01
[root@rabbitmq001 test]# docker exec -it tomcat01 ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
# 容器内由一个100→101的
100: eth0@if101: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
# 宿主机能ping通容器
[root@rabbitmq001 test]# ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.052 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.067 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.038 ms
^C
--- 172.17.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1999ms
rtt min/avg/max/mdev = 0.038/0.052/0.067/0.013 ms
[root@rabbitmq001 test]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:50:56:3a:cd:d4 brd ff:ff:ff:ff:ff:ff
    inet 192.168.95.10/24 brd 192.168.95.255 scope global noprefixroute dynamic ens33
       valid_lft 1608sec preferred_lft 1608sec
    inet6 fe80::2249:6aed:1006:87b6/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:e2:c2:af:ca brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:e2ff:fec2:afca/64 scope link 
       valid_lft forever preferred_lft forever
# 宿主机多了一个100-101的网卡
101: veth563bda8@if100: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether 3a:79:1c:54:e6:4a brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::3879:1cff:fe54:e64a/64 scope link 
       valid_lft forever preferred_lft forever
[root@rabbitmq001 test]#

```

![1593945645500](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1593945645500.png)

结论：tomcat01和tomcat02是公用的一个路由器，docker0

所有的容器不指定网络的情况下，都是docker0路由的，docker会给我们的容器分配一个默认的可用IP

![1593946086915](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1593946086915.png)

**docker 一般都是通过服务名访问，因为重启容器，ip会变动**

#### --link

```shell
#tomcat01 ping 不同 tomcat02
[root@rabbitmq001 test]# docker exec -it tomcat01 ping tomcat02
ping: tomcat02: No address associated with hostname
#tomcat03 和 tomcat02 网络相通
[root@rabbitmq001 test]# docker run -d -P --name tomcat03 --link tomcat02 tomcat
4ca3796d6ee7804b2f9caa7db07593eb5af1f8fe4b85eefd15181bf019cbbbb1
#tomcat02 和 tomcat03 网络不相通
[root@rabbitmq001 test]# docker exec -it tomcat02 ping tomcat03
ping: tomcat03: No address associated with hostname
[root@rabbitmq001 test]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                     NAMES
4ca3796d6ee7        tomcat              "catalina.sh run"   52 seconds ago      Up 51 seconds       0.0.0.0:32772->8080/tcp   tomcat03
d991dbca3651        tomcat              "catalina.sh run"   18 minutes ago      Up 18 minutes       0.0.0.0:32771->8080/tcp   tomcat02
a0bb01b9c91c        tomcat              "catalina.sh run"   36 minutes ago      Up 36 minutes       0.0.0.0:32770->8080/tcp   tomcat01
#tomcat03 和 tomcat02 网络相通
[root@rabbitmq001 test]# docker exec -it tomcat03 ping tomcat02
PING tomcat02 (172.17.0.3) 56(84) bytes of data.
64 bytes from tomcat02 (172.17.0.3): icmp_seq=1 ttl=64 time=0.059 ms
64 bytes from tomcat02 (172.17.0.3): icmp_seq=2 ttl=64 time=0.108 ms
64 bytes from tomcat02 (172.17.0.3): icmp_seq=3 ttl=64 time=0.110 ms
^C
--- tomcat02 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.059/0.092/0.110/0.024 ms

```

#### docker network

```shell
# --net bridge 是默认参数，表示的是docker0
docker run -d -P --name tomcat01 --net bridge tomcat

# docker0特点：
# 不写网络默认
# 域名不能访问

#  自定义网络
#	driver bridge 桥接
#	--subnet 192.168.0.0/16 子网
#	--gateway 192.168.0.1 网管
#	mynet 名称
[root@rabbitmq001 test]# docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
b2a1b016e4b121e610b2496e5ab27c9203a1689fedb5b1abe0af9618630f5a72
[root@rabbitmq001 test]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
55d03b18d7fc        bridge              bridge              local
2b858051aeac        host                host                local
b2a1b016e4b1        mynet               bridge              local
0a3c98a01132        none                null                local
[root@rabbitmq001 test]# docker network inspect mynet
[
    {
        "Name": "mynet",
        "Id": "b2a1b016e4b121e610b2496e5ab27c9203a1689fedb5b1abe0af9618630f5a72",
        "Created": "2020-07-05T19:19:58.260541086+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
# 通过自定义的网络mynet，都可以通过nameping通了
[root@rabbitmq001 test]# docker run -d -P --name tomcat-net-01 --net mynet tomcat
bdc879c8861ea7187836c84786e6c9c4f4500cbbcc5410a7bcd9dbf06f300f5c
[root@rabbitmq001 test]# docker run -d -P --name tomcat-net-02 --net mynet tomcat
370e5c4cd6cc81421f048a3158ef6cc2dae887dccd20b00497f0394888033755
[root@rabbitmq001 test]# docker exec -it tomcat-net-01 ping tomcat-net-02
PING tomcat-net-02 (192.168.0.3) 56(84) bytes of data.
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=1 ttl=64 time=0.043 ms
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=2 ttl=64 time=0.093 ms
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=3 ttl=64 time=0.039 ms
^C
--- tomcat-net-02 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 3ms
rtt min/avg/max/mdev = 0.039/0.058/0.093/0.025 ms
[root@rabbitmq001 test]# 

```

**自定义网络的好处：不同的集群使用不同的网络比如redis和mysql**

#### docker network connect

```shell
[root@rabbitmq001 test]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                     NAMES
a64848b4c99b        tomcat              "catalina.sh run"   5 seconds ago       Up 4 seconds        0.0.0.0:32776->8080/tcp   tomcat02
46244136f0e4        tomcat              "catalina.sh run"   9 seconds ago       Up 8 seconds        0.0.0.0:32775->8080/tcp   tomcat01
370e5c4cd6cc        tomcat              "catalina.sh run"   7 minutes ago       Up 7 minutes        0.0.0.0:32774->8080/tcp   tomcat-net-02
bdc879c8861e        tomcat              "catalina.sh run"   7 minutes ago       Up 7 minutes        0.0.0.0:32773->8080/tcp   tomcat-net-01
[root@rabbitmq001 test]# docker network connect --help

Usage:	docker network connect [OPTIONS] NETWORK CONTAINER

Connect a container to a network

Options:
      --alias strings           Add network-scoped alias for the container
      --driver-opt strings      driver options for the network
      --ip string               IPv4 address (e.g., 172.30.100.104)
      --ip6 string              IPv6 address (e.g., 2001:db8::33)
      --link list               Add link to another container
      --link-local-ip strings   Add a link-local address for the container
# 将一个容器加到网络中
[root@rabbitmq001 test]# docker network connect mynet tomcat01
[root@rabbitmq001 test]# docker network inspect mynet
[
    {
        "Name": "mynet",
        "Id": "b2a1b016e4b121e610b2496e5ab27c9203a1689fedb5b1abe0af9618630f5a72",
        "Created": "2020-07-05T19:19:58.260541086+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "370e5c4cd6cc81421f048a3158ef6cc2dae887dccd20b00497f0394888033755": {
                "Name": "tomcat-net-02",
                "EndpointID": "8a5f6f0517448fd88aa364f26a774c3a3d3d850af25fed7d151663b4415b51cb",
                "MacAddress": "02:42:c0:a8:00:03",
                "IPv4Address": "192.168.0.3/16",
                "IPv6Address": ""
            },
            # 会发现tomcat01被加到mynet中了
            "46244136f0e495f1462adce8fc4497f670417c99ea1958eb81e2487bc3159795": {
                "Name": "tomcat01",
                "EndpointID": "9ed45298539ba593bf854b14f426680b8893791a01749b0ec3bdcbb93a7972fb",
                "MacAddress": "02:42:c0:a8:00:04",
                "IPv4Address": "192.168.0.4/16",
                "IPv6Address": ""
            },
            "bdc879c8861ea7187836c84786e6c9c4f4500cbbcc5410a7bcd9dbf06f300f5c": {
                "Name": "tomcat-net-01",
                "EndpointID": "cc24b7f3dae5f57bcf2321141b58610c03f826baf20d0b8fbd174e42a3d011e4",
                "MacAddress": "02:42:c0:a8:00:02",
                "IPv4Address": "192.168.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
[root@rabbitmq001 test]# 

```

### springboot 用docker部署项目

Dockerfile编写

```
FROM java:8

COPY *.jar /app.jar

CMD ["--server.port=8080"]

EXPOSE 8080

ENTRYPOINT ["java","-jar","/app.jar"]
```

