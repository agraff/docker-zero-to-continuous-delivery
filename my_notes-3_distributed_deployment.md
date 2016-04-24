# Docker Swarm

Swarm Manager - only one instance running.
Swarm Agent - running on each workstation.
Consul - a distributed key/value storage, used for service discovery. Stores data about the agents (their IP address and port). Consul consists of a cluster of Consul servers, and many Consul agents.

Swarm Agent registers itself with Consul.
Swarm Manager queries Consul to get details about all Swarm Agents, then communicates directly with the Swarm Agents.


On each workstation, we will run a Consul Agent and Swarm Agent. The Swarm Agent communicates with the local Consul Agent. The Consul Agent communicates with the Consul Server.
The Consul Server will replicate the data with all the Consul Agents (eventual consistency).
On the machine(s) with the Swarm Manager, there is a Consul Agent running, which the Swarm Manager communicates with to get data about the Swarm Agents.


## Build the infrastructure

### Start the Consul Server Cluster
server config stored at: /var/data/consul_config/consul_server.json

$ docker run -d -v /var/data/consul_config/:/config -v /var/data/consul_data/:/data --net=host --label consul_server --name consul_server gliderlabs/consul-server:0.6
$ docker logs consul_server
Repeat this twice (on two different machines), so there are three servers - the minimum needed for a Consul server cluster.

### Start the Consul Agent on each workstation
agent config stored at: /var/data/consul_config/consul_agent.json
{
  "datacenter": "training",
  "node_name": "Workstation 7",
  "advertise_addr": "10.2.0.21",  # This is different on each 
  "retry_join": ["10.2.0.191","10.2.0.126","10.2.0.41"],
  "data_dir": "/data",
  "log_level": "INFO",
  "client_addr": "0.0.0.0",
  "leave_on_terminate": true
}

$ docker run -d -v /var/data/consul_config/:/config -v /var/data/consul_data:/data --net=host --label consul_agent --name consul_agent gliderlabs/consul-agent:0.6

### Start the Swarm Manager
First, start a consul agent on this machine. Then start swarm manager:
$ docker run -d -p:4567:2375 --label swarm_manager --name swarm_manager swarm:1.0.1 manage consul://<ip_address_of_local_consul_agent>:8500/training

$ docker -H tcp://<ip_address_of_swarm_manager>:4567 info

### Start the Swarm Agent

$ docker run -d --label swarm_agent --name swarm_agent swarm:1.0.1 join --advertise=<ip_address_of_swarm_agent>:2375 consul consul://<ip_address_of_swarm_agent>:8500/training_swarm

<ip_address_of_swarm_agent> is used to announce swarm agent, and is stored in consul key/value store. You should see it in the Key/Value page of the Consul UI.

$ docker -H tcp://<ip_address_of_swarm_manager>:4567 info
e.g. $ docker -H tcp://swarm-manager.training.local:4567 ps


Can use ansible, chef, AMI images, etc to run the commands needed to build all the swarm/consul infrastructure.


### Configure TeamCity
Edit the build step for 3. Deploy, so it looks like this:
#!/bin/bash
set -e
export IMAGE_VERSION=$(cat release.version)
export DOCKER_HOST=tcp://swarm-manager.training.local:4567
docker-compose down
docker-compose up -d

The change is to set the DOCKER_HOST to be the swarm-manager, and ** must be using the port of the swarm manager - not the port of docker **. Otherwise it will deploy to the swarm-manager machine.

Change the docker-compose.yml file so a random port is assigned.
replace:
```
    ports:
      - "80:4567"
```
with:
```
    ports:
      - "4567"
```
But now need a proxy / load balancer to route to the services running on random ports. See notes about service discovery.


Scale the service, by changing the build step for 3. Deploy.
replace:
```
docker-compose down
docker-compose up -d
```
with:
```
docker-compose scale workstation-7=0
docker-compose scale workstation-7=4
```

Make sure that the service name in docker-compose.yml matches the service name used with docker-compose above (e.g. workstation-7).