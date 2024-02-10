# Development environment

## Prerequisites

- Ensure you installed Docker on your machine. If not, please follow their official documentation.[^1]
- Ensure you cloned our repository.

## Overview

In this setup, we are going to:
1. Write Dockerfile for building custom Docker image for Redis.
2. Build the Docker image.
3. Configure Redis
4. Run Redis cluster on three container.

> [!IMPORTANT] 
> We strongly advise relying on the official documentation and support services [^2]. In this context, we offer a brief overview of the cluster setup with fundamental configurations.


## Write a Dockerfile

We have written two Dockerfiles, one for the base image and another for the Redis image, which already exist in the `docker` directory.

```bash
├── docker
│ ├── dockerfile.base.dev
│ └── dockerfile.redis.7.2.4
```

For best practices, 
- we install all the necessary softwares, included testing and debugging tools in the base image. This approach applicable only for development (DEV) environment. 
- We download and install Redis source in the Redis image.


## Build the base and Redis Docker image

Here, we derive the base image from the **Amazon Linux 2023** Docker image. Then, we create the Redis image based on the previously derived base image.

**Step 1:** Switch to the `redis-cluster` directory.

```bash
cd /opt/oss/redis-cluster
```

**Step 2:** Download Redis source from their official website.

```bash
wget https://github.com/redis/redis/archive/7.2.4.tar.gz -P docker/context/binary
```

**Step 3:** Build the base image.

```bash
docker image build -t redis-base:dev -f docker/dockerfile.base.dev docker/context
```

**Step 4:** Build the Redis image.  Suppose if you built separate base image for production and named `redis-base:prd`, you can use `--build-arg="ENV=prd"` flag to change the `ENV` arguments value in the `dockerfile.redis.7.2.4`.

```bash
docker image build -t redis:v7.2.4 -f docker/dockerfile.redis.7.2.4 docker/context
```


## Configure Redis

When running the cluster setup, it is crucial to maintain quorum for cluster stability. So, we are going to provision three nodes, each being a Docker container. We have already stored the configuration for each node in the `source/conf` directory.  You can get the configuration template from their official documentation [^3].

```bash
├── source
│ └── conf
│     ├── master.conf
│     ├── replica.conf
```

## Run Redis cluster

It is good practice to store configurable variables in the `.env` file and use this file when bringing up the container using Docker Compose.

**Step 1:** Add an environment variable file.

- Open a new file. 

```bash
vim docker/default.env
```

- Copy and paste the following content, then save the file.

```bash
REDIS_IMAGE=redis:v7.2.4
REDIS_NODE_IP_RANGES=10.1.1.0/24
REDIS_NODE_1_IP=10.1.1.11
REDIS_NODE_2_IP=10.1.1.12
REDIS_NODE_3_IP=10.1.1.13
```

**Step 2:** Up the container for Redis cluster.

```bash
docker compose -f docker/compose.yml --env-file docker/default.env -p redis-cluster up -d
```

**Step 3:** Create Redis cluster. **Execute below command only once.**

```bash
export NODE=node1
docker container exec -it redis-cluster-${NODE}-1 /usr/local/bin/redis-cli --cluster create 10.1.2.11:3000 10.1.2.21:3000 10.1.2.22:3000 10.1.2.11:3001 10.1.2.21:3001 10.1.2.22:3001 --cluster-replicas 1
```

By default redis required 6 nodes.  Below is the sample output from Redis service.

```bash
*** ERROR: Invalid configuration for cluster creation.
*** Redis Cluster requires at least 3 master nodes.
*** This is not possible with 2 nodes and 1 replicas per node.
*** At least 6 nodes are required.
```

Run below command to list cluster nodes.

```bash
docker container exec -it redis-cluster-${NODE}-1 /usr/local/bin/redis-cli -p 3000 cluster nodes
```


**Step 4:** Down the Redis cluster.

```bash
docker compose -f docker/compose.yml --env-file docker/default.env -p redis-cluster down
```

## Other useful commands

```bash
export NODE=node1
docker container exec -it redis-cluster-${NODE}-1 /bin/bash
docker container exec -it redis-cluster-${NODE}-1 /usr/bin/netstat -ntlp
docker container exec -it redis-cluster-${NODE}-1 /usr/bin/tail -n 200 /opt/redis/log/redis.log
```

- Cluster creation.

