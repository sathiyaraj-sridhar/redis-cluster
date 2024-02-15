# Development environment

## Prerequisites

- Ensure you installed Docker on your machine. If not, please follow Docker's official documentation.[^1]
- Ensure you cloned the repository.

## Overview

In this setup, we are going to:
1. Write a Dockerfile for building a custom container image for Redis.
2. Build the base and Redis container images.
3. Configure Redis cluster.
4. Run Redis cluster on three containers.

> [!IMPORTANT] 
> I strongly recommend relying on the Redis's official documentation and support services [^2]. In this context, I offer a brief overview of the cluster setup with fundamental configurations.

## Write a Dockerfile

I have written two Dockerfiles, one for the base container image and another for the Redis container image, which already exist in the `docker` directory.

```bash
├── docker
│ ├── dockerfile.base.dev
│ └── dockerfile.redis.7.2.4
```

For best practices,
- Install all the necessary software, including testing and debugging tools, in the base container image. This approach is applicable only for development (DEV) environments.
- Then Download and install Redis in the Redis container image.

## Build the base and Redis container image

Here, I derive the base container image from the **Amazon Linux 2023** container image. Then, I create the Redis container image from the previously derived base container image.

```mermaid
graph TD;
    A[Amazon Linux 2023 container image] --> B[base container image];
    B --> C[Redis container image];
```

**Step 1:** Switch to the `redis-cluster` directory.

```bash
cd /opt/oss/redis-cluster
```

**Step 2:** Download Redis source from their official website.

```bash
wget https://github.com/redis/redis/archive/7.2.4.tar.gz -P docker/context/binary
```

**Step 3:** Build the base container image.

```bash
docker image build -t redis-base:dev -f docker/dockerfile.base.dev docker/context
```

**Step 4:** Build the Redis container image.  Suppose if you built separate base image for production and named `redis-base:prd`, you can use `--build-arg="ENV=prd"` flag to change the `ENV` argument value in the `dockerfile.redis.7.2.4`.

```bash
docker image build -t redis:v7.2.4 -f docker/dockerfile.redis.7.2.4 docker/context
```

## Configure Redis cluster

When running the cluster setup, it is crucial to maintain quorum for cluster stability. So, we are going to provision three nodes, each being a Docker container. We have already stored the configuration for each node in the `source/conf` directory.  You can get the configuration template from Redis' official documentation [^3].

```bash
├── source
│ └── conf
│     ├── master.conf
│     ├── replica.conf
```

## Run Redis cluster

It is considered good practice to store configuration settings in environment variables and inject these variable values from the shell into **Docker Compose** configuration at runtime.

**Step 1:** Store configurations as environment variables in a file.

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

**Step 2:** Create and start the containers for the Redis cluster.

```bash
docker compose -f docker/compose.yml --env-file docker/default.env -p redis-cluster up -d
```

**Step 3:** Set up the Redis cluster by executing the following command. Only run this command once.

By default Redis Cluster requires at least 3 master nodes and 1 replicas per node.  I am going to run both the master and replica services on all three nodes. The master communicates on port 3000, while the replica communicates on port 3001.

```bash
export NODE=node1
docker container exec -it redis-cluster-${NODE}-1 /usr/local/bin/redis-cli --cluster create 10.1.1.11:3000 10.1.1.12:3000 10.1.1.13:3000 10.1.1.11:3001 10.1.1.12:3001 10.1.1.13:3001 --cluster-replicas 1
```

Verify the formation of the cluster by using the command below; this will list the cluster nodes.

```bash
docker container exec -it redis-cluster-${NODE}-1 /usr/local/bin/redis-cli -p 3000 cluster nodes
```


**Step 4:** Stop and destroy the containers for the Redis cluster.

```bash
docker compose -f docker/compose.yml --env-file docker/default.env -p redis-cluster down
```

## Additional helpful commands

```bash
export NODE=node1
docker container exec -it redis-cluster-${NODE}-1 /bin/bash
docker container exec -it redis-cluster-${NODE}-1 /usr/bin/netstat -ntlp
docker container exec -it redis-cluster-${NODE}-1 /usr/bin/ls /opt/redis/data
docker container exec -it redis-cluster-${NODE}-1 /usr/bin/tail -n 200 /opt/redis/log/master.log
```

- Cluster creation.

