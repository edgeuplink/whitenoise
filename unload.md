>UnDock from your comfort zone and let's learn about Docker. What I wish somebody told me before picking up Docker bandwagon is what I'm going to share with you in a full series about the technology and everything important happening behind the scenes.

[![Containers_1280x800©.jpg](https://d23f6h5jpj26xu.cloudfront.net/add9qrhj7wrapa_small.jpg)](http://img.svbtle.com/add9qrhj7wrapa.jpg)

As the prerequisites the more you know about Linux the better but we'll keep it accessible all along. And you can spin up 1-2 Linux virtual machines already.
 
> # What we'll learn today

- **Linux Containers** 

In order to best teach concepts and the likes there are gonna be times we outright violate and trumple all over best practices (to highlight things that we are leaning). We'll circle back later and explain how to do it the right way. Repetition is the mother of learning.

># Introducing Containers

Let's get back in time just a bit..

It's all about applications, back in the day we used to build servers and application in a 1 : 1 ratio. For every application we wanted to deploy we needed to buy and build a dedicated server for it. 

For 10 applications, 10 servers, 10 purchase orders, 10 operating systems installs, 10 licenses, 10 rack positions, 10 sets of redundant power, networking and storage infrastructure... And it quickly adds up. 

[![olddays.png](https://d23f6h5jpj26xu.cloudfront.net/zjdb5qhpeng8a_small.png)](http://img.svbtle.com/zjdb5qhpeng8a.png)

**Not great for AGILITY.** 

Bad old days. 

> **Massive waste** of resources

Each one of those servers, 99 times out of a hundred we only run them at a tiny fraction of their capability. Servers would be running at less than 10% average utilisation (some times 2 to 3%). We ended up having tons of proper servers sitting in the datacenter doing absolutely nothing most of the time.

[![resources.png](https://d23f6h5jpj26xu.cloudfront.net/ig8ov54lwqcigq_small.png)](http://img.svbtle.com/ig8ov54lwqcigq.png)

Something had to be done. And something was done.

> **Virtualisation** (Hypervisor Virtualization)

We take a physical machine and we cover it up in multiple virtual machines. Each Virtual Machine looks feels and tastes like a physical machine. Let's assume one app per virtual machine.

Take the previous 10 apps and each one consumes less than 10% of cpu / storage / ram from its physical machine. We can quite conceivably run all 10 apps in one single physical machine. One app in each virtual machine.

[![virtualmodel.png](https://d23f6h5jpj26xu.cloudfront.net/u2d0ztiq4ze6cw_small.png)](http://img.svbtle.com/u2d0ztiq4ze6cw.png)

Overnight we jumped from utilising only 10% of physical machine capabilities at a 1:1 ration to multiple apps running in the same physical server with profiles using up to almost 80% of the resources.

**It was a revolution** (massive improvement) but still far from perfect...

Now every app *"needed"* its virtual machine and like it was the last cookie we went commando and overindulged & over-consumed VMs ... 

**Let's take a closer look at the VM model:**

With one app per VM we still need one OS per VM, with this model we trimmed only the excess at the physical layer.

[![virtualos.png](https://d23f6h5jpj26xu.cloudfront.net/syk55cavyzlka_small.png)](http://img.svbtle.com/syk55cavyzlka.png)

**Meh.. not so cool..** every OS consumes CPU/RAM/Storage resources and some need INDIVIDUAL licensing. More OS is not better.

This VM model it's still pretty secure but each OS is a gross waste of resources.

> It's all about the Application

all it needs to run it's a secure isolated environment with minimum OS services and probably some QoS controls. The VM literally is now the bloated heavyweight in the room.

Something more efficient came along.

> # The Container

At a mega high level containers are a bit like virtual machines but the main point is that containers are **way more lightweight** than virtual machines.

Each and every container consumes **LESS** CPU/RAM/Storage than a VM and still provide a secure isolated environment to apps to live in.

> **Linux Containers**

After installing linux over bare-metal server the linux kernel owns and manages the hardware underneath it. Then in a world without containers we install an application or a service on a construct we call USER SPACE.

Well what Containers do is let us create multiple isolated instances of this USER SPACE area on top of the **same** operating system. Each isolated instance of user space is a container and then within each container we install an app or a service.

[![container.png](https://d23f6h5jpj26xu.cloudfront.net/5da9gx84wvpx4a_small.png)](http://img.svbtle.com/5da9gx84wvpx4a.png)

Conceptually we might hear it referring to **container virtualisation** or **OS level virtualisation**.

Back to the 10 apps example we just slashed resource utilisation from 10 OS (1 per each VM) pilling up about 100GB disk space / 40GB Ram / 50% CPU before we even install a single app to just **ONE** operating system, in this case all the containers share the same instance of a Linux kernel.

And Containers are also **faster** and more **portable** than VMs.

> Before we get under the hood you ask : *Why don't we install 5 applications on top of the same operating system and on the same User Space ?*

Well.. in that scenario we cannot stop one application from walking over each other (we can but it's not as manageable as containers). Imagine two applications need the same OS library but with different versions, it's a disaster waiting to happen and operating systems are not good at managing it in the same User Space.

Back to containers, a system using containers will have **isolated and independent instances of user space**, meaning (for example) we can have OS library version 1.0 for app container X and the **same** OS library running version 2.3 for app container Y.

> # **Under the hood** (How Containers work)

Inside a container we can have isolated instances of 

- **root filesystems** : An app running on container X can add/remove/edit any file within its view of the root filesystem without impacting apps in other containers.
- **process trees** : Same goes for the independent process trees. A process inside one container CANNOT send a signal to a process inside another container. (pid Namespace)
- **networking stacks** : Same here. Its own IP addresses, port ranges, routing table.

> ...Sounds great. Thank you **Kernel Namespaces**

For example the **user** Namespace allow us to have users with root priviledges INSIDE a container but not OUTSIDE the container.

Look at the pid Kernel Namespace : 

[![pidkernel.png](https://d23f6h5jpj26xu.cloudfront.net/icapzrf7wh2lrq_small.png)](http://img.svbtle.com/icapzrf7wh2lrq.png)

> And don't forget Control Groups (**cgroups**)

At a high level **cgroups** kernel feature let us group together resources and apply limits. Kind of like Quality of Service rules, in the case of containers we map containers to cgroups in a 1 : 1 mapping , and then we set limits on how much CPU/RAM/Block IO that the container has access to.

They are *very flexible* and much more *easy to manage and tweak* than vCPU or vRAM. Preventing that one container runs amok on the system and DOS every other container.

[![cgroup.png](https://d23f6h5jpj26xu.cloudfront.net/awayhtfr1s2xa_small.png)](http://img.svbtle.com/awayhtfr1s2xa.png)

When dealing with containers with hostile workloads sharing the same machine you should pick a container system that supports cgroups as Docker does.

> Have you heard of **Capabilities** ?

In the need of fine grain control of user/process privileges this kernel feature allow us to break down the **all or nothing** approach of root and non root users. 

It breaks down the root privileges into smaller privileges and allow us to assign them to a user/process when it needs them.

For example you need to assign a socket to a low numbered port within a container but aside from that you don't need any other special privilege. Grant that process a *CAP_NET_BIND_SERVICE* capability (that does what you are thinking) instead of granting full root privileges. 

[![capabilities.png](https://d23f6h5jpj26xu.cloudfront.net/foaokgkx6rvsg_small.png)](http://img.svbtle.com/foaokgkx6rvsg.png)

So capabilities are important from a **security** perspective.

Docker supports capabilities and operates in a **whitelist approach** where all are denied by default except all that are explicit listed on the whitelist.

> # AND THAT IS THE VERY BASICS OF HOW CONTAINERS OPERATE BEHIND THE SCENES

[![033 - fIk6nXr.jpg](https://d23f6h5jpj26xu.cloudfront.net/37h6jp6iongja_small.jpg)](http://img.svbtle.com/37h6jp6iongja.jpg)

>## Follow the full series 

- **Linux Containers** (in this article)
- **<a href="http://up.svbtle.com/undock" target="_blank">Docker Engine</a>** (next article)
- Docker Major Components

<hr></hr>

This article is part of White Noise Open Source Series meaning that the article can be updated / corrected / optimised by pull request at our public repositories.

> **☵☲ <a href="https://www.coinbase.com/edge" target=_blank>Buy me a beer</a> ☵☲** hkd up by Flávio HG

