
# docker-host

## Fork to stop the image using host.docker.internal as a way of picking the docker host IP. This allows the image itself to run with a network alias of 'host.docker.internal' and therefore can be used on Linux systems as if you were using Docker for Mac.
Docker image to forward all traffic to the docker host 
* Uses default gateway as docker host

You can manually override the destination IP address by setting the environment variable `DOCKER_HOST`.
This allows you to use this image to forward traffic to arbitrary destinations, not only the docker host.


# Examples

#### Prerequisite
Simulate localhost webserver on port 8080.
```sh
docker run --name nginx -p 8080:80 \
  -d nginx
```

## Docker Link
Run the dockerhost container.
```sh
docker run --name 'dockerhost' \
  --cap-add=NET_ADMIN --cap-add=NET_RAW \
  --restart on-failure \
  -d qoomon/docker-host
```
Run your application container and link the dockerhost container.
The dockerhost will be reachable through the domain/link `dockerhost` of the dockerhost container e.g. `dockerhost:8080`
This example uses `curl` as an application dummy.
```sh
docker run --rm \
  --link 'dockerhost' \
  appropriate/curl 'http://dockerhost:8080'
```

## Docker Network
Create the dockerhost network.
```sh
network_name="Network-$RANDOM"
docker network create "$network_name"
```
Run the dockerhost container within the dockerhost network.
```sh
docker run --name "${network_name}-dockerhost" \
  --cap-add=NET_ADMIN --cap-add=NET_RAW \
  --restart on-failure \
  --net=${network_name} --network-alias 'dockerhost' \
  qoomon/docker-host
```
Run your application container within the dockerhost network.
The dockerhost will be reachable through the domain/network-alias `dockerhost` of the dockerhost container e.g. `dockerhost:8080`
This example uses `curl` as an application dummy.
```sh
docker run --rm \
  --net=${network_name} \
  appropriate/curl 'http://dockerhost:8080'
```

## Docker Compose
```yaml
version: '2'

services:
    dockerhost:
        image: qoomon/docker-host
        cap_add: [ 'NET_ADMIN', 'NET_RAW' ]
        mem_limit: 4M
        restart: on-failure
    dummy:
        depends_on: [ dockerhost ]
        image: appropriate/curl
        command: ["http://dockerhost:8080"]
```
