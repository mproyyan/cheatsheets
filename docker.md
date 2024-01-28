<h1 align="center">Docker</h1>

## Basic Docker

### Image

#### List image

```bash
docker image ls
```

#### Pull image

```bash
# docker image pull imagename:tag
docker image pull alpine:latest
```

#### Remove image

Cannot delete the Docker image that is being used by the container. You must first stop and delete all containers that use the image

```bash
# docker image rm imagename:tag
docker image rm alpine:latest
```
