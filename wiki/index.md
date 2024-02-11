This document provides a guide on how to deploy **Redis** in various deployment environment such as DEV, STG, and PRD. However, It does not delve into the related technology in detail. For instance, It explains how to deploy **Redis** in Docker container without providing further information on Docker.

The intended audience for this document includes:
- software engineers
- program managers

Let's take a brief look at these learning materials.

```
├── README.md
├── docker
│ ├── compose.yml
│ ├── dockerfile.base.dev
│ └── dockerfile.redis.7.2.4
├── source
│ └── conf
│     ├── master.conf
│     └── replica.conf
├── supervisor
│ ├── redis.ini
│ └── supervisord.conf
└── wiki
    └── dev-environment.md
    └── index.md
```
- **source:** It includes everything needed for **Redis**, like certificates, configurations, scripts, and more.
- **supervisor:** It includes configuration file for `supervisord` and **Redis** programs.
- **docker:** It includes all the Docker-related materials like Dockerfile, Compose, and the Context directory.
- **wiki:** It includes user guides for utilizing these materials.

Let's download our repository.

**Step 1:** Create a directory to manage open-source software (OSS).

```bash
sudo mkdir /opt/oss
sudo chown -R $USER /opt/oss
```

**Step 2:** Clone our `redis-cluster` repository.

```bash
git clone https://github.com/sathiyaraj-sridhar/redis-cluster.git /opt/oss/redis-cluster
```

## Guides

**Development environment (DEV):**
1. [Running six-node Redis cluster in a Docker Container](dev-environment.md)
