
# Automating Everything: Use DevOps Practices and Automate Your Skills

## Chapter Overview
In this guide, we automate tasks and services using tools such as Bash, Ansible, Docker, Terraform, and Kubernetes. This enables us to execute tasks in a repeatable and consistent manner, simplifying deployments and server management. We'll also explore setting up a CI/CD pipeline for building, testing, and deploying applications.

### Table of Contents
1. [Basic Bash](#basic-bash)
2. [Automate Tasks with Ansible](#automate-tasks-with-ansible)
3. [Run Host Command from Docker](#run-host-command-from-docker)
4. [Pipeline Step One: Build and Push Docker Images](#pipeline-step-one-build-and-push-docker-images)
5. [Pipeline Step Two: Deploy with Terraform and Kubernetes](#pipeline-step-two-deploy-with-terraform-and-kubernetes)
6. [CI/CD Pipeline Setup](#cicd-pipeline-setup)
7. [Conclusion](#conclusion)

---

## Basic Bash

### Introduction
Bash scripts allow us to automate repetitive tasks. Let's begin with a script that installs Docker.

### Step 1: Create a Bash Script to Install Docker
Create a file named `install_docker.sh`:

```bash
#!/bin/bash

echo "Installing Docker..."
apt-get update

echo "Installing required packages"
apt-get install -y apt-transport-https ca-certificates curl gnupg lsb-release

echo "Getting Docker GPG key"
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "Setting up Docker repository"
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

echo "Installing Docker packages"
apt-get update
apt-get install -y docker-ce docker-ce-cli containerd.io
```

### Step 2: Make the Script Executable and Run It

```bash
chmod +x install_docker.sh
./install_docker.sh
```

### Step 3: System Status Report Script

```bash
#!/bin/bash

echo "Disk Usage:"
df -h
du -h --max-depth=0 /
du -h --max-depth=0 /var

echo "Inode Usage:"
for i in $(find . -type d); do echo "$(ls -a "$i" | wc -l) $i"; done | sort -n

echo "Network:"
ip a
ip r
cat /etc/resolv.conf

echo "Memory:"
free -h
```

---

## Automate Tasks with Ansible

### Introduction
Ansible automates system configurations and app deployments using SSH, without requiring agents.

### Step 1: Dockerfile for Ansible

```dockerfile
FROM ubuntu:latest
RUN apt update && apt install -y software-properties-common
RUN add-apt-repository --yes --update ppa:ansible/ansible
RUN apt install -y ansible
```

### Step 2: Docker Compose for Ansible

```yaml
services:
  ansible:
    build: .
    volumes:
      - ./playbooks:/opt/playbooks
      - ./files:/opt/files
      - ./hosts:/etc/ansible/
      - ./ssh:/root/.ssh
    command: tail -f /dev/null
```

### Step 3: Define Inventory Hosts

`hosts` file:

```yaml
all:
  vars:
    ansible_connection: ssh
    ansible_user: matte
  hosts:
    192.168.122.133:

docker:
  hosts:
    192.168.122.133:
apache:
  hosts:
    192.168.122.133:
```

### Step 4: Create Playbook to Install Docker

`install-docker.yaml`:

```yaml
- name: Install Docker
  hosts: all
  become: yes
  tasks:
    - name: Create directory
      file:
        path: /opt/files/
        state: directory

    - name: Copy Docker install script
      copy:
        src: /opt/files/install_docker.sh
        dest: /opt/files/install_docker.sh
        mode: '0755'

    - name: Upgrade all apt packages
      apt:
        upgrade: dist
        force_apt_get: yes
```

### Step 5: Run Ansible

```bash
docker compose build
docker compose run ansible /bin/bash
ansible-playbook --ask-pass --ask-become-pass /opt/playbooks/install-docker.yaml
```

---

## Run Host Command from Docker

### Introduction
You can use Docker to manipulate or read host files by mounting directories.

### Step 1: Minimal Dockerfile

```dockerfile
FROM ubuntu:latest
```

### Step 2: Docker Compose to Access Host `/etc`

```yaml
services:
  host-update:
    build: .
    volumes:
      - /etc:/mnt/etc
```

### Step 3: Example Command

```bash
docker compose run host-update cat /mnt/etc/passwd
```

---

## Pipeline Step One: Build and Push Docker Images

### Step 1: Docker Compose for Nginx

```yaml
services:
  nginx:
    image: nginx:latest
    volumes:
      - ./html:/usr/share/nginx/html
    ports:
      - "80:80"
```

### Step 2: HTML Content

`html/index.html`:

```html
<html>
  <head><title>Home</title></head>
  <body>
    <h1>Home</h1>
    <p>Home page</p>
  </body>
</html>
```

### Step 3: Script to Build and Push

`build_and_push.sh`:

```bash
#!/bin/bash
VERSION=$1
docker build -t mattiashem/ubuntu-static:$VERSION .
docker push mattiashem/ubuntu-static:$VERSION
```

```bash
./build_and_push.sh v1.0
```

---

## Pipeline Step Two: Deploy with Terraform and Kubernetes

### Step 1: Dockerfile for Terraform + Ansible

```dockerfile
FROM ubuntu:latest
RUN apt update && apt install -y wget unzip software-properties-common gnupg
RUN add-apt-repository --yes --update ppa:ansible/ansible && apt install -y ansible
WORKDIR /opt
RUN wget https://releases.hashicorp.com/terraform/1.6.5/terraform_1.6.5_linux_amd64.zip && \
    unzip terraform_1.6.5_linux_amd64.zip && \
    mv terraform /usr/local/bin/terraform && \
    rm terraform_1.6.5_linux_amd64.zip
RUN terraform --version
```

### Step 2: Terraform Configuration

`terraform.tf`:

```hcl
terraform {
  backend "local" {
    workspace_dir = "/opt/terraform/state/terraform.tfstate.d"
  }
}
```

`deployments.tf`:

```hcl
resource "kubernetes_deployment" "static" {
  metadata {
    name = "static-data"
    labels = {
      test = "static"
    }
  }

  spec {
    replicas = 3
    selector {
      match_labels = {
        app = "static"
      }
    }

    template {
      metadata {
        labels = {
          app = "static"
        }
      }

      spec {
        container {
          image = "mattiashem/ubuntu-static:${var.version}"
          name  = "static"
          resources {
            limits {
              cpu    = "0.5"
              memory = "512Mi"
            }
            requests {
              cpu    = "250m"
              memory = "50Mi"
            }
          }
        }
      }
    }
  }
}
```

### Step 3: Deploy with Terraform

```bash
terraform init
terraform plan
terraform apply
```

```terraform
Plan: 1 to add, 0 to change, 1 to destroy.
...
Apply complete! Resources: 1 added, 0 changed, 1 destroyed.
```

### Step 4: Verify Deployment

```bash
kubectl get pods
```

```bash
NAME                           READY   STATUS    RESTARTS   AGE
static-data-xxx-yyyy           1/1     Running   0          12s
...
```

---

## CI/CD Pipeline Setup

### Step 1: Use GitHub Actions or Jenkins

Set up a pipeline that:
1. Builds and tags the Docker image.
2. Pushes it to Docker Hub.
3. Applies the Terraform deployment with the new tag.

---

## Conclusion

By combining Bash, Ansible, Docker, Terraform, and Kubernetes, you can automate infrastructure and deployment processes efficiently. Integrating CI/CD ensures changes are tested and deployed consistently.
```

Let me know if you'd like me to export this to a file or split this up into multiple chapters for easier reading!