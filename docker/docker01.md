

# docker

## docker概述

### 出现原因

* 一次编译,~~到处报错~~
* 开发环境和生产环境需要分别搭建
* 搭建环境的繁杂,特别是搭建集群
* docker出现,将应用和环境一起打包. jar+(mysql+redis+tomcat...)

### 官网

[docker官网](https://www.docker.com/)

[docker文档](https://docs.docker.com/)

[docker hub](https://hub.docker.com/)

### 虚拟化与容器化

> 虚拟化

* 模拟硬件,一个完整的操作系统
* 占用资源多
* 启动慢

> 容器化

* 不模拟硬件
* 没有自己内核,使用宿主机的内核
* 容器隔离,互不干扰

> DevOps

* 更快的交付与部署
* 更边界的升级
* 更简单的运维
* 更高效的资源利用

### docker架构图

![](https://img-blog.csdnimg.cn/img_convert/84b68abdf269c09e2ffd3716df08f690.png)



**镜像(image):**

创建容器的模板.类似于class

**容器(container):**

类似于对象

**仓库(repository):**

私有仓库和共有仓库

## docker安装

> 环境 centos7以上  xshell连接

```shell
uname -a
cat /etc/os-release
```

[install docker on centos](https://docs.docker.com/engine/install/centos/)

```shell
docker version
docker info
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/675a957f25f748d7a93f25784ac829ef.png#pic_center)


```shell
docker run hello-world

docker images
```

## 阿里云镜像加速

```shell
 sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://igh3g3of.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 底层原理

* docker是cs架构的系统,docker守护进程运行在主机上,通过socket从客户端访问
* 每一个docker容器就是一个小的linux虚拟机,有着独立的端口号,和宿主机隔离,和其他docker容器隔离
* ![](https://img-blog.csdnimg.cn/img_convert/d9248e5d238509ada420da57da31490f.png)

## Docker常用命令

### 帮助命令

```shell
docker version
docker info
docker  --help
```

[docker docs](https://docs.docker.com/reference/)

### 镜像命令

* docker images

```shell
docker images # 查看所有本地镜像
Usage:  docker images [OPTIONS] [REPOSITORY[:TAG]]

List images

Options:
  -a, --all             Show all images (default hides intermediate images)
      --digests         Show digests
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print images using a Go template
      --no-trunc        Don't truncate output
  -q, --quiet           Only show image IDs

[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker images -a
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
ubuntu        20.04     a0ce5a295b63   3 months ago    72.8MB
mysql         latest    ff3b5098b416   3 months ago    447MB
hello-world   latest    feb5d9fea6a5   14 months ago   13.3kB
```

* docker search

```shell
docker search # 搜索镜像  也可以去docker hub上找
[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker search --help

Usage:  docker search [OPTIONS] TERM

Search the Docker Hub for images

Options:
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print search using a Go template
      --limit int       Max number of search results (default 25)
      --no-trunc        Don't truncate output

[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker search mysql
NAME                            DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql                           MySQL is a widely used, open-source relation…   13544     [OK]       
mariadb                         MariaDB Server is a high performing open sou…   5165      [OK]       
phpmyadmin                      phpMyAdmin - A web interface for MySQL and M…   699       [OK]       
percona                         Percona Server is a fork of the MySQL relati…   595       [OK]       
[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker search mysql --filter=STARS=5000  #查找Stars>=5000
NAME      DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql     MySQL is a widely used, open-source relation…   13544     [OK]       
mariadb   MariaDB Server is a high performing open sou…   5165      [OK]       

```

* docker pull

```shell
docker pull # 拉取镜像
[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker pull --help

Usage:  docker pull [OPTIONS] NAME[:TAG|@DIGEST]

Pull an image or a repository from a registry

Options:
  -a, --all-tags                Download all tagged images in the repository
      --disable-content-trust   Skip image verification (default true)
      --platform string         Set platform if server is multi-platform capable
  -q, --quiet                   Suppress verbose output


[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker pull nginx  # 不写tag默认latest 表示最新版
Using default tag: latest
latest: Pulling from library/nginx
a2abf6c4d29d: Pull complete   # 分层下载
a9edb18cadd1: Pull complete 
589b7251471a: Pull complete 
186b1aaa4aa6: Pull complete 
b4df32aa5a72: Pull complete 
a0bcbecc962e: Pull complete 
Digest: sha256:0d17b565c37bcbd895e9d92315a05c1c3c9a29f762b011a10c54a66cd53c9b31  # sha256校验
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest

[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
ubuntu        20.04     a0ce5a295b63   3 months ago    72.8MB
mysql         latest    ff3b5098b416   3 months ago    447MB
nginx         latest    605c77e624dd   11 months ago   141MB
hello-world   latest    feb5d9fea6a5   14 months ago   13.3kB

```

* docker rmi  # 删除镜像

```shell
docker rmi  # 删除镜像
[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker rmi --help

Usage:  docker rmi [OPTIONS] IMAGE [IMAGE...]

Remove one or more images

Options:
  -f, --force      Force removal of the image
      --no-prune   Do not delete untagged parents

[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker rmi -f hello-world:latest  # 使用名称删除
Untagged: hello-world:latest
Untagged: hello-world@sha256:7d246653d0511db2a6b2e0436cfd0e52ac8c066000264b3ce63331ac66dca625
Deleted: sha256:feb5d9fea6a5e9606aa995e879d862b825965ba48de054caab5ef356dc6b3412
[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
ubuntu       20.04     a0ce5a295b63   3 months ago    72.8MB
mysql        latest    ff3b5098b416   3 months ago    447MB
nginx        latest    605c77e624dd   11 months ago   141MB

[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker rmi -f 605c77e624dd # 使用id删除
Untagged: nginx:latest
Untagged: nginx@sha256:0d17b565c37bcbd895e9d92315a05c1c3c9a29f762b011a10c54a66cd53c9b31
Deleted: sha256:605c77e624ddb75e6110f997c58876baa13f8754486b461117934b24a9dc3a85
Deleted: sha256:b625d8e29573fa369e799ca7c5df8b7a902126d2b7cbeb390af59e4b9e1210c5
Deleted: sha256:7850d382fb05e393e211067c5ca0aada2111fcbe550a90fed04d1c634bd31a14
Deleted: sha256:02b80ac2055edd757a996c3d554e6a8906fd3521e14d1227440afd5163a5f1c4
Deleted: sha256:b92aa5824592ecb46e6d169f8e694a99150ccef01a2aabea7b9c02356cdabe7c
Deleted: sha256:780238f18c540007376dd5e904f583896a69fe620876cabc06977a3af4ba4fb5
Deleted: sha256:2edcec3590a4ec7f40cf0743c15d78fb39d8326bc029073b41ef9727da6c851f
[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
ubuntu       20.04     a0ce5a295b63   3 months ago   72.8MB
mysql        latest    ff3b5098b416   3 months ago   447MB
```



### 容器命令

> 有了镜像才可以创建容器

```shell
docker pull centos:centos7.9.2009
[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker pull centos:centos7.9.2009
centos7.9.2009: Pulling from library/centos
2d473b07cdd5: Pull complete 
Digest: sha256:9d4bcbbb213dfd745b58be38b13b996ebb5ac315fe75711bd618426a630e0987
Status: Downloaded newer image for centos:centos7.9.2009
docker.io/library/centos:centos7.9.2009
```

- docker run

```shell
docker run
[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker run --help

Usage:  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

Run a command in a new container

Options:
      --add-host list                  Add a custom host-to-IP mapping (host:ip)
  -a, --attach list                    Attach to STDIN, STDOUT or STDERR
      --blkio-weight uint16            Block IO (relative weight), between 10 and 1000, or 0 to disable (default 0)
      --blkio-weight-device list       Block IO weight (relative device weight) (default [])
      --cap-add list                   Add Linux capabilities
      --cap-drop list                  Drop Linux capabilities
      --cgroup-parent string           Optional parent cgroup for the container
      --cgroupns string                Cgroup namespace to use (host|private)
                                       'host':    Run the container in the Docker host's cgroup namespace
                                       'private': Run the container in its own private cgroup namespace
                                       '':        Use the cgroup namespace as configured by the
                                                  default-cgroupns-mode option on the daemon (default)
      --cidfile string                 Write the container ID to the file
      --cpu-period int                 Limit CPU CFS (Completely Fair Scheduler) period
      --cpu-quota int                  Limit CPU CFS (Completely Fair Scheduler) quota
      --cpu-rt-period int              Limit CPU real-time period in microseconds
      --cpu-rt-runtime int             Limit CPU real-time runtime in microseconds
  -c, --cpu-shares int                 CPU shares (relative weight)
      --cpus decimal                   Number of CPUs
      --cpuset-cpus string             CPUs in which to allow execution (0-3, 0,1)
      --cpuset-mems string             MEMs in which to allow execution (0-3, 0,1)
  -d, --detach                         Run container in background and print container ID
      --detach-keys string             Override the key sequence for detaching a container
      --device list                    Add a host device to the container
      --device-cgroup-rule list        Add a rule to the cgroup allowed devices list
      --device-read-bps list           Limit read rate (bytes per second) from a device (default [])
      --device-read-iops list          Limit read rate (IO per second) from a device (default [])
      --device-write-bps list          Limit write rate (bytes per second) to a device (default [])
      --device-write-iops list         Limit write rate (IO per second) to a device (default [])
      --disable-content-trust          Skip image verification (default true)
      --dns list                       Set custom DNS servers
      --dns-option list                Set DNS options
      --dns-search list                Set custom DNS search domains
      --domainname string              Container NIS domain name
      --entrypoint string              Overwrite the default ENTRYPOINT of the image
  -e, --env list                       Set environment variables
      --env-file list                  Read in a file of environment variables
      --expose list                    Expose a port or a range of ports
      --gpus gpu-request               GPU devices to add to the container ('all' to pass all GPUs)
      --group-add list                 Add additional groups to join
      --health-cmd string              Command to run to check health
      --health-interval duration       Time between running the check (ms|s|m|h) (default 0s)
      --health-retries int             Consecutive failures needed to report unhealthy
      --health-start-period duration   Start period for the container to initialize before starting health-retries countdown (ms|s|m|h) (default 0s)
      --health-timeout duration        Maximum time to allow one check to run (ms|s|m|h) (default 0s)
      --help                           Print usage
  -h, --hostname string                Container host name
      --init                           Run an init inside the container that forwards signals and reaps processes
  -i, --interactive                    Keep STDIN open even if not attached
      --ip string                      IPv4 address (e.g., 172.30.100.104)
      --ip6 string                     IPv6 address (e.g., 2001:db8::33)
      --ipc string                     IPC mode to use
      --isolation string               Container isolation technology
      --kernel-memory bytes            Kernel memory limit
  -l, --label list                     Set meta data on a container
      --label-file list                Read in a line delimited file of labels
      --link list                      Add link to another container
      --link-local-ip list             Container IPv4/IPv6 link-local addresses
      --log-driver string              Logging driver for the container
      --log-opt list                   Log driver options
      --mac-address string             Container MAC address (e.g., 92:d0:c6:0a:29:33)
  -m, --memory bytes                   Memory limit
      --memory-reservation bytes       Memory soft limit
      --memory-swap bytes              Swap limit equal to memory plus swap: '-1' to enable unlimited swap
      --memory-swappiness int          Tune container memory swappiness (0 to 100) (default -1)
      --mount mount                    Attach a filesystem mount to the container
      --name string                    ==Assign a name to the container
      --network network                Connect a container to a network
      --network-alias list             Add network-scoped alias for the container
      --no-healthcheck                 Disable any container-specified HEALTHCHECK
      --oom-kill-disable               Disable OOM Killer
      --oom-score-adj int              Tune host's OOM preferences (-1000 to 1000)
      --pid string                     PID namespace to use
      --pids-limit int                 Tune container pids limit (set -1 for unlimited)
      --platform string                Set platform if server is multi-platform capable
      --privileged                     Give extended privileges to this container
  -p, --publish list                   Publish a container's port(s) to the host
  -P, --publish-all                    Publish all exposed ports to random ports
      --pull string                    Pull image before running ("always"|"missing"|"never") (default "missing")
      --read-only                      Mount the container's root filesystem as read only
      --restart string                 Restart policy to apply when a container exits (default "no")
      --rm                             Automatically remove the container when it exits
      --runtime string                 Runtime to use for this container
      --security-opt list              Security Options
      --shm-size bytes                 Size of /dev/shm
      --sig-proxy                      Proxy received signals to the process (default true)
      --stop-signal string             Signal to stop a container (default "SIGTERM")
      --stop-timeout int               Timeout (in seconds) to stop a container
      --storage-opt list               Storage driver options for the container
      --sysctl map                     Sysctl options (default map[])
      --tmpfs list                     Mount a tmpfs directory
  -t, --tty                            Allocate a pseudo-TTY
      --ulimit ulimit                  Ulimit options (default [])
  -u, --user string                    Username or UID (format: <name|uid>[:<group|gid>])
      --userns string                  User namespace to use
      --uts string                     UTS namespace to use
  -v, --volume list                    Bind mount a volume
      --volume-driver string           Optional volume driver for the container
      --volumes-from list              Mount volumes from the specified container(s)
  -w, --workdir string                 Working directory inside the container


--name="Name" 给容器取名字来区分容器
-d 			  后台运行
-it			  使用交互式运行进入容器
-p 			  指定容器端口 -p 8080:8080  主机端口:容器端口  小写p
-P			  随机指定端口 大写P

[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker run -it centos:centos7.9.2009 /bin/bash
[root@80beffa4fe2a /]# ls  # 容器centos7.9.2009内部信息
anaconda-post.log  bin  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@80beffa4fe2a /]# exit
exit
[root@iZ2zef4iumnunw6gqcbjxsZ ~]# 

```

* docker ps

```shell
[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker ps --help

Usage:  docker ps [OPTIONS]

List containers

Options:
  -a, --all             Show all containers (default shows just running)
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print containers using a Go template
  -n, --last int        Show n last created containers (includes all states) (default -1)
  -l, --latest          Show the latest created container (includes all states)
      --no-trunc        Don't truncate output
  -q, --quiet           Only display container IDs
  -s, --size            Display total file sizes
  
  
[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED        STATUS      PORTS                                                  NAMES
1b6c89113558   mysql:latest   "docker-entrypoint.s…"   2 months ago   Up 3 days   33060/tcp, 0.0.0.0:3307->3306/tcp, :::3307->3306/tcp   mysql
[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker ps -a
CONTAINER ID   IMAGE                   COMMAND                  CREATED         STATUS                          PORTS                                                  NAMES
80beffa4fe2a   centos:centos7.9.2009   "/bin/bash"              4 minutes ago   Exited (0) About a minute ago                                                          gracious_dijkstra
f4818b63d749   feb5d9fea6a5            "/hello"                 3 days ago      Exited (0) 3 days ago                                                                  vigorous_ramanujan
aaac99563c0d   ubuntu:20.04            "/bin/bash"              4 days ago      Exited (0) 4 days ago                                                                  hopeful_boyd
40974b298e2e   feb5d9fea6a5            "/hello"                 4 days ago      Exited (0) 4 days ago                                                                  interesting_gauss
1b6c89113558   mysql:latest            "docker-entrypoint.s…"   2 months ago    Up 3 days                       33060/tcp, 0.0.0.0:3307->3306/tcp, :::3307->3306/tcp   mysql
5a4c2c0798ad   ubuntu:20.04            "/bin/bash"              2 months ago    Exited (127) 2 months ago                                                              vigilant_morse
e0a48f4c0351   feb5d9fea6a5            "/hello"                 2 months ago    Exited (0) 2 months ago                                                                magical_bassi

```

* 退出

```shell
exit 关闭容器并退出
ctrl + p + q # 不关闭容器退出
```

* docker rm

```shell
[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker rm --help

Usage:  docker rm [OPTIONS] CONTAINER [CONTAINER...]  # 用来删除容器

Remove one or more containers

Options:
  -f, --force     Force the removal of a running container (uses SIGKILL)
  -l, --link      Remove the specified link
  -v, --volumes   Remove anonymous volumes associated with the container
  
docker rm 容器id
docker rm -f $(docker ps -aq) # 删除所有容器
```

* 启动和停止容器

```shell
docker start 容器id
docker restart 容器id
docker stop  id  # 关闭
docker kill  id  # 强制关闭
```

### 常用其他命令

* docker logs

```shell
[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker logs --help

Usage:  docker logs [OPTIONS] CONTAINER

Fetch the logs of a container

Options:
      --details        Show extra details provided to logs
  -f, --follow         Follow log output
      --since string   Show logs since timestamp (e.g. 2013-01-02T13:23:37Z) or relative (e.g. 42m for 42 minutes)
  -n, --tail string    Number of lines to show from the end of the logs (default "all")
  -t, --timestamps     Show timestamps
      --until string   Show logs before a timestamp (e.g. 2013-01-02T13:23:37Z) or relative (e.g. 42m for 42 minutes)
[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker logs mysql
2022-09-15 09:17:58+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.30-1.el8 started.
2022-09-15 09:17:58+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2022-09-15 09:17:58+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.30-1.el8 started.
2022-09-15 09:17:59+00:00 [Note] [Entrypoint]: Initializing database files
2022-09-15T09:17:59.099655Z 0 [Warning] [MY-011068] [Server] The syntax '--skip-host-cache' is deprecated and will be removed in a future release. Please use SET GLOBAL host_cache_size=0 instead.
2022-09-15T09:17:59.099746Z 0 [System] [MY-013169] [Server] /usr/sbin/mysqld (mysqld 8.0.30) initializing of server in progress as process 78
2022-09-15T09:17:59.111770Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2022-09-15T09:18:00.045694Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2022-09-15T09:18:01.563885Z 6 [Warning] [MY-010453] [Server] root@localhost is created with an empty password ! Please consider switching off the --initialize-insecure option.
2022-09-15 09:18:04+00:00 [Note] [Entrypoint]: Database files initialized
2022-09-15 09:18:04+00:00 [Note] [Entrypoint]: Starting temporary server
2022-09-15T09:18:04.509741Z 0 [Warning] [MY-011068] [Server] The syntax '--skip-host-cache' is deprecated and will be removed in a future release. Please use SET GLOBAL host_cache_size=0 instead.
2022-09-15T09:18:04.512026Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.30) starting as process 125
...


```

* docker top 容器id

```shell
docker top 容器id  # 查看容器中进程信息
```

* docker inspect

```shell
docker inspect 容器id/容器名  查看容器元数据
[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker inspect mysql
[
    {
        "Id": "1b6c8911355870f43487eb85db4f4b2d6421bed6d257a71e079798bb97dd8646",
        "Created": "2022-09-15T09:17:57.95142708Z",
        "Path": "docker-entrypoint.sh",
        "Args": [
            "mysqld"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 1899611,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2022-11-28T06:36:37.083409742Z",
            "FinishedAt": "2022-11-28T06:29:53.12265741Z"
        },
        "Image": "sha256:ff3b5098b416cc4294d8d5c43c2f0f8251e91711347318e73cb290ffe2783bcb",
        "ResolvConfPath": "/var/lib/docker/containers/1b6c8911355870f43487eb85db4f4b2d6421bed6d257a71e079798bb97dd8646/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/1b6c8911355870f43487eb85db4f4b2d6421bed6d257a71e079798bb97dd8646/hostname",
        "HostsPath": "/var/lib/docker/containers/1b6c8911355870f43487eb85db4f4b2d6421bed6d257a71e079798bb97dd8646/hosts",
        "LogPath": "/var/lib/docker/containers/1b6c8911355870f43487eb85db4f4b2d6421bed6d257a71e079798bb97dd8646/1b6c8911355870f43487eb85db4f4b2d6421bed6d257a71e079798bb97dd8646-json.log",
        "Name": "/mysql",
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
            "PortBindings": {
                "3306/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "3307"
                    }
                ]
            },
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "host",
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
                "LowerDir": "/var/lib/docker/overlay2/91d3deba8a700cc7d7bc5565c4dfad48bf6ab6bfb65b4f8ddcc18f84f3e66c28-init/diff:/var/lib/docker/overlay2/84c6f250503ccc216ccd087ab24e0376e00c84040079427e66b589178030fbe4/diff:/var/lib/docker/overlay2/de1b5e419fd75126d4cf9b3fb0f5f448d4350976fc85126bdc8b76baa6934ebf/diff:/var/lib/docker/overlay2/53d948f7d13ae78d3e0b73e4dfef26beac04fdb6afb2d44564f733c486abce8d/diff:/var/lib/docker/overlay2/53b25bc12f7e3201bfd2763d4562346223f0fb3f45ee3c628149135029c2788d/diff:/var/lib/docker/overlay2/42672c099f7dcd6ab3671ad532de59508be376e8ac4b23b64ffbba8b91c54e37/diff:/var/lib/docker/overlay2/4deb7132c15dde066d697ce7837d14bb0a80b5e36ad53e1620d905306bc5e895/diff:/var/lib/docker/overlay2/9cfa455898783ddbccb67b93b4627570a3cc007a5c6d0d02c818060de5846470/diff:/var/lib/docker/overlay2/145b2258730c4cf8c0c649c1a666d289382519e6ad89612c462979fdd4d45373/diff:/var/lib/docker/overlay2/a5236ba1820cfedb5de717f72277eb3ae597bb312225439ad821b9d049c5172f/diff:/var/lib/docker/overlay2/d48af4744f69821d07822f97fcee9ad291082df99d88e71521a5850a6ac84280/diff:/var/lib/docker/overlay2/201a0a0a31dd18c8c2a7f0f2d9e4b0cb5ab94aeb92ec3c21041eb6ec13095674/diff",
                "MergedDir": "/var/lib/docker/overlay2/91d3deba8a700cc7d7bc5565c4dfad48bf6ab6bfb65b4f8ddcc18f84f3e66c28/merged",
                "UpperDir": "/var/lib/docker/overlay2/91d3deba8a700cc7d7bc5565c4dfad48bf6ab6bfb65b4f8ddcc18f84f3e66c28/diff",
                "WorkDir": "/var/lib/docker/overlay2/91d3deba8a700cc7d7bc5565c4dfad48bf6ab6bfb65b4f8ddcc18f84f3e66c28/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [
            {
                "Type": "volume",
                "Name": "1e76895438bb3fed4d7d813e2319f8d0abef464acb4890f423f9a248f56ce702",
                "Source": "/var/lib/docker/volumes/1e76895438bb3fed4d7d813e2319f8d0abef464acb4890f423f9a248f56ce702/_data",
                "Destination": "/var/lib/mysql",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
        ],
        "Config": {
            "Hostname": "1b6c89113558",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "3306/tcp": {},
                "33060/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "MYSQL_ROOT_PASSWORD=xxx",
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "GOSU_VERSION=1.14",
                "MYSQL_MAJOR=8.0",
                "MYSQL_VERSION=8.0.30-1.el8",
                "MYSQL_SHELL_VERSION=8.0.30-1.el8"
            ],
            "Cmd": [
                "mysqld"
            ],
            "Image": "mysql:latest",
            "Volumes": {
                "/var/lib/mysql": {}
            },
            "WorkingDir": "",
            "Entrypoint": [
                "docker-entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {}
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "cb6885d469116f82982b1688b0c55d1df5616c331753a32023fc9a92f3e4e6d7",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {
                "3306/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "3307"
                    },
                    {
                        "HostIp": "::",
                        "HostPort": "3307"
                    }
                ],
                "33060/tcp": null
            },
            "SandboxKey": "/var/run/docker/netns/cb6885d46911",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "2d3143b9236633cf50ebd4bd4e70904daed055d297517a5551aa348c781102a6",
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
                    "NetworkID": "dd10917ed190de5e85dfa700607287e01529269f92334a442d91c05ff9060041",
                    "EndpointID": "2d3143b9236633cf50ebd4bd4e70904daed055d297517a5551aa348c781102a6",
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

* 进入当前正在运行的容器

```shell
docker exec -it 容器id /bin/bash  # 进入容器后开启一个新的终端
docker attach 容器id   # 进入容器正在执行的终端
```

* 从容器内拷贝文件到主机

```shell
docker cp 容器id:容器内路径  目的主机路径
```

## 安装nginx

```shell
docker pull nginx
[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
a2abf6c4d29d: Pull complete 
a9edb18cadd1: Pull complete 
589b7251471a: Pull complete 
186b1aaa4aa6: Pull complete 
b4df32aa5a72: Pull complete 
a0bcbecc962e: Pull complete 
Digest: sha256:0d17b565c37bcbd895e9d92315a05c1c3c9a29f762b011a10c54a66cd53c9b31
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker images
REPOSITORY   TAG              IMAGE ID       CREATED         SIZE
ubuntu       20.04            a0ce5a295b63   3 months ago    72.8MB
mysql        latest           ff3b5098b416   3 months ago    447MB
nginx        latest           605c77e624dd   11 months ago   141MB
centos       centos7.9.2009   eeb6ee3f44bd   14 months ago   204MB
[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker run -d --name nginx01 -p 2333:80 nginx
b7ac691d77cbfc3a757de182bbb871116cf8fdc2991ec9eec7e36ad85b653df5
[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                                                  NAMES
b7ac691d77cb   nginx          "/docker-entrypoint.…"   4 seconds ago   Up 2 seconds   0.0.0.0:2333->80/tcp, :::2333->80/tcp                  nginx01
1b6c89113558   mysql:latest   "docker-entrypoint.s…"   2 months ago    Up 3 days      33060/tcp, 0.0.0.0:3307->3306/tcp, :::3307->3306/tcp   mysql
[root@iZ2zef4iumnunw6gqcbjxsZ ~]# curl localhost:2333
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker exec -it nginx01 /bin/bash
root@b7ac691d77cb:/# ls
bin  boot  dev	docker-entrypoint.d  docker-entrypoint.sh  etc	home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@b7ac691d77cb:/# whereis nginx
nginx: /usr/sbin/nginx /usr/lib/nginx /etc/nginx /usr/share/nginx
root@b7ac691d77cb:/# ls /etc/nginx/
conf.d	fastcgi_params	mime.types  modules  nginx.conf  scgi_params  uwsgi_params

```

## 安装Tomcat

```shell
[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker pull tomcat
Using default tag: latest
latest: Pulling from library/tomcat
0e29546d541c: Pull complete 
9b829c73b52b: Pull complete 
cb5b7ae36172: Pull complete 
6494e4811622: Pull complete 
668f6fcc5fa5: Pull complete 
dc120c3e0290: Pull complete 
8f7c0eebb7b1: Pull complete 
77b694f83996: Pull complete 
0f611256ec3a: Pull complete 
4f25def12f23: Pull complete 
Digest: sha256:9dee185c3b161cdfede1f5e35e8b56ebc9de88ed3a79526939701f3537a52324
Status: Downloaded newer image for tomcat:latest
docker.io/library/tomcat:latest
[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker run -d --name tomcat01 -p 8081:8080 tomcat
ca8fae9c1f57d8ac8d86ff74551b918b0fd69bb1e85328cd050571928a637072
[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                                                  NAMES
ca8fae9c1f57   tomcat         "catalina.sh run"        9 seconds ago    Up 7 seconds    0.0.0.0:8081->8080/tcp, :::8081->8080/tcp              tomcat01
b7ac691d77cb   nginx          "/docker-entrypoint.…"   19 minutes ago   Up 19 minutes   0.0.0.0:2333->80/tcp, :::2333->80/tcp                  nginx01
1b6c89113558   mysql:latest   "docker-entrypoint.s…"   2 months ago     Up 4 days       33060/tcp, 0.0.0.0:3307->3306/tcp, :::3307->3306/tcp   mysql
[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker logs tomcat01
NOTE: Picked up JDK_JAVA_OPTIONS:  --add-opens=java.base/java.lang=ALL-UNNAMED --add-opens=java.base/java.io=ALL-UNNAMED --add-opens=java.base/java.util=ALL-UNNAMED --add-opens=java.base/java.util.concurrent=ALL-UNNAMED --add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED
02-Dec-2022 06:20:01.143 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server version name:   Apache Tomcat/10.0.14
02-Dec-2022 06:20:01.202 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server built:          Dec 2 2021 22:01:36 UTC
02-Dec-2022 06:20:01.202 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server version number: 10.0.14.0
02-Dec-2022 06:20:01.202 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log OS Name:               Linux
02-Dec-2022 06:20:01.202 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log OS Version:            4.18.0-348.7.1.el8_5.x86_64
02-Dec-2022 06:20:01.202 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Architecture:          amd64
02-Dec-2022 06:20:01.203 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Java Home:             /usr/local/openjdk-11
02-Dec-2022 06:20:01.203 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log JVM Version:           11.0.13+8
02-Dec-2022 06:20:01.203 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log JVM Vendor:            Oracle Corporation
02-Dec-2022 06:20:01.203 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log CATALINA_BASE:         /usr/local/tomcat
02-Dec-2022 06:20:01.203 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log CATALINA_HOME:         /usr/local/tomcat
02-Dec-2022 06:20:01.275 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: --add-opens=java.base/java.lang=ALL-UNNAMED
02-Dec-2022 06:20:01.281 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: --add-opens=java.base/java.io=ALL-UNNAMED
02-Dec-2022 06:20:01.281 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: --add-opens=java.base/java.util=ALL-UNNAMED
02-Dec-2022 06:20:01.281 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: --add-opens=java.base/java.util.concurrent=ALL-UNNAMED
02-Dec-2022 06:20:01.281 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: --add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED
02-Dec-2022 06:20:01.282 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.util.logging.config.file=/usr/local/tomcat/conf/logging.properties
02-Dec-2022 06:20:01.282 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager
02-Dec-2022 06:20:01.282 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djdk.tls.ephemeralDHKeySize=2048
02-Dec-2022 06:20:01.282 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.protocol.handler.pkgs=org.apache.catalina.webresources
02-Dec-2022 06:20:01.282 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dorg.apache.catalina.security.SecurityListener.UMASK=0027
02-Dec-2022 06:20:01.282 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dignore.endorsed.dirs=
02-Dec-2022 06:20:01.282 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dcatalina.base=/usr/local/tomcat
02-Dec-2022 06:20:01.283 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dcatalina.home=/usr/local/tomcat
02-Dec-2022 06:20:01.283 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.io.tmpdir=/usr/local/tomcat/temp
02-Dec-2022 06:20:01.330 INFO [main] org.apache.catalina.core.AprLifecycleListener.lifecycleEvent Loaded Apache Tomcat Native library [1.2.31] using APR version [1.7.0].
02-Dec-2022 06:20:01.332 INFO [main] org.apache.catalina.core.AprLifecycleListener.lifecycleEvent APR capabilities: IPv6 [true], sendfile [true], accept filters [false], random [true], UDS [true].
02-Dec-2022 06:20:01.340 INFO [main] org.apache.catalina.core.AprLifecycleListener.initializeSSL OpenSSL successfully initialized [OpenSSL 1.1.1k  25 Mar 2021]
02-Dec-2022 06:20:02.575 INFO [main] org.apache.coyote.AbstractProtocol.init Initializing ProtocolHandler ["http-nio-8080"]
02-Dec-2022 06:20:02.741 INFO [main] org.apache.catalina.startup.Catalina.load Server initialization in [2487] milliseconds
02-Dec-2022 06:20:02.968 INFO [main] org.apache.catalina.core.StandardService.startInternal Starting service [Catalina]
02-Dec-2022 06:20:02.972 INFO [main] org.apache.catalina.core.StandardEngine.startInternal Starting Servlet engine: [Apache Tomcat/10.0.14]
02-Dec-2022 06:20:03.004 INFO [main] org.apache.coyote.AbstractProtocol.start Starting ProtocolHandler ["http-nio-8080"]
02-Dec-2022 06:20:03.047 INFO [main] org.apache.catalina.startup.Catalina.start Server startup in [305] milliseconds
[root@iZ2zef4iumnunw6gqcbjxsZ ~]# docker exec -it tomcat01 /bin/bash
root@ca8fae9c1f57:/usr/local/tomcat# ls
BUILDING.txt	 LICENSE  README.md	 RUNNING.txt  conf  logs	    temp     webapps.dist
CONTRIBUTING.md  NOTICE   RELEASE-NOTES  bin	      lib   native-jni-lib  webapps  work
```