```bash
docker container exec -it redis-cluster-${NODE}-1 /usr/local/bin/redis-cli --cluster create 10.1.1.11:3000 10.1.1.12:3000 10.1.1.13:3000 10.1.1.11:3001 10.1.1.12:3001 10.1.1.13:3001 --cluster-replicas 1
```

```
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 10.1.1.12:3001 to 10.1.1.11:3000
Adding replica 10.1.1.13:3001 to 10.1.1.12:3000
Adding replica 10.1.1.11:3001 to 10.1.1.13:3000
M: 040651ea90c6da0f24aa14fdad0f7529af19d96a 10.1.1.11:3000
   slots:[0-5460] (5461 slots) master
M: ceb20c026aaf8ebdcd4ee4ef6e20201e944c032a 10.1.1.12:3000
   slots:[5461-10922] (5462 slots) master
M: 77cb7d2f98bf2563fde96f5c12acfc7844492bc4 10.1.1.13:3000
   slots:[10923-16383] (5461 slots) master
S: 94d9d5c5b03e189042616020dd729240eaae673f 10.1.1.11:3001
   replicates 77cb7d2f98bf2563fde96f5c12acfc7844492bc4
S: f50b97b5a757b09364b5fbba9286b28b56c80721 10.1.1.12:3001
   replicates 040651ea90c6da0f24aa14fdad0f7529af19d96a
S: 847b27d0eada904670815717783a1b1441eb391f 10.1.1.13:3001
   replicates ceb20c026aaf8ebdcd4ee4ef6e20201e944c032a
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
.
>>> Performing Cluster Check (using node 10.1.1.11:3000)
M: 040651ea90c6da0f24aa14fdad0f7529af19d96a 10.1.1.11:3000
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: 77cb7d2f98bf2563fde96f5c12acfc7844492bc4 10.1.1.13:3000
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: f50b97b5a757b09364b5fbba9286b28b56c80721 10.1.1.12:3001
   slots: (0 slots) slave
   replicates 040651ea90c6da0f24aa14fdad0f7529af19d96a
S: 94d9d5c5b03e189042616020dd729240eaae673f 10.1.1.11:3001
   slots: (0 slots) slave
   replicates 77cb7d2f98bf2563fde96f5c12acfc7844492bc4
S: 847b27d0eada904670815717783a1b1441eb391f 10.1.1.13:3001
   slots: (0 slots) slave
   replicates ceb20c026aaf8ebdcd4ee4ef6e20201e944c032a
M: ceb20c026aaf8ebdcd4ee4ef6e20201e944c032a 10.1.1.12:3000
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
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
77cb7d2f98bf2563fde96f5c12acfc7844492bc4 10.1.1.13:3000@13000 master - 0 1707810907778 3 connected 10923-16383
f50b97b5a757b09364b5fbba9286b28b56c80721 10.1.1.12:3001@13001 slave 040651ea90c6da0f24aa14fdad0f7529af19d96a 0 1707810908783 1 connected
94d9d5c5b03e189042616020dd729240eaae673f 10.1.1.11:3001@13001 slave 77cb7d2f98bf2563fde96f5c12acfc7844492bc4 0 1707810907000 3 connected
040651ea90c6da0f24aa14fdad0f7529af19d96a 10.1.1.11:3000@13000 myself,master - 0 1707810907000 1 connected 0-5460
847b27d0eada904670815717783a1b1441eb391f 10.1.1.13:3001@13001 slave ceb20c026aaf8ebdcd4ee4ef6e20201e944c032a 0 1707810909788 2 connected
ceb20c026aaf8ebdcd4ee4ef6e20201e944c032a 10.1.1.12:3000@13000 master - 0 1707810905000 2 connected 5461-10922
```

- Get shell access of the Redis server.

```bash
docker container exec -it redis-cluster-${NODE}-1 /usr/local/bin/redis-cli -c -h 10.1.1.11 -p 3000
```

```
# Simple verification.
10.1.1.11:3000> keys *
(empty array)
10.1.1.11:3000> set user:1 sathiyaraj
-> Redirected to slot [10778] located at 10.1.1.12:3000
OK
10.1.1.12:3000> get user:1
"sathiyaraj"
10.1.1.12:3000> exit
```

## References
- [Redis lab documentation](https://developer.redis.com/operate/redis-at-scale/scalability/exercise-1/)

[^1]: [Docker Engine installation](https://docs.docker.com/engine/install)
[^2]: [Redis official documentation](https://redis.io/docs/management/scaling/)
[^3]: [Redis configuration file](https://redis.io/docs/management/config-file/)
