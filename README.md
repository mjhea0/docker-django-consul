# Docker, Django, and Consul

## Part 1 - setup

### Consul

Source: https://hub.docker.com/r/progrium/consul/

1. Create new Docker Machine, and then make it active:

  ```sh
  $ docker-machine create --driver virtualbox consul

  $ eval $(docker-machine env consul)
  ```

1. Pull consul image and run it as a daemon:

  ```sh
  $ docker pull progrium/consul:latest

  $ docker run --name consul -d -p 8400:8400 -p 8500:8500 -p 8600:53/udp -h node1 progrium/consul -server -bootstrap -ui-dir /ui
  ```

  NOTE: The compose file would look like (sans UI):

  ```yaml
  consul:
    image: progrium/consul
    restart: always
    hostname: consul
    ports:
      - 8500:8500
    command: "-server -bootstrap"
  ```

1. Get IP:

  ```sh
  $ docker-machine ip consul
  192.168.99.100
  ```

1. View the web interface (if used) @ http://192.168.99.100/:8500/ui/#/dc1/services

### Swarm

1. Create one Swarm master and three nodes:

  ```sh
  $ docker-machine create --driver virtualbox --swarm --swarm-master --swarm-discovery consul://192.168.99.100:8500/ swarm-master

  $ docker-machine create --driver virtualbox --swarm --swarm-discovery consul://192.168.99.100:8500/ swarm-node-1

  $ docker-machine create --driver virtualbox --swarm --swarm-discovery consul://192.168.99.100:8500/ swarm-node-2

  $ docker-machine create --driver virtualbox --swarm --swarm-discovery consul://192.168.99.100:8500/ swarm-node-3
  ```

1. Ensure that consul is aware of the master and nodes @ http://192.168.99.101:8500/ui/#/dc1/kv/docker/swarm/nodes/

## Part 2 - add containers

Django, Postgres, nginx