# docker management

## docker container

## docker image

## docker network

### Driver
1. **`bridge`**: default network driver. Bridge networks are usually used when your applications run in standalone containers that need to communicate.
2. **`host`**: only available for `swarm services on Docker 17.06 and higher`. For standalone containers, remove network isolation between the container and the Docker host, and use the host’s networking directly.
3. **`overlay`**: Overlay networks connect multiple Docker daemons together and enable swarm services to communicate with each other. 
4. **`macvlan`**: Macvlan networks allow you to assign a MAC address to a container, making it appear as a physical device on your network. 
5. **`none`**: For this container, disable all networking. Usually used in conjunction with a custom network driver. 
6. **`Network plugins`**: You can install and use third-party network plugins with Docker. 

## docker plugin

## docker system

## docker trust

## docker volume
By default all files created inside a container are stored on a writable container layer. This means that:
+ The data doesn’t persist when that container is no longer running, and it can be difficult to get the data out of the container if another process needs it.
+ A container’s writable layer is tightly coupled to the host machine where the container is running. You can’t easily move the data somewhere else.
+ Writing into a container’s writable layer requires a storage driver to manage the filesystem. The storage driver provides a union filesystem, using the Linux kernel. This extra abstraction reduces performance as compared to using data volumes, which write directly to the host filesystem.

### Type
1. **`volumes`**: are stored in a part of the host filesystem which is managed by Docker `(/var/lib/docker/volumes/ on Linux)`.Non-Docker processes should not modify this part of the filesystem. Volumes are the best way to persist data in Docker.
2. **`bind mounts`**: may be stored *anywhere* on the host system. They may even be important system files or directories. Non-Docker processes on the Docker host or a Docker container can modify them at any time.
3. **`tmpfs mount`**(only for linux host): are stored in the host system’s memory only, and are never written to the host system’s filesystem.