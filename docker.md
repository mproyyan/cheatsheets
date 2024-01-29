<h1 align="center">Docker</h1>

## Basic Docker

### Image

#### List image

```bash
docker image ls
```

#### Pull image

```bash
# docker image pull <imagename>:<tag>
docker image pull alpine:latest
```

#### Remove image

Cannot delete the Docker image that is being used by the container. You must first stop and delete all containers that use the image

```bash
# docker image rm <imagename>:<tag>
docker image rm alpine:latest
```

### Container

#### List container

Add the `-a` flag to see all containers whether they are running or not

```bash
docker container ls
```

#### Create container

`--name` is used to define the name of the container. Container names cannot be the same

```bash
# docker container create --name <containername> <imagename>:<tag>
docker container create --name alpinecont alpine:latest
```

#### Running container

```bash
# docker container start <containerID> / <containername>
docker container start alpinecont
```

#### Stop container

```bash
# docker container stop <containerID> / <containername>
docker container stop alpinecont
```

#### Remove container

```bash
# docker container rm <containerID> / <containername>
docker container rm alpinecont
```

#### Container log

These logs include information about important events that occur while the container is running, such as error messages, system events, application output, and more

add the `-f` flag to monitor in real time

```bash
# docker container logs <containerID> / <containername>
docker container logs alpinecont
```

#### Container exec

used to execute commands inside a running container. This is very useful when you need to enter an already running container to perform a task or check its internal health.

```bash
# docker exec [OPTIONS] <container> <command> [ARG...]
# <command> : The command you want to execute in the container.

docker container exec -it rediscont /bin/bash
# -i is an interactive argument, keeping input active
# -t is the argument for pseudo-TTY (access terminal) allocation
# /bin/bash is the command exists and is executed inside the container (in this case, opening a Bash shell inside the container).
```

#### Port forwarding

add the `--publish <port-host>:<port-container>` flag to perform port forwarding

> `<port-host>` : The port on the host that will be used to forward connections to the container.
> <br>

> `<port-container>` : The port in the container that will be connected to the port on the host. To find out, do a container inspection

```bash
docker container create --name nginxcont --publish 8080:80 nginx:latest
# then we can access nginx in the browser via localhost:8080 (0.0.0.0:8080->80/tcp)
```

#### Container environment

When creating an application, using Environment Variables is one of the techniques Application configuration can be changed dynamically. By using environment variables, we can change the application configuration, without have to change the application code again <br>

To add environment variables use the `--env KEY=value` flag

```bash
# we will set the root user password
docker container create --name mysqlcont --env MYSQL_ROOT_PASSWORD=root --publish 3306:3306 mysql:latest
```

#### Container stats

```bash
docker container stats
```

#### Resource limit

We can limit the resources that can be used by the container, in this case memory and CPU

##### Memory

We can add the size in the form of b (bytes), k (kilo bytes), m (mega bytes), or g (giga bytes), for example 100m means 100 mega bytes

```bash
docker container create --name rediscont redis:latest --memory 100m
```

##### CPU

```bash
docker container create --name rediscont redis:latest --cpus 0.5
```

### Bind mount

Bind mounts allow you to bind directories or files from the host system into a container running using Docker. This feature is very useful when, for example, we want to send configurations from outside the container, or for example, store data created in an application in a container into a folder on the host system (sharing). <br>

To bind mount use `--mount “type=bind,source=folder,destination=folder,readonly”`

| Parameter   | Description                                                                              |
| ----------- | ---------------------------------------------------------------------------------------- |
| type        | bind or volume                                                                           |
| source      | The location of the file or folder on the host system                                    |
| destination | The location of the file or folder in the container                                      |
| readonly    | If there is, then the file or folder can only be read in the container, can't be written |

```bash
docker container create --name mysqlcont --mount "type=bind,source=/home/roy/mysql,destination=/var/lib/mysql" --env MYSQL_ROOT_PASSWORD=root --publish 3306:3306 mysql:latest
```

### Volume

Docker Volumes are similar to Bind Mounts, the difference is that there is Volume management, where we can create Volumes, view the Volume list, and delete Volumes. Volume itself can be considered storage used to store data, the difference is with Bind Mounts, in bind mounts, the data is stored on the host system, whereas in volumes, the data is stored on the host system managed by Docker

#### List volume

```bash
docker volume ls
```

#### Create volume

```bash
# docker volume create <volumename>
docker volume create mysqlvol
```

#### Remove volume

```bash
# docker volume rm <volumename>
docker volume rm mysqlvol
```

#### Bind volume

```bash
docker container create --name mysqlcont --mount "type=volume,source=mysqlvol,destination=/var/lib/mysql" --env MYSQL_ROOT_PASSWORD=root --publish 3306:3306 mysql:latest
```

### Network

Network allows containers to communicate with each other within an isolated network. With Docker Network, you can create a dedicated network for your container groups so they can communicate without having direct exposure to outside networks or hosts

#### List network

```bash
docker network ls
```

#### Create network

```bash
# docker network create --driver <driver> <network-name>
# driver : bridge, host, none

docker network create --driver bridge mynetwork
```

#### Remove network

```bash
# docker network rm <network-name>
docker network rm mynetwork
```

### Container network

We can insert a container into the network so that the container can access other containers by specifying the hostname of the container, namely the name of the container

#### Create container with network

Use `--network <network-name>`

```bash
docker container create --name nginxcont --network mynetwork nginx:latest
```

#### Remove container from network

```bash
# docker network disconect <network-name> <container-name>
docker network disconect mynetwork nginxcont
```

#### Add already exists container to network

```bash
# docker network connect <network-name> <container-name>
docker network connect mynetwork nginxcont
```

## Dockerfile

