##   Docker的常用命令

### 帮助命令

```shell
 docker version #显示docker的版本信息
 docker info #显示docker的系统信息，包括镜像和容器的数量等
 docker [命令] --help 	#帮助命令
```

帮助文档地址： https://docs.docker.com/reference/ 

![1593875953234](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1593875953234.png)

![1593875913803](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\1593875913803.png)

### 镜像命令

#### 	docker images

```shell
[root@rabbitmq001 ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              bf756fb1ae65        6 months ago        13.3kB

# 解释
# REPOSITORY 镜像的仓库源
# TAG 镜像的标签
# IMAGE ID 镜像的id
# CREATED 镜像的创建时间
# SIZE 镜像的大小
```

#### 	docker search

```SHELL
docker search mysql --filter=STARS=5000
```

#### 	docker pull

```shell
#下载镜像 docker pull 镜像名[:tag]
[root@rabbitmq001 ~]# docker pull mysql
Using default tag: latest #如果不写tag,默认就是 latest
latest: Pulling from library/mysql
8559a31e96f4: Pull complete  # 分层下载,docker image的核心 联合文件系统
d51ce1c2e575: Pull complete 
c2344adc4858: Pull complete 
fcf3ceff18fc: Pull complete 
16da0c38dc5b: Pull complete 
b905d1797e97: Pull complete 
4b50d1c6b05c: Pull complete 
c75914a65ca2: Pull complete 
1ae8042bdd09: Pull complete 
453ac13c00a3: Pull complete 
9e680cd72f08: Pull complete 
a6b5dc864b6c: Pull complete 
Digest: sha256:8b7b328a7ff6de46ef96bcf83af048cb00a1c86282bfca0cb119c84568b4caf6
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest #真实地址,等同于docker pull mysql
[root@rabbitmq001 ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
mysql               latest              be0dbf01a0f3        3 weeks ago         541MB
hello-world         latest              bf756fb1ae65        6 months ago        13.3kB
[root@rabbitmq001 ~]# docker pull mysql:5.7 #指定版本
5.7: Pulling from library/mysql
8559a31e96f4: Already exists 
d51ce1c2e575: Already exists 
c2344adc4858: Already exists 
fcf3ceff18fc: Already exists 
16da0c38dc5b: Already exists 
b905d1797e97: Already exists 
4b50d1c6b05c: Already exists 
d85174a87144: Pull complete 
a4ad33703fa8: Pull complete 
f7a5433ce20d: Pull complete 
3dcd2a278b4a: Pull complete 
Digest: sha256:32f9d9a069f7a735e28fd44ea944d53c61f990ba71460c5c183e610854ca4854
Status: Downloaded newer image for mysql:5.7
docker.io/library/mysql:5.7
```

#### 	docker rmi 

```shell
#单个删除,多个删除可以指定多个容器id
[root@rabbitmq001 ~]# docker rmi -f 9cfcce23593a 
Untagged: mysql:5.7
Untagged: mysql@sha256:32f9d9a069f7a735e28fd44ea944d53c61f990ba71460c5c183e610854ca4854
Deleted: sha256:9cfcce23593a93135ca6dbf3ed544d1db9324d4c40b5c0d56958165bfaa2d46a
Deleted: sha256:98de3e212919056def8c639045293658f6e6022794807d4b0126945ddc8324be
Deleted: sha256:17e8b88858e400f8c5e10e7cb3fbab9477f6d8aacba03b8167d34a91dbe4d8c1
Deleted: sha256:c04c087c2af9abd64ba32fe89d65e6d83da514758923de5da154541cc01a3a1e
Deleted: sha256:ab8bf065b402b99aec4f12c648535ef1b8dc954b4e1773bdffa10ae2027d3e00

#递归删除
[root@rabbitmq001 ~]# docker rmi -f $(docker images -aq)
Untagged: mysql:latest
Untagged: mysql@sha256:8b7b328a7ff6de46ef96bcf83af048cb00a1c86282bfca0cb119c84568b4caf6
Deleted: sha256:be0dbf01a0f3f46fc8c88b67696e74e7005c3e16d9071032fa0cd89773771576
Deleted: sha256:086d66e8d1cb0d52e9337eabb11fb9b95960e2e1628d90100c62ea5e8bf72306
Deleted: sha256:f37c61ee1973b18c285d0d5fcf02da4bcdb1f3920981499d2a20b2858500a110
Deleted: sha256:e40b8bca7dc63fc8d188a412328e56caf179022f5e5d5b323aae57d233fb1069
Deleted: sha256:339f6b96b27eb035cbedc510adad2560132925a835f0afddbcc1d311c961c14b
Deleted: sha256:d38b06cdb26a5c98857ddbc6ef531d3f57b00e325c0c314600b712efc7ff6ab0
Deleted: sha256:09687cd9cdf4c704fde969fdba370c2d848bc614689712bef1a31d0d581f2007
Deleted: sha256:b704a4a65bf536f82e5d8b86e633d19185e26313de8380162e778feb2852011a
Deleted: sha256:c37206160543786228aa0cce738e85343173851faa44bb4dc07dc9b7dc4ff1c1
Deleted: sha256:12912c9ec523f648130e663d9d4f0a47c1841a0064d4152bcf7b2a97f96326eb
Deleted: sha256:57d29ad88aa49f0f439592755722e70710501b366e2be6125c95accc43464844
Deleted: sha256:b17c024283d0302615c6f0c825137da9db607d49a83d2215a79733afbbaeb7c3
Deleted: sha256:13cb14c2acd34e45446a50af25cb05095a17624678dbafbcc9e26086547c1d74
Untagged: hello-world:latest
Untagged: hello-world@sha256:d58e752213a51785838f9eed2b7a498ffa1cb3aa7f946dda11af39286c3db9a9
Deleted: sha256:bf756fb1ae65adf866bd8c456593cd24beb6a0a061dedf42b26a993176745f6b

```



