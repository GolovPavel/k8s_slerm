# Docker compose practice

Creates 2 containers:

- the python web application;
- the redis server.

For the redis container creates the volume and limits by cpus and memory.
The web application is dependend on redis server using health check.

## How to start the applications

```
docker-compose up
```
