# Docker limits (cgroups)

## How to build container
```
docker build -t test_limit .
```

## How to start container with memory and cpu limits
```
docker run --name test_limit -m 100m --cpus 0.5 test_limit
```