### 容器命令

#### 	run

```shell
docker run centos
# docker run [可选参数] image
#参数说明
# --name="name" 容器名字 tomcat nginx1 nginx2
# -d 	后台方式运行
# -it	使用交互方式运行，进入容器查看内容
# -p(小写)	指定容器的端口 -p 8080:80
#	-p ip:宿主机端口：容器端口
#	-p 宿主机端口：容器端口
#	-p 容器端口
# -P(大写)	随机指定端口
#	-rm 启动完即删除容器
[root@rabbitmq001 ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
centos              latest              831691599b88        2 weeks ago         215MB
#启动并进入容器
[root@rabbitmq001 ~]# docker run -it centos /bin/bash
WARNING: IPv4 forwarding is disabled. Networking will not work.
[root@43cce1db0fa4 /]# ##这里已经进入容器了 
[root@43cce1db0fa4 /]# exit #退出
exit
```

#### ps

```shell
[root@rabbitmq001 ~]# docker ps	#查看正在运行的容器
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@rabbitmq001 ~]# docker ps -a	#查看所有的容器
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                            PORTS               NAMES
43cce1db0fa4        centos              "/bin/bash"         5 minutes ago       Exited (127) About a minute ago                       trusting_hypatia
c263cb251369        bf756fb1ae65        "/hello"            About an hour ago   Exited (0) About an hour ago                          boring_leavitt
[root@rabbitmq001 ~]# docker ps -a -n=1	#显示最近创建的容器
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                       PORTS               NAMES
43cce1db0fa4        centos              "/bin/bash"         7 minutes ago       Exited (127) 3 minutes ago                       trusting_hypatia
[root@rabbitmq001 ~]# docker ps -aq	#显示所有容器id
43cce1db0fa4
c263cb251369
```

#### 退出容器

```shell
exit #停止并退出
ctrl + p + q 	#退出不回停止容器
```

#### docker rm

```shell
docker rm [容器id]	#删除指定容器，不能删除正在运行的容器
docker rm -f [容器id]	#删除包括运行的
docker rm -f $(docker ps -aq)	#删除所有容器，包括在运行的
docker ps -a -q|xargs docker rm -f #删除所有容器，包括在运行的 
```

```shell
[root@rabbitmq001 ~]# docker rm c263cb251369
c263cb251369
[root@rabbitmq001 ~]# docker rm 8c04f8ddad58
Error response from daemon: You cannot remove a running container 8c04f8ddad588db016f772f98dabe9f06a3c44b01ac91bd66093c2f62d7684cd. Stop the container before attempting removal or force remove
[root@rabbitmq001 ~]# docker rm -f 8c04f8ddad58
8c04f8ddad58
[root@rabbitmq001 ~]# docker rm -f $(docker ps -aq)
43cce1db0fa4
[root@rabbitmq001 ~]# 
```

#### docker start

```shell
docker start <容器id>	#启动容器
```

#### docker restart

```shell
docker restart <容器id>	#重启容器
```

#### docker stop

```shell
docker stop <容器id>	#停止一个在运行的容器
```

#### docker kill

```shell
docker kill <容器id>	#强制停止当前容器
```

### 常用其他命令

#### 	后台启动容器

