# Docker volumes

There are 2 ways to use persistance in Docker: binding and volumes.
Volumes are more secure way to store data.
Moreover, we can run our application without specific location on the disk with volumes.

## How to build container 
```
docker build -t volume_test .
```

## How to start container with specific volume name
```
docker run --rm --name volume_test -d --mount type=volume,source=data,destination=/data volume_test
```

## Documentation
https://docs.docker.com/storage/bind-mounts/
