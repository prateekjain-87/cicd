docker run -t nginx
docker run -it nginx
docker run -dit nginx
docker run -dit --name test1 nginx
docker exec -it /bin/sh
docker cp ~/docker/test test1:/
docker run -p 8080:80 -dit nginx
docker logs -f 
docker rm
docker network ls
https://docs.docker.com/engine/reference/commandline/network_create/
docker run -dit --network=multi-host-network busybox
https://docs.docker.com/storage/bind-mounts/
docker volume create myvol   — attach to container
docker volume inspect myvol
docker run -dit -v ~/docker/:/  --name test2 nginx
docker run -dit -v ~/docker/:/app  --name test2 nginx
Timestamp
Edit from container
docker image inspect nginx

https://www.geeksforgeeks.org/docker-arg-instruction/
