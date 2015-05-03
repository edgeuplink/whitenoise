The second article on the Docker Series.

[![dockerlogo.png](https://d23f6h5jpj26xu.cloudfront.net/shxackvgfnxva_small.png)](http://img.svbtle.com/shxackvgfnxva.png)

Docker is both a company and a platform. At its core Docker is a container runtime. It brings together all what we previously talked (Namespaces, cgroups,Capabilities, ...) into a product.

It's a packaged shipping implementation you can get support for. It provides a uniform and standard runtime, meaning developers can code apps in Docker containers on their laptops and lift them and drop them to whatever environment running the docker engine runtime.

[![engine.png](https://d23f6h5jpj26xu.cloudfront.net/cgjcabojugxxw_small.png)](http://img.svbtle.com/cgjcabojugxxw.png)

> Nice & Cool application **portability**

But as a platform Docker includes a Registry (Docker Hub), Clustering (Docker Swarm), Orchestration (Docker Compose), ... It started as an inside project of DotCloud by Solomon Hykes, it's crafted in GoLang and released under the Apache 2 license.

We'll focus on the **Docker technology** (client and daemon) and not on the platform. By the time you are reading this article Docker might be already on Windows not meaning that continuing to follow the series is obsolete material because we will focus on the Linux implementation.

> But what about **LXC** ?

In the early days Docker Engine relied on **LXC** to interface the kernel features. But that approach had a few annoyances to Docker developers.

- How to make sure the version of LXC was up to date and worked with Docker ?
- Relying on something that is out of your control but is a core component of your solution is a huge problem.

So Docker decided to write and open source a new execution driver called **libcontainer** as a drop-in replacement to LXC (as far as Docker concerns) still giving Docker direct access to the kernel features and also allowing them to go cross-platform. Shipping libcontainer inside Docker Engine is massive for its ecosystem.

> Before you continue pay attention to this

Even though you can code an app inside a container and ship it wherever you please you can't run a Windows App Container on top of Docker Engine on top of Linux. And of course the other way around.

<hr></hr>

In the future we should *lose the fat* of running Docker Engine inside of virtual machines (like AWS EC2) and spin them right up Linux or Windows over the Physical Machines. As Intel and AMD came up with chip-level assists (VT-X,AMD-V) to virtual machines once there was enough market demand same would happen for Containers.

[![docker_vm.jpg](https://d23f6h5jpj26xu.cloudfront.net/ucganwzicnqxwa_small.jpg)](http://img.svbtle.com/ucganwzicnqxwa.jpg)

At **application design** level you are encouraged to use the **Microservices** approach building it as a group of smaller components each in its container interacting with each other forming the overall app. Each component is individual upgradeable and independent of all the other components.

<hr></hr>

> # Time to play around with the Docker Engine

We are going to play with the Docker **client** and **daemon** in a client-server model.

- Client send commands (requests)
- Daemon does the hard work of creating containers constructing all the kernel namespace, cgroups, capabilities needed to instantiate them.

[![dengine.png](https://d23f6h5jpj26xu.cloudfront.net/q58xgz8o0dzgyg_small.png)](http://img.svbtle.com/q58xgz8o0dzgyg.png)

Okay power on your 2 virtual machines and let's install different versions of Docker in each vm, then we'll connect Docker client from **vm-1** to Docker daemon listening on a network port on **vm-2**

> Pick up your ubuntu vm-1 and tackle it with the latest version

```bash

# sudo su
# wget -qO- https://get.docker.com/ | sh

/* Add your user to docker group */

# sudo usermod -aG docker YOUR.USER

/* Log off and back on */

# docker version
Client version: 1.6.0
Client API version: 1.18
Go version (client): go1.4.2
Git commit (client): 4749651
OS/Arch (client): linux/amd64
Server version: 1.6.0
Server API version: 1.18
Go version (server): go1.4.2
Git commit (server): 4749651
OS/Arch (server): linux/amd64

# docker info
Containers: 0
Images: 0
Storage Driver: aufs
 Root Dir: /var/lib/docker/aufs
 Backing Filesystem: extfs
 Dirs: 0
 Dirperm1 Supported: false
Execution Driver: native-0.2
Kernel Version: 3.13.0-24-generic
Operating System: Ubuntu 14.04.2 LTS
CPUs: 4
Total Memory: 1.955 GiB
Name: dock01
ID: H7QF:LKVV:HZIL:FNFF:HSKP:EFJF:SUSR:ME5C:WIHF:VD4Q:SWAQ:BSDZ
WARNING: No swap limit support


```

Seems good and up to date let's spin up our first container :

```bash

# docker run -it ubuntu /bin/bash
Unable to find image 'ubuntu:latest' locally
latest: Pulling from ubuntu
511136ea3c5a: Pull complete
f3c84ac3a053: Pull complete
a1a958a24818: Pull complete
9fec74352904: Pull complete
d0955f21bf24: Already exists
Digest: sha256:7b27f7cc97d4c94fdebb2cf99ddaadd0e8fc8ec4aed2cf56a8ee8fe7dc2ec4a4
Status: Downloaded newer image for ubuntu:latest
root@07e6050b6ec1:/#

```

And we're inside our first container (07e6050b6ec1 is its unique ID) with root privileges. You can type exit and return to your prompt because we're going to bite the Docker daemon.

At vm-1 make sure the daemon is not listening in any network port

```bash

# netstat -tlp
Active Internet connections (only servers)
Proto ... Local Address   Foreign Address    State    PID/name
tcp     ...    *:ssh         *:*           LISTEN       937/sshd
tcp6   ...    [::]:ssh      [::]:*         LISTEN       937/sshd

/* We're cool here , now stop the service */

# service docker stop
docker stop/waiting

/* Reconfigure it with your ip address listening at port 2375 */

# docker -H 172.20.122.115:2375 -d & 
[1] 20877
root@dock01:~# INFO[0000] +job init_networkdriver()
INFO[0000] +job serveapi(tcp://172.20.122.115:2375)
INFO[0000] Listening for HTTP on tcp (172.20.122.115:2375)
INFO[0000] -job init_networkdriver() = OK (0)
WARN[0000] Your kernel does not support cgroup swap limit.
INFO[0000] Loading containers: start.
..
INFO[0000] Loading containers: done.
INFO[0000] docker daemon: 1.6.0 4749651; execdriver: native-0.2; graphdriver: aufs
INFO[0000] +job acceptconnections()
INFO[0000] -job acceptconnections() = OK (0)
INFO[0000] Daemon has completed initialization

```

Recheck the listening ports and try running **docker info** command

```bash

# netstat -tlp
Active Internet connections (only servers)
Proto Local Address Foreign Address State PID/Program name
tcp     *:ssh               *:*      LISTEN       937/sshd
tcp     172.20.122.115:2375 *:*      LISTEN   20877/docker
tcp6    [::]:ssh            [::]:*   LISTEN       937/sshd

# docker info
FATA[0000] Get http:///var/run/docker.sock/v1.18/info: dial unix /var/run/docker.sock: no such file or directory.


```

Don't panic, Docker client is trying to connect to the previous local unix socket used by the daemon. We're listening on the network now.

[![what_is_docker.png](https://d23f6h5jpj26xu.cloudfront.net/yt9yftaw5c4sgg_small.png)](http://img.svbtle.com/yt9yftaw5c4sgg.png)

Jump to vm-2 and configure the client to connect to the Docker daemon on vm-1 that we just configured.

```bash

# export DOCKER_HOST="tcp://172.20.122.115:2375"
# docker info
Containers: 0
Images: 0
Storage Driver: aufs
 Root Dir: /var/lib/docker/aufs
 Backing Filesystem: extfs
 Dirs: 0
 Dirperm1 Supported: false
Execution Driver: native-0.2
Kernel Version: 3.13.0-24-generic
Operating System: Ubuntu 14.04.2 LTS
CPUs: 4
Total Memory: 1.955 GiB
Name: dock01 /* LOOK HERE */
ID: H7QF:LKVV:HZIL:FNFF:HSKP:EFJF:SUSR:ME5C:WIHF:VD4Q:SWAQ:BSDZ
WARNING: No swap limit support


```

And you're getting the *info* from the Docker daemon on vm-1... Great!

[![servclient.png](https://d23f6h5jpj26xu.cloudfront.net/ygptczah0wuqca_small.png)](http://img.svbtle.com/ygptczah0wuqca.png)

*As a side note you can bind the daemon both to an unix socket and to a tcp port for example.*

> **Now we get back to playing with the container**

Start to spin a CentOS container on top of your ubuntu vm-1

```bash

# docker run -it centos /bin/bash
Unable to find image 'centos:latest' locally
latest: Pulling from centos
b6718650e87e: Pull complete
3d3c8202a574: Pull complete
0114405f9ff1: Already exists
511136ea3c5a: Already exists
centos:latest: The image you are pulling has been verified. Important: image verification is a tech preview feature and should not be relied on to provide security.
Digest: sha256:9dcbb28b39607cdf924f22c22e5260c91fb778cee4d3d92407ac47bbb640349d
Status: Downloaded newer image for centos:latest
[root@fef492813149 /]#

/* inside CentOS container check this */
[root@fef492813149 /]# cat /etc/hosts
172.17.0.1	fef492813149
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
/* the hostname matches the prompt */

/* you can do trivial stuff with the system */
[root@fef492813149 /]# ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=56 time=7.16 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=56 time=5.84 ms
^C
--- 8.8.8.8 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 5.845/6.505/7.165/0.660 ms


```

Huuummm sounds like it's really a linux machine (because it is), but we have a CentOS container running on an Ubuntu host. **Check this out :**

```bash


[root@fef492813149 /]# uname -a
Linux fef492813149 3.13.0-24-generic #47-Ubuntu SMP Fri May 2 23:30:00 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux

[root@fef492813149 /]# cat /etc/redhat-release
CentOS Linux release 7.1.1503 (Core)

[root@fef492813149 /]# apt-get update
bash: apt-get: command not found

[root@fef492813149 /]# yum check-update
Loaded plugins: fastestmirror
base                                                                                                                                     | 3.6 kB  00:00:00
extras                                                                                                                                   | 3.4 kB  00:00:00
systemdcontainer                                                                                                                         | 1.9 kB  00:00:00
updates                                                                                                                                  | 3.4 kB  00:00:00
(1/4): updates/7/x86_64/primary_db                                                                                                       | 957 kB  00:00:00
(2/4): extras/7/x86_64/primary_db                                                                                                        |  41 kB  00:00:00
(3/4): base/7/x86_64/group_gz                                                                                                            | 154 kB  00:00:00
(4/4): base/7/x86_64/primary_db                                                                                                          | 5.1 MB  00:00:00
systemdcontainer/primary_db                                                                                                              |  20 kB  00:00:00
Determining fastest mirrors
 * base: ftp.dei.uc.pt
 * extras: ftp.dei.uc.pt
 * updates: ftp.dei.uc.pt

tzdata.noarch                                                                2015c-1.el7                                                                 updates


```

> **Definately a CentOS box just sharing the kernel of the Ubuntu host!**

Inside your brand new **ultra lightweight** CentOS container install vim, write something on **/tmp/myfile** and exit the container.

Confirm that the container is not running **# docker ps** and *write down* **your** CONTAINER ID **# docker ps -a**

Before spinning the container back up and verify that the file is still there let's peek on something. Our **CONTAINER ID** was *fef492813149*.

```bash

# ls -l /var/lib/docker/aufs/diff/
total 48
drwxr-xr-x  2 root root 4096 Apr 21 00:27 0114405f9ff12fb7b012d0f7eb2f958c6ab8638061f67bc0a12cfc308ed31ee4
drwxr-xr-x  5 root root 4096 Apr 20 23:22 07e6050b6ec10e75ed149086829bec3f83de2aa577a478cd7c2808ce3e9e73e6
drwxr-xr-x  6 root root 4096 Apr 20 20:56 07e6050b6ec10e75ed149086829bec3f83de2aa577a478cd7c2808ce3e9e73e6-init
drwxr-xr-x 17 root root 4096 Apr 21 00:27 3d3c8202a57465ab6a24852559d21ca72a4af0243ada9a8799ccefb56f4d8a3f
drwxr-xr-x  2 root root 4096 Apr 20 20:56 511136ea3c5a64f264b78b5433614aec563103b4d4702f3ba7d4d2698e22c158
drwxr-xr-x  3 root root 4096 Apr 20 20:56 9fec74352904baf5ab5237caa39a84b0af5c593dc7cc08839e2ba65193024507
drwxr-xr-x  6 root root 4096 Apr 20 20:56 a1a958a248181c9aa6413848cd67646e5afb9797f1a3da5995c7a636f050f537
drwxr-xr-x  2 root root 4096 Apr 21 00:27 b6718650e87e3706c52682c87ecfd7a7e1fc176c9095d73d627ca41b2584839b
drwxr-xr-x  2 root root 4096 Apr 20 20:56 d0955f21bf24f5bfffd32d2d0bb669d0564701c271bc3dfc64cfc5adfdec2d07
drwxr-xr-x 21 root root 4096 Apr 20 20:56 f3c84ac3a0533f691c9fea4cc2ceaaf43baec22bf8d6a479e069f6d814be9b86
drwxr-xr-x 10 root root 4096 Apr 21 00:53 fef492813149b711805e8fbf2792a72b9506e02d457aa1f213974e1151872fec
drwxr-xr-x  6 root root 4096 Apr 21 00:27 fef492813149b711805e8fbf2792a72b9506e02d457aa1f213974e1151872fec-init


/* Find your CONTAINER ID in the first part of the UID on the list or just stick it to the end of your ls command */

# ls -l /var/lib/docker/aufs/diff/fef492813149b711805e8fbf2792a72b9506e02d457aa1f213974e1151872fec/
total 24
drwxr-xr-x 2 root root 4096 Apr 21 00:52 etc
dr-xr-x--- 2 root root 4096 Apr 21 00:53 root
drwxr-xr-x 2 root root 4096 Apr 21 00:52 run
drwxrwxrwt 2 root root 4096 Apr 21 00:53 tmp
drwxr-xr-x 4 root root 4096 Apr 21 00:52 usr
drwxr-xr-x 6 root root 4096 Apr 21 00:52 var


```

Cool, here's our container **tmp/** folder... myfile better be inside ! Just **cat** it.

```bash

# cat /var/lib/docker/aufs/diff/fef492813149b711805e8fbf2792a72b9506e02d457aa1f213974e1151872fec/tmp/myfile
Happy whales !

```

> Now spin that guy up **# docker start fef492813149** and jump on it **#docker attach fef492813149**


```bash


# docker start fef492813149
fef492813149

# docker attach fef492813149
[root@fef492813149 /]#
[root@fef492813149 /]# cat /tmp/myfile
Happy whales !


```

[![docker_wave_whale.png](https://d23f6h5jpj26xu.cloudfront.net/ew2o3o7llmgzxw_small.png)](http://img.svbtle.com/ew2o3o7llmgzxw.png)


Allright we made our point and are good for now. Need a recap ? Nah, if I'd write a recap nobody would pay attention to the article and just read this TLDR section ! But if you feel we are a bit naked without one send it via pull request. See you next week for some *ass whoopin* on major Docker components.

>## Follow the full series 

- <a href="http://up.svbtle.com/unload" target="_blank">Linux Containers</a>
- **Docker Engine** (this article)
- Docker Major Components (next article)

<hr></hr>

This article is part of White Noise Open Source Series meaning that the article can be updated / corrected / optimised by pull request at our public repositories.

> **☵☲ <a href="https://www.coinbase.com/edge" target=_blank>Buy me a beer</a> ☵☲** hkd up by Flávio HG
