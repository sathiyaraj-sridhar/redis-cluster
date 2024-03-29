This document provides a guide on how to deploy **Redis** in various deployment environment such as DEV, STG, and PRD. However, It does not delve into the related technology in detail. For instance, It explains how to deploy **Redis** in Docker container without providing further information on Docker.

The intended audience for this document includes:
- Software engineers
- Program managers

Let's take a brief look at these learning materials.

```
├── docker
│ ├── compose.yml
│ ├── dockerfile.base.dev
│ └── dockerfile.redis.7.2.4
│── docs
│ └── dev-environment.md
│ └── README.md
├── source
│ └── conf
│     ├── master.conf
│     └── replica.conf
├── supervisor
│ ├── redis.ini
│ └── supervisord.conf
```
- **source:** It includes everything needed for **Redis**, like certificates, configurations, scripts, and more.
- **supervisor:** It includes configuration file for `supervisord` and **Redis** programs.
- **docker:** It includes all the Docker-related materials like Dockerfile, Compose, and the Context directory.
- **docs:** It includes user guides for utilizing these materials.

Let's start.

**Step 1:** Create a directory to manage all my open-source repositories under the `/opt/oss` path. You can also clone it under your custom path.

```bash
sudo mkdir /opt/oss
sudo chown -R $USER /opt/oss
```

**Step 2:** Clone the repository.

```bash
git clone https://github.com/sathiyaraj-sridhar/redis-cluster.git /opt/oss/redis-cluster
```

## Roadmaps

**Development environment (DEV):**
- [x] [Deploying six-node Redis cluster in a Docker Container.](dev-environment.md)
- [ ] Testing Ansible playbooks in a Docker container.
- [ ] Testing Chef cookbooks in a Docker container.

**Staging and QA environment (STG & QA):**
- [ ] Provisioning infrastructure for a Kubernetes setup using Terraform.
- [ ] Deploying a six-node Redis cluster in Kubernetes.
- [ ] Let's understand how to scale Redis cluster in Kubernetes.
- [ ] Let's understand how to backup Redis cluster data in Kubernetes.

**Production environment (PRD):**
- [ ] Provisioning infrastructure on the AWS cloud using Terraform.
- [ ] Deploying a six-node Redis cluster in the AWS cloud using Ansible.
- [ ] Deploying a six-node Redis cluster in the AWS cloud using Chef.
- [ ] Let's understand how to scale Redis cluster in the AWS cloud.
- [ ] Let's understand how to backup Redis cluster data in the AWS cloud.
