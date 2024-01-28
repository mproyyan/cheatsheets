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
