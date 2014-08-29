# Docker

## Linux Container

* namespace
pid、net、ipc、mnt、uts、user等namespace将container的进程、网络、消息、文件系统、UTS("UNIX Time-sharing System")和用户空间隔离开。

* cgroups
cgroups 实现了对资源的配额和度量。 cgroups 的使用非常简单，提供类似文件的接口，在 /cgroup目录下新建一个文件夹即可新建一个group，在此文件夹中新建task文件，并将pid写入该文件，即可实现对该进程的资源控制。

* AUFS (AnotherUnionFS)
支持将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)的文件系统

* AppArmor, SELinux, GRSEC

## Docker

基于LXC技术之上构建的Container容器引擎

### Components

* Docker client and server
* Docker Images
* Registries
* Docker Containers
	* An image format
	* A set of standard operations
	* An execution environment

## boot2docker

`boot2docker start`<br />
`boot2docker up`<br />
`boot2docker update`<br />
`boot2docker down`<br />
`boot2docker ip`<br />

`docker info`<br />
`docker run -d -P --name web nginx`: 
	`-d` daemon `-P` automatically forwards the ports specified in the image
	
`docker ps`<br />
`echo $(docker-ip) dockerhost | sudo tee -a /etc/hosts`<br /> (Port Forwarding)
`docker stop web`<br />
`docker rm web`<br />

`docker run -i -t ubuntu /bin/bash`:
	`-i` keep STDIN open from container `-t`assign a pseudo-tty to container

`docker start`<br />
`docker attach`<br />
`docker inspect`<br />


**`docker run` with -v will not work with boot2docker**

## References

* http://www.infoq.com/cn/articles/docker-core-technology-preview
* http://viget.com/extend/how-to-use-docker-on-os-x-the-missing-guide