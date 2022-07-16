<h3>### Docker ###</h3>

<h4>Описание домашнего задания</h4>

<p>Создайте свой кастомный образ nginx на базе alpine. После запуска nginx должен отдавать кастомную страницу (достаточно изменить дефолтную страницу nginx)</p>
<p>Определите разницу между контейнером и образом</p>
<p>Вывод опишите в домашнем задании.</p>
<p>Ответьте на вопрос: Можно ли в контейнере собрать ядро?</p>
<p>Собранный образ необходимо запушить в docker hub и дать ссылку на ваш репозиторий.</p>

<h4>Установка Docker</h4>

<p>Установим пакет yum-utils:</p>

<pre>[user@localhost ~]$ sudo yum install yum-utils -y
[sudo] password for user: 
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: centos-mirror.rbc.ru
 * epel: mirror.logol.ru
 * extras: centos-mirror.rbc.ru
 * rpmfusion-free-updates: mirror.yandex.ru
 * updates: centos-mirror.rbc.ru
https://rpm.releases.hashicorp.com/RHEL/7/x86_64/stable/repodata/repomd.xml: [Errno 14] HTTPS Error 403 - Forbidden
Trying other mirror.
To address this issue please refer to the below wiki article

https://wiki.centos.org/yum-errors

If above article doesn't help to resolve this issue please use https://bugs.centos.org/.

Package yum-utils-1.1.31-54.el7_8.noarch already installed and latest version
Nothing to do
[user@localhost ~]$</pre>

<p>Как видим, пакет yum-utils уже установлен.</p>

<p>Добавим официальный репозиторий Docker в систему:</p>

<pre>[user@localhost ~]$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
Loaded plugins: fastestmirror, langpacks
adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
grabbing file https://download.docker.com/linux/centos/docker-ce.repo to /etc/yum.repos.d/docker-ce.repo
repo saved to /etc/yum.repos.d/docker-ce.repo
[user@localhost ~]$</pre>

<p>Установим Docker:</p>

<pre>[user@localhost ~]$ sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y
...
Installed:
  containerd.io.x86_64 0:1.6.6-3.1.el7                                          
  docker-ce.x86_64 3:20.10.17-3.el7                                             
  docker-ce-cli.x86_64 1:20.10.17-3.el7                                         
  docker-compose-plugin.x86_64 0:2.6.0-3.el7                                    

Dependency Installed:
  container-selinux.noarch 2:2.119.2-1.911c772.el7_8                            
  docker-ce-rootless-extras.x86_64 0:20.10.17-3.el7                             
  docker-scan-plugin.x86_64 0:0.17.0-3.el7                                      
  fuse-overlayfs.x86_64 0:0.7.2-6.el7_8                                         
  fuse3-libs.x86_64 0:3.6.1-4.el7                                               
  slirp4netns.x86_64 0:0.4.3-4.el7_8                                            

Complete!
[user@localhost ~]$</pre>

<p>В домашней директории создадим каталог docker:</p>

<pre>[user@localhost otus]$ mkdir ./docker
[user@localhost otus]$</pre>

<p>Перейдём в этот каталог:</p>

<pre>[user@localhost otus]$ cd ./docker/
[user@localhost docker]$</pre>

<p>В этом каталоге создадим файл Dockerfile:</p>

<pre>[user@localhost ansible]$ vi ./Dockerfile</pre>

<p>Заполним его следующим содержимым:</p>

<pre>FROM alpine
LABEL mainteiner="SergSha"
RUN apk add --update --no-cache nginx && mkdir -p /run/nginx
COPY index.html /var/lib/nginx/html/index.html
COPY default.conf /etc/nginx/http.d/default.conf
EXPOSE 80
CMD ["nginx","-g","daemon off;"]</pre>

<p>Создадим nginx конфиг файл:</p>

<pre>[user@localhost ansible]$ vi ./default.conf</pre>

<pre># /etc/nginx/http.d/myconf.conf

server {
	listen 80 default_server;
	root /var/lib/nginx/html/;
	index index.html;
}</pre>

