# Docker architecture

Docker is a tool that packages, provisions and runs containers independent of the OS.
Docker Client where Docker environments should be installed to build Docker images with a target application.
Docker Host is a managed host with Docker daemon (also known as Dockerd which is the persistent process that manages containers).
Docker Registry provides or stores different Docker images. The best known open communities for Docker images are Docker hub, nginx (the official Docker image) and the Docker store.


![alt text](https://github.com/prateekjain-87/cicd/blob/main/docker.png?raw=true)


The Docker client can communicate with Docker daemon by using the RESTful API over UNIX sockets or a network interface in two ways:
•	Docker clients can run on the same system with Docker daemon.
•	Docker clients connect to a remote Docker daemon.