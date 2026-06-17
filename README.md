# 🚀 vProfile — Kubernetes Deployment on AWS with kOps

<div align="center">

![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![RabbitMQ](https://img.shields.io/badge/RabbitMQ-FF6600?style=for-the-badge&logo=rabbitmq&logoColor=white)
![Maven](https://img.shields.io/badge/Maven-C71A36?style=for-the-badge&logo=apachemaven&logoColor=white)

**Production-grade deployment of a multi-tier Java web application on a Kubernetes cluster provisioned with kOps on AWS EC2, using S3 for cluster state storage.**

</div>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [Prerequisites](#-prerequisites)
- [Infrastructure Setup with kOps](#-infrastructure-setup-with-kops)
- [Docker — Containerization](#-docker--containerization)
- [Kubernetes — Deployment](#-kubernetes--deployment)
- [Jenkins — CI/CD Pipeline](#-jenkins--cicd-pipeline)
- [Ansible — Configuration Management](#-ansible--configuration-management)
- [Application Properties](#-application-properties)
- [Cleanup](#-cleanup)
- [Contributing](#-contributing)
- [License](#-license)

---

## 🔍 Overview

**vProfile** is a multi-tier Java web application built with the Spring Framework, demonstrating a real-world deployment workflow on Kubernetes. The project covers the full lifecycle — from building and containerizing the application with Docker, to orchestrating it on a Kubernetes cluster provisioned on AWS using **kOps**, with **S3** as the state store and **EC2** instances as worker/master nodes.

### Key Highlights

- ☸️ **Kubernetes orchestration** via kOps on AWS (EC2 + S3)
- 🐳 **Dockerized microservices** — App, DB, Web (Nginx), Memcached, RabbitMQ
- 🔄 **CI/CD pipeline** with Jenkins (build → test → code analysis → artifact publish)
- 🔐 **Kubernetes Secrets** for sensitive credential management
- 💾 **Persistent Volumes** for MySQL data durability
- 🌐 **Ingress Controller** with Nginx for external traffic routing
- 🤖 **Ansible playbooks** for Tomcat setup and artifact deployment

---

## 🏗 Architecture

```
                          ┌──────────────────────────────────────────────┐
                          │              AWS Cloud (kOps)               │
                          │                                              │
   User Request           │   ┌──────────────┐                          │
  ─────────────────────►  │   │   Ingress     │   (Nginx Ingress)       │
                          │   │   Controller  │                          │
                          │   └──────┬───────┘                          │
                          │          │                                    │
                          │          ▼                                    │
                          │   ┌──────────────┐                          │
                          │   │   vproapp     │   (Tomcat + WAR)        │
                          │   │   Service     │   Port: 8080            │
                          │   └──┬───┬───┬───┘                          │
                          │      │   │   │                                │
                          │      ▼   ▼   ▼                                │
                          │   ┌────┐┌────┐┌────────┐                    │
                          │   │ DB ││ MC ││  RMQ   │                    │
                          │   │3306││1121││  5672  │                    │
                          │   └──┬─┘└────┘└────────┘                    │
                          │      │                                        │
                          │   ┌──┴────────┐                              │
                          │   │  PVC (3Gi) │  Persistent Storage        │
                          │   └───────────┘                              │
                          │                                              │
                          │   S3 Bucket ── kOps State Store              │
                          └──────────────────────────────────────────────┘
```

| Component       | Image / Technology         | Port  | Purpose                              |
| --------------- | -------------------------- | ----- | ------------------------------------ |
| **vproapp**     | `vprocontainers/vprofileapp` | 8080  | Tomcat application server (Java WAR) |
| **vprodb**      | `vprocontainers/vprofiledb`  | 3306  | MySQL 8.0 database                   |
| **vprocache01** | `memcached`                  | 11211 | In-memory caching layer              |
| **vpromq01**    | `rabbitmq`                   | 5672  | Message broker                       |
| **vproweb**     | `vprocontainers/vprofileweb` | 80    | Nginx reverse proxy (Docker Compose) |
| **Ingress**     | Nginx Ingress Controller     | 80    | External traffic routing (K8s)       |

---

## 🛠 Tech Stack

| Category                 | Technology                                                          |
| ------------------------ | ------------------------------------------------------------------- |
| **Application**          | Java 17, Spring Framework 6, Spring Security 6, Hibernate 7        |
| **Build Tool**           | Apache Maven                                                        |
| **Database**             | MySQL 8.0.33                                                        |
| **Caching**              | Memcached                                                           |
| **Message Broker**       | RabbitMQ                                                            |
| **Search**               | Elasticsearch 7.10.2                                                |
| **Containerization**     | Docker, Docker Compose                                              |
| **Orchestration**        | Kubernetes (kOps on AWS)                                            |
| **Cloud Infrastructure** | AWS EC2 (compute), S3 (kOps state store), Route53 (DNS — optional) |
| **CI/CD**                | Jenkins (Maven build, SonarQube, Nexus)                             |
| **Config Management**    | Ansible                                                             |
| **Web Server / Proxy**   | Nginx                                                               |

---

## 📁 Project Structure

```
vprofile-project-kubeapp/
│
├── src/                           # Java source code (Spring MVC web app)
│   ├── main/
│   │   ├── java/                  # Application source (controllers, services, models)
│   │   ├── resources/
│   │   │   ├── application.properties   # App config (DB, cache, MQ connections)
│   │   │   ├── db_backup.sql            # Database schema & seed data
│   │   │   └── logback.xml              # Logging configuration
│   │   └── webapp/                # JSP views, static assets, WEB-INF config
│   └── test/                      # Unit & integration tests
│
├── Docker-files/                  # Dockerfiles for each service
│   ├── app/
│   │   ├── Dockerfile             # Tomcat app image (single-stage)
│   │   └── multistage/
│   │       └── Dockerfile         # Multi-stage build (clone → build → deploy)
│   ├── db/
│   │   ├── Dockerfile             # MySQL image with schema initialization
│   │   └── db_backup.sql          # SQL dump loaded on container startup
│   └── web/
│       ├── Dockerfile             # Nginx reverse proxy image
│       └── nginvproapp.conf       # Nginx upstream config → vproapp:8080
│
├── kubedefs/                      # Kubernetes manifest files
│   ├── secret.yaml                # K8s Secret (DB & RabbitMQ passwords)
│   ├── dbpvc.yaml                 # PersistentVolumeClaim (3Gi for MySQL)
│   ├── dbdeploy.yaml              # MySQL Deployment + init container
│   ├── dbservice.yaml             # MySQL ClusterIP Service
│   ├── mcdep.yaml                 # Memcached Deployment
│   ├── mcservice.yaml             # Memcached ClusterIP Service
│   ├── rmqdeploy.yaml             # RabbitMQ Deployment
│   ├── rmqservice.yaml            # RabbitMQ ClusterIP Service
│   ├── appdeploy.yaml             # vproapp Deployment + init containers
│   ├── appservice.yaml            # vproapp ClusterIP Service
│   └── appingress.yaml            # Nginx Ingress (host-based routing)
│
├── ansible/                       # Ansible automation
│   ├── ansible.cfg                # Ansible configuration
│   ├── site.yml                   # Master playbook
│   ├── tomcat_setup.yml           # Tomcat 8 installation & service setup
│   ├── vpro-app-setup.yml         # Artifact download from Nexus & deploy
│   └── templates/                 # Jinja2 templates
│       ├── application.j2         # Application properties template
│       ├── epel6-svcfile.j2       # CentOS 6 systemd service file
│       ├── epel7-svcfile.j2       # CentOS 7 systemd service file
│       ├── ubuntu14_15-svcfile.j2 # Ubuntu 14/15 init script
│       └── ubuntu16-svcfile.j2    # Ubuntu 16+ systemd service file
│
├── docker-compose.yml             # Compose file to run all services locally
├── Jenkinsfile                    # CI/CD pipeline definition
└── pom.xml                        # Maven project configuration
```

---

## ✅ Prerequisites

Ensure you have the following installed and configured before proceeding:

| Tool            | Version       | Purpose                            |
| --------------- | ------------- | ---------------------------------- |
| **AWS CLI**     | v2+           | AWS resource management            |
| **kOps**        | v1.28+        | Kubernetes cluster provisioning     |
| **kubectl**     | v1.28+        | Kubernetes cluster interaction      |
| **Docker**      | 20.10+        | Building container images           |
| **Maven**       | 3.9+          | Building the Java application       |
| **JDK**         | 17+           | Java compilation                    |
| **Ansible**     | 2.9+ (optional) | Configuration management         |
| **Jenkins**     | 2.x (optional)  | CI/CD pipeline                   |

### AWS Prerequisites

- An **AWS account** with IAM user having permissions for EC2, S3, VPC, Route53, and IAM
- An **S3 bucket** for kOps state storage
- A **domain name** (optional, for Ingress — e.g., `vprofile.yourdomain.com`)
- An **SSH key pair** for EC2 access

---

## ☸️ Infrastructure Setup with kOps

### 1. Create the S3 State Store

```bash
aws s3 mb s3://vprofile-kops-state --region us-east-1
```

### 2. Export Environment Variables

```bash
export NAME=vprofile.k8s.local
export KOPS_STATE_STORE=s3://vprofile-kops-state
```

### 3. Generate SSH Keys

```bash
ssh-keygen -f ~/.ssh/kops_rsa
```

### 4. Create the Kubernetes Cluster

```bash
kops create cluster \
  --name=$NAME \
  --state=$KOPS_STATE_STORE \
  --zones=us-east-1a,us-east-1b \
  --node-count=2 \
  --node-size=t3.small \
  --master-size=t3.medium \
  --dns-zone=vprofile.k8s.local \
  --node-volume-size=8 \
  --master-volume-size=8 \
  --ssh-public-key=~/.ssh/kops_rsa.pub
```

### 5. Apply the Cluster Configuration

```bash
kops update cluster --name $NAME --yes --admin
```

### 6. Validate the Cluster

```bash
# Wait a few minutes for instances to initialize, then:
kops validate cluster --wait 10m
```

---

## 🐳 Docker — Containerization

The project provides Dockerfiles for three custom services and uses official images for Memcached and RabbitMQ.

### Build Images

```bash
# Build the application WAR first
mvn clean install -DskipTests

# Build all images with Docker Compose
docker-compose build
```

### Run Locally with Docker Compose

```bash
docker-compose up -d
```

This starts the full stack:

| Service         | Container    | Host Port | Description            |
| --------------- | ------------ | --------- | ---------------------- |
| **vprodb**      | `vprodb`     | 3306      | MySQL with seeded data |
| **vprocache01** | —            | 11211     | Memcached              |
| **vpromq01**    | —            | 5672      | RabbitMQ               |
| **vproapp**     | `vproapp`    | 8080      | Tomcat application     |
| **vproweb**     | `vproweb`    | 80        | Nginx reverse proxy    |

### Multi-Stage Build (Alternative)

A multi-stage Dockerfile is available at `Docker-files/app/multistage/Dockerfile` that clones the source, builds the WAR, and creates the final Tomcat image — all in a single `docker build` command:

```bash
docker build -t vprocontainers/vprofileapp -f Docker-files/app/multistage/Dockerfile .
```

---

## ☸️ Kubernetes — Deployment

All Kubernetes manifests are in the `kubedefs/` directory. Deploy them in the following order to respect service dependencies.

### 1. Create Secrets

```bash
kubectl apply -f kubedefs/secret.yaml
```

> **Secrets stored (base64-encoded):**
> - `db-pass` — MySQL root password
> - `rmq-pass` — RabbitMQ password

### 2. Deploy Database (MySQL)

```bash
kubectl apply -f kubedefs/dbpvc.yaml        # PersistentVolumeClaim (3Gi)
kubectl apply -f kubedefs/dbdeploy.yaml      # MySQL Deployment
kubectl apply -f kubedefs/dbservice.yaml     # ClusterIP Service (port 3306)
```

> The DB deployment includes an **init container** that cleans up `lost+found` in the PVC mount before MySQL starts.

### 3. Deploy Caching Layer (Memcached)

```bash
kubectl apply -f kubedefs/mcdep.yaml         # Memcached Deployment
kubectl apply -f kubedefs/mcservice.yaml     # ClusterIP Service (port 11211)
```

### 4. Deploy Message Broker (RabbitMQ)

```bash
kubectl apply -f kubedefs/rmqdeploy.yaml     # RabbitMQ Deployment
kubectl apply -f kubedefs/rmqservice.yaml    # ClusterIP Service (port 5672)
```

### 5. Deploy Application (Tomcat)

```bash
kubectl apply -f kubedefs/appdeploy.yaml     # App Deployment
kubectl apply -f kubedefs/appservice.yaml    # ClusterIP Service (port 8080)
```

> The app deployment includes **init containers** that wait for MySQL (`vprodb`) and Memcached (`vprocache01`) DNS resolution before starting the application container.

### 6. Deploy Ingress

```bash
kubectl apply -f kubedefs/appingress.yaml    # Nginx Ingress rule
```

> **Note:** Update the `host` field in `appingress.yaml` to match your domain. The default is `vprofile.hkhinfoteck.xyz`.

### Deploy Everything at Once

```bash
kubectl apply -f kubedefs/
```

### Verify Deployment

```bash
kubectl get all
kubectl get ingress
kubectl get pvc
kubectl get secrets
```

---

## 🔄 Jenkins — CI/CD Pipeline

The [Jenkinsfile](Jenkinsfile) defines a complete CI/CD pipeline with the following stages:

```
BUILD ──► UNIT TEST ──► INTEGRATION TEST ──► CHECKSTYLE ──► SONARQUBE ──► NEXUS PUBLISH
```

| Stage                     | Description                                                  |
| ------------------------- | ------------------------------------------------------------ |
| **BUILD**                 | `mvn clean install -DskipTests` — Compiles and packages WAR |
| **UNIT TEST**             | `mvn test` — Runs unit tests                                |
| **INTEGRATION TEST**      | `mvn verify -DskipUnitTests` — Runs integration tests       |
| **CHECKSTYLE**            | `mvn checkstyle:checkstyle` — Static code analysis          |
| **SONARQUBE ANALYSIS**    | Runs SonarQube scanner with quality gate enforcement         |
| **NEXUS PUBLISH**         | Uploads the WAR artifact to Nexus Repository Manager         |

### Jenkins Environment Variables

| Variable                | Value                 |
| ----------------------- | --------------------- |
| `NEXUS_URL`             | `172.31.40.209:8081`  |
| `NEXUS_REPOSITORY`      | `vprofile-release`    |
| `NEXUS_CREDENTIAL_ID`   | `nexuslogin`          |
| `ARTVERSION`            | `${env.BUILD_ID}`     |

> ⚠️ **Update** the `NEXUS_URL` and other environment variables in the Jenkinsfile to match your infrastructure.

---

## 🤖 Ansible — Configuration Management

Ansible playbooks in the `ansible/` directory automate Tomcat installation and application deployment on target servers.

### Playbooks

| Playbook                | Purpose                                                            |
| ----------------------- | ------------------------------------------------------------------ |
| `site.yml`              | Master playbook — imports `tomcat_setup.yml` + `vpro-app-setup.yml` |
| `tomcat_setup.yml`      | Installs JDK, downloads & configures Tomcat 8, sets up systemd     |
| `vpro-app-setup.yml`    | Downloads WAR from Nexus, backs up current deployment, deploys new |

### Run Playbooks

```bash
cd ansible/
ansible-playbook -i inventory site.yml
```

> **Supports:** CentOS 6/7 and Ubuntu 14/15/16/18 with automatic OS detection for service configuration.

---

## ⚙️ Application Properties

The application connects to backend services using **Kubernetes service DNS names** defined in `src/main/resources/application.properties`:

```properties
# Database
jdbc.url=jdbc:mysql://vprodb:3306/accounts
jdbc.username=root
jdbc.password=vprodbpass

# Memcached
memcached.active.host=vprocache01
memcached.active.port=11211

# RabbitMQ
rabbitmq.address=vpromq01
rabbitmq.port=5672
rabbitmq.username=guest
rabbitmq.password=guest

# Elasticsearch
elasticsearch.host=localhost
elasticsearch.port=9300
```

> These DNS names (`vprodb`, `vprocache01`, `vpromq01`) correspond to the Kubernetes Service names defined in `kubedefs/`.

---

## 🧹 Cleanup

### Delete Kubernetes Resources

```bash
kubectl delete -f kubedefs/
```

### Destroy the kOps Cluster

```bash
kops delete cluster --name=$NAME --yes
```

### Remove S3 State Store

```bash
aws s3 rb s3://vprofile-kops-state --force
```

### Stop Docker Compose

```bash
docker-compose down -v
```

---

## 🤝 Contributing

1. **Fork** the repository
2. **Create** a feature branch (`git checkout -b feature/amazing-feature`)
3. **Commit** your changes (`git commit -m 'Add amazing feature'`)
4. **Push** to the branch (`git push origin feature/amazing-feature`)
5. **Open** a Pull Request

---

## 📄 License

This project is open source and available for educational purposes.

---

<div align="center">

**⭐ If you found this project helpful, give it a star! ⭐**

</div>