<p>Создадим файл веб-страницы index.html:</p>

<pre>[user@localhost ansible]$ vi ./index.html</pre>

<pre><!DOCTYPE html>
<html>
<head>
<title>My Web Page</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to my web page!</h1>
<p>This my web page from alpine-nginx docker.</p>
</body>
</html></pre>

<p>Создаём образ с названием alpinx (сокращенное ALPine-ngINX):</p>

<pre>[user@localhost docker]$ docker build -t alpinx -f ./Dockerfile .
Sending build context to Docker daemon  4.096kB
Step 1/7 : FROM alpine
latest: Pulling from library/alpine
2408cc74d12b: Pull complete 
Digest: sha256:686d8c9dfa6f3ccfc8230bc3178d23f84eeaf7e457f36f271ab1acc53015037c
Status: Downloaded newer image for alpine:latest
 ---> e66264b98777
Step 2/7 : LABEL mainteiner="SergSha"
 ---> Running in 26eeb5246930
Removing intermediate container 26eeb5246930
 ---> fa2f778cecda
Step 3/7 : RUN apk add --update --no-cache nginx && mkdir -p /run/nginx
 ---> Running in ba7e2d50906d
fetch https://dl-cdn.alpinelinux.org/alpine/v3.16/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.16/community/x86_64/APKINDEX.tar.gz
(1/2) Installing pcre (8.45-r2)
(2/2) Installing nginx (1.22.0-r1)
Executing nginx-1.22.0-r1.pre-install
Executing nginx-1.22.0-r1.post-install
Executing busybox-1.35.0-r13.trigger
OK: 7 MiB in 16 packages
Removing intermediate container ba7e2d50906d
 ---> 0e5c78b9cf2f
Step 4/7 : COPY index.html /var/lib/nginx/html/index.html
 ---> cd709458ae82
Step 5/7 : COPY default.conf /etc/nginx/http.d/default.conf
 ---> 78626db2342d
Step 6/7 : EXPOSE 80
 ---> Running in 55897e2b8488
Removing intermediate container 55897e2b8488
 ---> 5f7f4d085bcd
Step 7/7 : CMD ["nginx","-g","daemon off;"]
 ---> Running in 6274c9fdbc22
Removing intermediate container 6274c9fdbc22
 ---> fe3f7a490895
Successfully built fe3f7a490895
Successfully tagged alpinx:latest
[user@localhost docker]$</pre>

<p>Смотрим, создался ли наш образ alpinx:</p>

<pre>[user@localhost docker]$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED              SIZE
alpinx       latest    fe3f7a490895   About a minute ago   6.98MB
alpine       latest    e66264b98777   7 weeks ago          5.53MB
[user@localhost docker]$</pre>

<p>Как видим, наш образ alpinx (image id fe3f7a490895) создался на базе скачанного с dockerhub образа alpine.</p>

<p>С нашего образа alpinx запустим контейнер с именем "alpine-nginx" в фоновом режиме (-d), с последующим удалением после остановки (--rm), с внешним портом 80:</p>

<pre>[user@localhost docker]$ docker run --name alpine-nginx --rm -d -p 80:80 alpinx
5da5bd9a825f1f9f0e8ce426c5e79c24477315039cf6fa07cc92dafd3673c92b
[user@localhost docker]$</pre>

<p>Смотрим, запустился ли наш контейнер alpine-nginx:</p>

<pre>[user@localhost docker]$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                               NAMES
5da5bd9a825f   alpinx    "nginx -g 'daemon of…"   3 minutes ago   Up 3 minutes   0.0.0.0:80->80/tcp, :::80->80/tcp   alpine-nginx
[user@localhost docker]$</pre>

<p>Наблюдаем, что наш контейнер alpine-nginx (container id 5da5bd9a825f) запущен.</p>

<p>Информация по запущенному контейнеру:</p>