```shell
# 命令 docker run -d 镜像名
[root@rabbitmq001 ~]# docker run -d centos
WARNING: IPv4 forwarding is disabled. Networking will not work.
09534e08e4d4044d5f3252b23a0ad5e013b0e36c1a859a3948499bd80c99b162
[root@rabbitmq001 ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@rabbitmq001 ~]# 
# 问题docker ps，发现没有容器启动
#docker使用后台启动运行，就必须要有一个前台进程，docker发现没有应用，就会自行停止容器
```

#### 	查看日志

```shell
#监控日志后10条
docker logs -t -f --tail 10 [容器id]
```

#### 	docker top 

```shell
# 查看容器中进程
# docker top [容器id]
[root@rabbitmq001 ~]# docker top 1f0aeb255b00
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                100031              100015              0                   00:35               ?                   00:00:00            /bin/bash

```

#### 	docker inspect

```shell
[root@rabbitmq001 ~]# docker inspect 1f0aeb255b00
[
    {
        "Id": "1f0aeb255b00a954ae1c3456db65e006336401e15b1f6f1810fa91575898a0e7",
        "Created": "2020-07-04T16:35:27.456839221Z",
        "Path": "/bin/bash",
        "Args": [],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 100031,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2020-07-04T16:35:27.779522221Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:831691599b88ad6cc2a4abbd0e89661a121aff14cfa289ad840fd3946f274f1f",
        "ResolvConfPath": "/var/lib/docker/containers/1f0aeb255b00a954ae1c3456db65e006336401e15b1f6f1810fa91575898a0e7/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/1f0aeb255b00a954ae1c3456db65e006336401e15b1f6f1810fa91575898a0e7/hostname",
        "HostsPath": "/var/lib/docker/containers/1f0aeb255b00a954ae1c3456db65e006336401e15b1f6f1810fa91575898a0e7/hosts",
        "LogPath": "/var/lib/docker/containers/1f0aeb255b00a954ae1c3456db65e006336401e15b1f6f1810fa91575898a0e7/1f0aeb255b00a954ae1c3456db65e006336401e15b1f6f1810fa91575898a0e7-json.log",
        "Name": "/cranky_bassi",
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
                "LowerDir": "/var/lib/docker/overlay2/fd3364edbbf29ade015062e10800c5d4ddb218bfd162da37adefeb8da533cc71-init/diff:/var/lib/docker/overlay2/af13b024aeb86a6af03c115bc5c4777017ef4bb33cd84945504f02a0d5463d52/diff",
                "MergedDir": "/var/lib/docker/overlay2/fd3364edbbf29ade015062e10800c5d4ddb218bfd162da37adefeb8da533cc71/merged",
                "UpperDir": "/var/lib/docker/overlay2/fd3364edbbf29ade015062e10800c5d4ddb218bfd162da37adefeb8da533cc71/diff",
                "WorkDir": "/var/lib/docker/overlay2/fd3364edbbf29ade015062e10800c5d4ddb218bfd162da37adefeb8da533cc71/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "1f0aeb255b00",
            "Domainname": "",
            "User": "",
            "AttachStdin": true,
            "AttachStdout": true,
            "AttachStderr": true,
            "Tty": true,
            "OpenStdin": true,
            "StdinOnce": true,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/bash"
            ],
            "Image": "centos",
            "Volumes": null,
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
            "SandboxID": "4ae94c9700029ec59127b3f6aba181dafd58742f38030b1b994bea21cf650170",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/4ae94c970002",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "5a33c17099cd5478c84100c8c657d8030434af50efa4e4b271b3ff51e62c877d",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "03981de8ed5a7619a7ae74ccd2b7311856d1f4b754e49833cde49af9aaae1787",
                    "EndpointID": "5a33c17099cd5478c84100c8c657d8030434af50efa4e4b271b3ff51e62c877d",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]

```

#### 	进入当前正在运行的容器

```shell

[root@rabbitmq001 ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
1f0aeb255b00        centos              "/bin/bash"         11 minutes ago      Up 11 minutes                           cranky_bassi
# docker exec -it <容器id> /bin/bash
[root@rabbitmq001 ~]# docker exec -it 1f0aeb255b00 /bin/bash
# docker attach 1f0aeb255b00
# exec 是进入容器后开启一个新的终端，可以在里面操作
# docker attach 进入容器正在执行的终端，不会启动新的进程
[root@1f0aeb255b00 /]# 
```

#### 	从容器内拷贝文件到主机上

