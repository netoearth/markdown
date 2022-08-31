Sometimes it's useful to start with a clean slate and remove all Docker containers and even images. Here are some handy shortcuts.

## List all containers (only IDs)

```
docker ps -aq
```

## Stop all running containers

```
docker stop $(docker ps -aq)
```

## Remove all containers

```
docker rm $(docker ps -aq)
```

## Remove all images

```
docker rmi $(docker images -q)
```