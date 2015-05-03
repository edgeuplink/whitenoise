In the last article we got our hands on the Docker Engine, pretty basic. Today we're going to smuggle some Images & Containers and briefly talk about the Registry and Repositories.

**High level picture of each Component**

- Docker Engine : The Shipping Yard 
- Docker Images : Shipping Manifests
- Docker Containers : Shipping Containers

Before this streamlined flow of Containers, **taking an app to production took embarrassing amounts of time**.

[![linkedin.jpg](https://d23f6h5jpj26xu.cloudfront.net/kfptgcc8hthog_small.jpg)](http://img.svbtle.com/kfptgcc8hthog.jpg)


You code an app in a Docker container at your workstation or your laptop and you run it **unchanged** wherever your production environment happens to be. Unbelievable fast time to production !

> Security precautions should remain the same for unverified /untrusted code in production. Containers **can contain malicious code.**

Alright it's time to start peeking through the list of components.



<hr></hr>



# Engine in brief

This is our Docker program that we previously installed in the series. The shipping yard ! All the stuff that matters for application infrastructure and runtime dependencies. Things like root filesystems, network stacks, process hierarchies, variables, access to the kernel features, resource allocation, all the good things and **all standardised**. Every Docker install looks the same from one Docker host to another.

This standardisation brings AWESOME things, as long as the runtime environment provided by the container technology is the same then no matter what the underlined platform be that VMs on your workstation, bare metal on your datacenter, or instances in the cloud.. you can run your application with no changes necessary.

**So this is the Docker Engine :** A standardised runtime environment that looks and feels the same no matter what platform it's running on, and it makes application portability insanely trivial.

```bash

docker run -it fedora /bin/bash


```

Docker Images are what we launch containers from. They are like templates in a virtual machine world. If you dont specify the image version docker will pull the **:latest** version from the Docker Hub cloud.

At Docker Hub you'll find a complete set of repos, both officially maintained (like ubuntu,fedora, redis, nodejs, ...) and user repos.

Images contain all the data and metadata needed for firing up a container.
They are locally stored in your **/var/lib/docker/< storage-driver >/**

> If our images are the buildtime constructs then our containers are the runtime constructs. A Container is basically a running instance of an image.



[![horizon.jpg](https://d23f6h5jpj26xu.cloudfront.net/t2ntmwrpt7dja_small.jpg)](http://img.svbtle.com/t2ntmwrpt7dja.jpg)



<hr></hr>

> # Basics Out : **Deep Dive**

Docker Images are considered being layered (or stacked) ... huummm ... meaning a single Image might be formed by a group of stacked Images.

> Why ? **Modularity** with super thin layers !

[![layersimage.png](https://d23f6h5jpj26xu.cloudfront.net/bkeriyibjuxbya_small.png)](http://img.svbtle.com/bkeriyibjuxbya.png)

Each layer gets its own unique id, then these uuids are listed inside our Docker Image (purple) plus some metadata that tells Docker how to build the Container **at runtime**.

At runtime we have a single combined view of all layers. If there's a conflict (updated version of the same file at an upper layer) the higher layer wins.

> How ? **Union Mounts**.

The ability to mount multiple filesystems over the top of each other. All the layers on this image are mounted as **read only** (can be shared by many Containers) and the top layer (an additional layer that is added when we launch a Container) is the only **writable layer**. Meaning that when we edit a file from a read only layer in fact you are copying it to the Container R/W layer and saving the changes there.

When we start a container there is a small **bootfs** that exists below the rootfs and it's very short-lived (not there after the container started). You don't need to worry about this lightweight layer.


<hr></hr>


> # Image Layers

Look how it will download those multiple layers we just talked about when you pull an Image out of the Hub.

```bash


# docker pull coreos/apache
Pulling repository coreos/apache
87026dcb0044: Download complete
511136ea3c5a: Download complete
6170bb7b0ad1: Download complete
9cd978db300e: Download complete
Status: Downloaded newer image for coreos/apache:latest


# docker images
REPOSITORY          TAG            IMAGE ID           VIRTUAL SIZE
coreos/apache       latest         87026dcb0044        294.4 MB


# docker images --tree
└─511136ea3c5a Virtual Size: 0 B
  └─6170bb7b0ad1 Virtual Size: 0 B
    └─9cd978db300e Virtual Size: 204.4 MB
      └─87026dcb0044 Virtual Size: 294.4 MB Tags: coreos/apache:latest


```

Awesome, and if you feel like it you can inspect the layer contents at /var/lib/docker/aufs/layers/

> # Container-Fu

Generally for the Microservices model Containers **run a single process**. When a process inside a container exits so does the container. Let's ping Google just 30 times : 

```bash


$ docker run -d ubuntu /bin/bash -c "ping 8.8.8.8 -c 30"

$ docker ps
CONTAINER ID   ...  COMMAND            ...  STATUS 
b18531b66f8f   ...   /bin/bash -c ping    ... Up

$ docker top b18531b66f8f
PID                 USER                COMMAND
1657                root                ping 8.8.8.8 -c 30

$ docker ps
CONTAINER ID    ...       COMMAND        ...       STATUS
b18531b66f8f   ...       /bin/bash -c ping    ...    Up

$ docker ps
CONTAINER ID               COMMAND             STATUS

# Aaaand it's gone

$ docker ps -a
CONTAINER ID      COMMAND              STATUS
b18531b66f8f      /bin/bash -c ping     Exited (0) 7 minutes ago



```

That's how they work. We haven't been specific with our images using the **:latest** but you should be with your Images. Be explicit / ex: ubuntu:14.04 / so you always know what you're getting.


```bash

# 1024 would be all cpu shares

$ docker run --cpu-shares=256 ...

# By default all containers get equal access to shares

$ docker run memory=1g ...

# The man page is great on these parameters 


$ docker run -d ubuntu:14.04.1 /bin/bash -c "ping 8.8.8.8"
Unable to find image 'ubuntu:14.04.1' locally
Pulling repository ubuntu
5ba9dab47459: Download complete
511136ea3c5a: Download complete
27d47432a69b: Download complete
5f92234dcf1e: Download complete
51a9c7c1f8bb: Download complete
Status: Downloaded newer image for ubuntu:14.04.1
53a2277f2ba687c0ec6a9709f197b0b7fcd40d81a10cc837539ca899b69e11f0

# It's running detached

$ docker ps
CONTAINER ID     IMAGE            COMMAND          CREATED              
53a2277f2ba6     ubuntu:14.04.1   /bin/bash -c ping    3 minutes ago
STATUS                       NAMES
Up About a minute          adoring_lovelace

# Inspection time

$ docker inspect 53a2277f2ba6
[{
    "AppArmorProfile": "",
    "Args": [
        "-c",
        "ping 8.8.8.8"
    ],
    "Config": {
        "AttachStderr": false,
        "AttachStdin": false,
        "AttachStdout": false,
        "Cmd": [
            "/bin/bash",
            "-c",
            "ping 8.8.8.8"
        ],
        "CpuShares": 0,
        "Cpuset": "",
        "Domainname": "",
        "Entrypoint": null,
        "Env": [
            "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
        ],
        "ExposedPorts": null,
        "Hostname": "53a2277f2ba6",
        "Image": "ubuntu:14.04.1",
        "Labels": {},
        "MacAddress": "",
        "Memory": 0,
        "MemorySwap": 0,
        "NetworkDisabled": false,
        "OnBuild": null,
        "OpenStdin": false,
        "PortSpecs": null,
        "StdinOnce": false,
        "Tty": false,
        "User": "",
        "Volumes": null,
        "WorkingDir": ""
    },
    "Created": "2015-04-30T15:41:50.898484079Z",
    "Driver": "aufs",
    "ExecDriver": "native-0.2",
    "ExecIDs": null,
  ...
    "Name": "/adoring_lovelace",
    "NetworkSettings": {
        "Bridge": "docker0",
        "Gateway": "172.17.42.1",
        "GlobalIPv6Address": "",
        "GlobalIPv6PrefixLen": 0,
        "IPAddress": "172.17.0.2",
        "IPPrefixLen": 16,
        "IPv6Gateway": "",
        "LinkLocalIPv6Address": "fe80::42:acff:fe11:2",
        "LinkLocalIPv6PrefixLen": 64,
        "MacAddress": "02:42:ac:11:00:02",
        "PortMapping": null,
        "Ports": {}
    },
    "Path": "/bin/bash",
    "ProcessLabel": "",
...
    "RestartCount": 0,
    "State": {
        "Dead": false,
        "Error": "",
        "ExitCode": 0,
        "FinishedAt": "0001-01-01T00:00:00Z",
        "OOMKilled": false,
        "Paused": false,
        "Pid": 2045,
        "Restarting": false,
        "Running": true,
        "StartedAt": "2015-04-30T15:41:51.257467884Z"
    },
    "Volumes": {},
    "VolumesRW": {}
}
]

# Humm looks suspicious , let's go inside it


$ docker attach 53a2277f2ba6
# or 
$ docker attach adoring_lovelace
# how cute is our Container :)

64 bytes from 8.8.8.8: icmp_seq=490 ttl=61 time=18.3 ms
64 bytes from 8.8.8.8: icmp_seq=491 ttl=61 time=14.7 ms
64 bytes from 8.8.8.8: icmp_seq=492 ttl=61 time=15.2 ms
64 bytes from 8.8.8.8: icmp_seq=493 ttl=61 time=14.8 ms
64 bytes from 8.8.8.8: icmp_seq=494 ttl=61 time=13.1 ms
^C
--- 8.8.8.8 ping statistics ---
494 packets transmitted, 494 received, 0% packet loss, time 494294ms
rtt min/avg/max/mdev = 6.654/14.711/124.407/6.092 ms

$

# We got in , Ctrl-C the ping command and just quitted the Container by stopping its main process..


```

[![kungfu-bruce-lee.jpg](https://d23f6h5jpj26xu.cloudfront.net/ggwzuy5l9wr1sq_small.jpg)](http://img.svbtle.com/ggwzuy5l9wr1sq.jpg)


```bash

$ docker run -it ubuntu:14.04.1 /bin/bash
root@12d7aa42efb5:/#

# To exit without stopping the container hit Ctrl + P + Q
# We call this "detaching" from our Container

root@12d7aa42efb5:/# %
$


# But we can stop it from the outside with a SIGTERM

$ docker stop 12d7aa42efb5

# or a SIGKILL

$ docker kill 12d7aa42efb5

# or another signal with 

$ docker kill -s <signal> 12d7aa42efb5



```

These signal go inside the container to the process running PID 1, in our case is the bash process. It's important to know that to get a shell prompt after you attach the Container has to be running a shell as process 1. The attach command attaches us to the PID1 inside the container.

> But but.. PID 1 should be **init**

For the sake of simplicity let's say all the Linux server processes are forked from the init (the mother of all processes) and she is used to receive the signals and gracefully kill / stop / manage its child processes.

On the other hand our Containers usually run Application daemons that are not designed to do the **init** job, so in the event of receiving a SIGTERM there is no guarantee they will warn the other processes. **This is one of the major reasons why we should run one process per Container.**

*We can and it's a reality running multiple processes per Container, it's your choice.*

<hr></hr>

This article is part of White Noise Open Source Series meaning that the article can be updated / corrected / optimised by pull request at our public repositories.

> **☵☲ <a href="https://www.coinbase.com/edge" target=_blank>Buy me a beer</a> ☵☲** hkd up by Flávio HG



