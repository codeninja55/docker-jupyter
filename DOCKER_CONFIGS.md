# Docker Configurations

## Sharing notebooks between the host and container

REF: https://www.dataquest.io/blog/docker-data-science/

REF: https://docs.docker.com/storage/volumes/

```shell
> docker run -p 8888:8888 -v C:/Users/codeninja/Dropbox:/home/codeninja --name=jupyter jupyter-image
```

## Start a Docker container

Start the container with STDOUT/STDERR and forward signals attached to current console.

```powershell
> docker -ai start jupyter
```



## Accessing a Shell in Running Container

```shell
> docker exec -it <container_name_or_id> bash
```