<pre>[user@localhost docker]$ docker inspect alpine-nginx
[
    {
        "Id": "5da5bd9a825f1f9f0e8ce426c5e79c24477315039cf6fa07cc92dafd3673c92b",
        "Created": "2022-07-16T08:56:15.845932406Z",
        "Path": "nginx",
        "Args": [
            "-g",
            "daemon off;"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 10174,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2022-07-16T08:56:16.254321262Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:fe3f7a4908958ac12881ac81711f4ff7a8fc15406b234207feeeacb952da76b3",
        "ResolvConfPath": "/var/lib/docker/containers/5da5bd9a825f1f9f0e8ce426c5e79c24477315039cf6fa07cc92dafd3673c92b/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/5da5bd9a825f1f9f0e8ce426c5e79c24477315039cf6fa07cc92dafd3673c92b/hostname",
        "HostsPath": "/var/lib/docker/containers/5da5bd9a825f1f9f0e8ce426c5e79c24477315039cf6fa07cc92dafd3673c92b/hosts",
        "LogPath": "/var/lib/docker/containers/5da5bd9a825f1f9f0e8ce426c5e79c24477315039cf6fa07cc92dafd3673c92b/5da5bd9a825f1f9f0e8ce426c5e79c24477315039cf6fa07cc92dafd3673c92b-json.log",
        "Name": "/alpine-nginx",
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
                "80/tcp": [
                    {
                        "HostIp": "",
                        "HostPort": "80"
                    }
                ]
            },
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": true,
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
                "LowerDir": "/var/lib/docker/overlay2/f69a0085b839ea14cbb21fd7c4e88f4787ced3a9986ffa0eff031bfab3611fd1-init/diff:/var/lib/docker/overlay2/91a9e3aa222806b45b664bab4e968f2fbc6c2360d333f81fff96966a006791cd/diff:/var/lib/docker/overlay2/13a1e6c25d8f28627262edc4418d8bf94eb4299a64547e8869b26898a9fe9f9b/diff:/var/lib/docker/overlay2/9691e81c1006673a38dafc39e82a552fec3034496f734f8f4c40bd2f9554e073/diff:/var/lib/docker/overlay2/9248f31490cfc382c56cbf7c943527e81497f110674ec25810f168d78d5fd824/diff",
                "MergedDir": "/var/lib/docker/overlay2/f69a0085b839ea14cbb21fd7c4e88f4787ced3a9986ffa0eff031bfab3611fd1/merged",
                "UpperDir": "/var/lib/docker/overlay2/f69a0085b839ea14cbb21fd7c4e88f4787ced3a9986ffa0eff031bfab3611fd1/diff",
                "WorkDir": "/var/lib/docker/overlay2/f69a0085b839ea14cbb21fd7c4e88f4787ced3a9986ffa0eff031bfab3611fd1/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "5da5bd9a825f",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "nginx",
                "-g",
                "daemon off;"
            ],
            "Image": "alpinx",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "mainteiner": "SergSha"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "417b93b467f5889190d33434a674c808dadd7cebc9ee3b87a10cdd1519c24353",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {
                "80/tcp": [
                    {
                        "HostIp": "0.0.0.0",
                        "HostPort": "80"
                    },
                    {
                        "HostIp": "::",
                        "HostPort": "80"
                    }
                ]
            },
            "SandboxKey": "/var/run/docker/netns/417b93b467f5",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "dbdf1d979b2e16ae80ef6c23e62bffb741f1daa985dfcd8ff6c82cb571600336",
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
                    "NetworkID": "8af783a4164961a0b7c2474303000346e65fd1afdedeec9034c7c25cf1b7e130",
                    "EndpointID": "dbdf1d979b2e16ae80ef6c23e62bffb741f1daa985dfcd8ff6c82cb571600336",
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
[user@localhost docker]$</pre>

<p>Вывод логов нашего контейнера alpine-nginx:</p>

<pre>[user@localhost docker]$ docker logs --details alpine-nginx
[user@localhost docker]$</pre>

<p>В адресной строке браузера введём 127.0.0.1 и смотрим результат работы нашего контейнера alpine-nginx, запущенного на базе нашего созданного образа alpinx:</p>

![image](https://user-images.githubusercontent.com/96518320/179348812-10ceb42e-3ae3-4e07-a579-8a453deaaf1e.png)



