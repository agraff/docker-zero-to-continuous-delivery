## Docker Swarm

Swarm Manager - only one instance running.
Swarm Agent - running on each workstation.
Consul - a distributed key/value storage, used for service discovery. Stores data about the agents (their IP address and port).

Swarm Agent registers itself with Consul.
Swarm Manager queries Consul to get details about all agents, then communicates directly with the agents.