```shell
# docker cp 容器id:容器内路径 宿主机目录
[root@rabbitmq001 ~]# docker cp 1f0aeb255b00:/home/hello.json /home
[root@rabbitmq001 ~]# cd /home/
[root@rabbitmq001 home]# ll
total 0
drwx------. 3 elasticsearch elasticsearch 114 Jul  4 14:48 elasticsearch
-rw-r--r--. 1 root          root            0 Jul  5 00:59 hello.json
drwx------. 2 lusifer       lusifer        62 Apr 11  2018 lusifer

```

#### 	docker stats

```shell
# 查看cpu状态
[root@rabbitmq001 home]# docker stats
CONTAINER ID        NAME                CPU %               MEM USAGE / LIMIT     MEM %               NET I/O             BLOCK I/O           PIDS
f48adf7cc0fb        nginx1              0.00%               1.422MiB / 3.683GiB   0.04%               2.4kB / 2.3kB       0B / 0B             2
^C
[root@rabbitmq001 home]# 

```



### 练习

#### 	安装nginx

```shell
[root@rabbitmq001 home]# docker run -d --name nginx1 -p 8000:80  nginx
WARNING: IPv4 forwarding is disabled. Networking will not work.
0178b62ddcf58c446c163354bbfcb4e40653ee3408d39ce8a772fcc7dd4d0e8d

```

#### 安装tomcat

```shell
[root@rabbitmq001 ~]# docker pull tomcat
Using default tag: latest
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
docker.io/library/tomcat:latest
[root@rabbitmq001 ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
centos              latest              831691599b88        2 weeks ago         215MB
tomcat              latest              2eb5a120304e        3 weeks ago         647MB
nginx               latest              2622e6cca7eb        3 weeks ago         132MB
[root@rabbitmq001 ~]# docker run -d -p 8001:8080 --name tomcat01 tomcat
1e37b255ef788e3b80c36e3481b41ccacf255232e0e8e0a0de789581462e840f
[root@rabbitmq001 ~]# docker exec -it tomcat01 /bin/bash
root@1e37b255ef78:/usr/local/tomcat# ls
BUILDING.txt	 LICENSE  README.md	 RUNNING.txt  conf  logs	    temp     webapps.dist
CONTRIBUTING.md  NOTICE   RELEASE-NOTES  bin	      lib   native-jni-lib  webapps  work

```

#### 安装es和kibana

```shell
docker run  --name elasticsearch  -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.7.0
```

### 可视化

#### 	portainer

```
docker run -d -p 8088:9000 \
--restart=always -v /var/run/dokcer.sock:/var/run/docker.sock --privileged=true portainer/portainer
```

### commit镜像

```shell
# docker commit  提交容器成一个新的副本
# docker commit -m="提交的描述信息" -a="作者" [容器id] 目标镜像：[TAG]
[root@rabbitmq001 ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                    NAMES
2536981e0351        tomcat              "catalina.sh run"   21 seconds ago      Up 20 seconds       0.0.0.0:8082->8080/tcp   tomcat02
[root@rabbitmq001 ~]# docker exec -it 2536981e0351 /bin/bash
root@2536981e0351:/usr/local/tomcat# cd webapps
root@2536981e0351:/usr/local/tomcat# ls
BUILDING.txt	 LICENSE  README.md	 RUNNING.txt  conf  logs	    temp     webapps.dist
CONTRIBUTING.md  NOTICE   RELEASE-NOTES  bin	      lib   native-jni-lib  webapps  work
root@2536981e0351:/usr/local/tomcat# cp -r webapps.dist/*  webapps
root@2536981e0351:/usr/local/tomcat# cd webapps
root@2536981e0351:/usr/local/tomcat/webapps# ls
ROOT  docs  examples  host-manager  manager
root@2536981e0351:/usr/local/tomcat/webapps# exit
exit
[root@rabbitmq001 ~]# docker commit -a="st" -m="empty tomcat" 2536981e0351 tomcat02:1.0
sha256:560f2d2b7746cd3ba0365f9f3fab1d1bca8e8c0b5c12f778e1635e80a93e25ff
[root@rabbitmq001 ~]# docker images
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
tomcat02              1.0                 560f2d2b7746        3 seconds ago       652MB
centos                latest              831691599b88        2 weeks ago         215MB
redis                 latest              235592615444        3 weeks ago         104MB
tomcat                latest              2eb5a120304e        3 weeks ago         647MB
nginx                 latest              2622e6cca7eb        3 weeks ago         132MB
portainer/portainer   latest              cd645f5a4769        4 weeks ago         79.1MB
elasticsearch         7.7.0               7ec4f35ab452        7 weeks ago         757MB
elasticsearch         latest              5acf0e8da90b        21 months ago       486MB
[root@rabbitmq001 ~]# 

```