A Dockerfile is a text file that contains a set of instructions used to create a Docker image. A Dockerfile defines the steps and configuration required to form an image that can be used to create a Docker container.

### Docker build

Command used to build a Docker image from a Dockerfile. Use the `--progress=plain` option to display detailed build text and the `--no-cache` option to allow the build to be rebuilt

```bash
# docker build [OPTIONS] <dockerfile-path>
# -t : provide a name and tag for the image being built
docker build -t mproyyan/app:1.0.1 .
```

### FROM

`FROM` is used to create a **build stage** from the image that we specify

```dockerfile
# FROM <image-name>:<tag>
FROM alpine:latest
```

### RUN

`RUN` is an instruction to execute commands in the image during the **build stage** only, so once it has become a Docker image it will not be executed again

```dockerfile
# RUN has 2 formats
# RUN command
# RUN ["executable", "argument", "..."]

FROM alpine:latest

RUN echo "hello world" > "hello/world.txt"
RUN ["cat", "hello/world.txt"]
```

### CMD

`CMD` is an instruction that is used when the Docker Container is running, so it will not be executed during the build stage

```dockerfile
# CMD command param ...
# CMD ["executable", "param", ...]

FROM alpine:3

RUN mkdir hello
RUN echo "hello world" > "hello/world.txt"

CMD cat "hello/world.txt"
```

### LABEL

The `LABEL` instruction is an instruction used to add metadata to the Docker Image that we create

```dockerfile
# LABEL <key1>=<value1> <key2>=<value2> …

FROM alpine:3

# use docker image inspect to see metadata
LABEL author="Muhammad Pandu Royyan"
LABEL message="its joever"
```

### ADD

`ADD` is an instruction that can be used to add files from source to the destination folder in Docker Image. The `ADD` command can detect whether a source file is a compressed file such as tar.gz, gzip, etc. If it detects that the source file is a compressed file, the file will automatically be extracted in the destination folder

```dockerfile
# ADD <source> <destination>
FROM alpine:3

RUN mkdir hello
ADD text/world.txt
ADD text/*-test.txt hello

RUN ls hello/
```

### COPY

`COPY` is similar to `ADD`, but the difference is that `COPY` only copies files, while `ADD`, apart from copying, can download sources from URLs and automatically extract compressed files.

```dockerfile
# COPY <source> <destination>
FROM alpine:3

RUN mkdir hello
COPY text/world.txt
COPY text/*-test.txt hello

RUN ls hello/
```

### EXPOSE

`EXPOSE` is an instruction to notify that the container will listen to a port on a certain number and protocol. `EXPOSE` will not publish or forward any ports because this command is only used to notify that the container will be run on that port number.

```dockerfile
# EXPOSE <port-number>
FROM golang:1.21-alpine3.17

RUN mkdir app
COPY main.go app
EXPOSE 8080

CMD go run app/main.go

# So if you want to create a container with this image
# and want to do port forwarding then use --publish <host-port>:8080
```

### ENV

`ENV` is an instruction used to change environment variables, either during the build stage or when running in a Docker Container

```dockerfile
# ENV <key1>=<value1> <key2>=<value2> …
FROM golang:1.21-alpine3.17

ENV APP_PORT=8080
RUN mkdir app
COPY main.go app

EXPOSE ${APP_PORT}

CMD go run app/main.go
# You can change it when creating the container by including --env APP_PORT=<your-port-number>
```

### VOLUME

The `VOLUME` command is used to declare a volume on a container. It is important to note that the `VOLUME` command **does not create a volume directly**, but only **declares** a volume on a specific path. The volume will be created automatically when the container is run or can be connected to a host volume or other Docker volume by creating it manually using the `docker volume create <volume-name>` command.

```dockerfile
FROM golang:1.21-alpine3.17

ENV APP_PORT=8080
ENV APP_DATA=/logs

RUN mkdir ${APP_DATA}
RUN mkdir app
COPY main.go app

EXPOSE ${APP_PORT}
VOLUME ${APP_DATA}

CMD go run app/main.go
```

```bash
docker build -t mproyyan/volume:latest .
```

```bash
docker volume create golangvol
```

```bash
docker container create --name volume-golang --publish 8080:8080 --mount "type=volume,source=golangvol,destination=/logs" mproyyan/volume:latest
```

### WORKDIR

`WORKDIR` is an instruction to specify the directory/folder to execute the instruction `RUN`, `CMD`, `ENTRYPOINT`, `COPY` and `ADD`. `WORKDIR` can also be used as a path to the first location when we enter Docker Containers

```dockerfile
FROM golang:1.21-alpine3.17

WORKDIR /app
COPY main.go .

EXPOSE 8080
CMD go run main.go
```

### USER

USER is the instruction used to change the user or user group when Docker Image is run. By default, Docker will use the root user, but in some cases, there may be applications that don't want to run in the root user, so we can change the user using the USER instruction

> `USER <user>` change user
> <br>
> `USER <user>:<group>` change user and group

```dockerfile
FROM alpine:3

RUN mkdir /app

# This command is used to create a new user group with the name roygroup.
# The -S option is used to create a system group, which is usually used for groups
# that is not associated with the user
RUN addgroup -S roygroup

# This command is used to create a new user with the name royuser.
# The -S and -D options are used to create system users without a login shell and without
# home directory. This user is also assigned to a previously created roygroup.
# The user's home directory is set to /app, which was created with the first command
RUN adduser -S -D -h /app royuser roygroup

# This command is used to change the ownership and ownership group of all content
# /app directory to royuser and roygroup. This aims to ensure that Royuser users have access
# read/write to directory
RUN chown royuser:roygroup /app

USER royuser

RUN whoami
RUN ls -la /
```