```bash
docker container exec -it redis-cluster-${NODE}-1 /usr/local/bin/redis-cli --cluster create 10.1.1.11:3000 10.1.1.12:3000 10.1.1.13:3000 10.1.1.11:3001 10.1.1.12:3001 10.1.1.13:3001 --cluster-replicas 1
```

```
# Sample output.
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 10.1.2.21:3001 to 10.1.2.11:3000
Adding replica 10.1.2.22:3001 to 10.1.2.21:3000
Adding replica 10.1.2.11:3001 to 10.1.2.22:3000
M: ac890bf2e2f68e6d9fb78ecb343541cdcfea52e3 10.1.2.11:3000
   slots:[0-5460] (5461 slots) master
M: de80094ea9dea4c8e16f3a25de42e87214ba1eb3 10.1.2.21:3000
   slots:[5461-10922] (5462 slots) master
M: 49c4be3036b553122b3e1bf840158cc3160c030b 10.1.2.22:3000
   slots:[10923-16383] (5461 slots) master
S: 141a46c39350309d2bd797a604280e417ba736cf 10.1.2.11:3001
   replicates 49c4be3036b553122b3e1bf840158cc3160c030b
S: dcacee0465055edebc29aa21efbff98a9b6b3265 10.1.2.21:3001
   replicates ac890bf2e2f68e6d9fb78ecb343541cdcfea52e3
S: 92d03cf00df94b66795e2d7677e03fae807af167 10.1.2.22:3001
   replicates de80094ea9dea4c8e16f3a25de42e87214ba1eb3
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join

>>> Performing Cluster Check (using node 10.1.2.11:3000)
M: ac890bf2e2f68e6d9fb78ecb343541cdcfea52e3 10.1.2.11:3000
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: de80094ea9dea4c8e16f3a25de42e87214ba1eb3 10.1.2.21:3000
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 92d03cf00df94b66795e2d7677e03fae807af167 10.1.2.22:3001
   slots: (0 slots) slave
   replicates de80094ea9dea4c8e16f3a25de42e87214ba1eb3
M: 49c4be3036b553122b3e1bf840158cc3160c030b 10.1.2.22:3000
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: dcacee0465055edebc29aa21efbff98a9b6b3265 10.1.2.21:3001
   slots: (0 slots) slave
   replicates ac890bf2e2f68e6d9fb78ecb343541cdcfea52e3
S: 141a46c39350309d2bd797a604280e417ba736cf 10.1.2.11:3001
   slots: (0 slots) slave
   replicates 49c4be3036b553122b3e1bf840158cc3160c030b
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

- List cluster nodes.

```bash
docker container exec -it redis-cluster-${NODE}-1 /usr/local/bin/redis-cli -p 3000 cluster nodes
```

```
# Sample output.
de80094ea9dea4c8e16f3a25de42e87214ba1eb3 10.1.2.21:3000@13000 master - 0 1706206810479 2 connected 5461-10922
ac890bf2e2f68e6d9fb78ecb343541cdcfea52e3 10.1.2.11:3000@13000 myself,master - 0 1706206808000 1 connected 0-5460
92d03cf00df94b66795e2d7677e03fae807af167 10.1.2.22:3001@13001 slave de80094ea9dea4c8e16f3a25de42e87214ba1eb3 0 1706206807458 2 connected
49c4be3036b553122b3e1bf840158cc3160c030b 10.1.2.22:3000@13000 master - 0 1706206807000 3 connected 10923-16383
dcacee0465055edebc29aa21efbff98a9b6b3265 10.1.2.21:3001@13001 slave ac890bf2e2f68e6d9fb78ecb343541cdcfea52e3 0 1706206809472 1 connected
141a46c39350309d2bd797a604280e417ba736cf 10.1.2.11:3001@13001 slave 49c4be3036b553122b3e1bf840158cc3160c030b 0 1706206809000 3 connected
```

- Get shell access of the Redis server.

```bash
docker container exec -it redis-cluster-${NODE}-1 /usr/local/bin/redis-cli -c -h 10.1.1.11 -p 3000
```

## References
- [Redis lab documentation](https://developer.redis.com/operate/redis-at-scale/scalability/exercise-1/)

[^1]: [Docker Engine installation](https://docs.docker.com/engine/install)
[^2]: [Redis official documentation](https://redis.io/docs/management/scaling/)
[^3]: [Redis configuration file](https://redis.io/docs/management/config-file/)
