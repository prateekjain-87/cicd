## Container vs VM

Containers are nothing but OS processes whereas VMs run complete operating system including the kernel.

Unlike virtual machines where hypervisor divides physical hardware into parts, Containers uses concept of namespaces where each namespace has its own isolated resources without actual partitioning of the underlying hardware. This is the reason why launching a container takes seconds and creating a virtual machine takes minutes.

![alt text](https://github.com/prateekjain-87/cicd/blob/main/vm-containers.png?raw=true)

## Docker architecture

Docker is a tool that packages, provisions and runs containers independent of the OS.

Docker Client is the host where Docker environments should be installed to build Docker images with a target application.

Docker Host is a managed host with Docker daemon (also known as Dockerd which is the persistent process that manages containers).

Docker Registry provides or stores different Docker images. The best known open communities for Docker images are Docker hub, nginx (the official Docker image) and the Docker store.


![alt text](https://github.com/prateekjain-87/cicd/blob/main/docker.png?raw=true)


The Docker client can communicate with Docker daemon by using the RESTful API over UNIX sockets or a network interface in two ways:
•	Docker clients can run on the same system with Docker daemon.
•	Docker clients connect to a remote Docker daemon.


#### Connection to remote docker host:
![alt text](https://github.com/prateekjain-87/cicd/blob/main/remote_docker.png?raw=true)

![alt text](https://github.com/prateekjain-87/cicd/blob/main/container.png?raw=true)


