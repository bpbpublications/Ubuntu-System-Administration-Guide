
# Up and Running with Kubernetes and Docker

This guide walks you through setting up Docker on Ubuntu, running your first container, managing multi-container applications using Docker Compose, and getting started with Kubernetes for local development.

---

## Table of Contents

1. [Installing Docker on Ubuntu](#1-installing-docker-on-ubuntu)  
2. [Running Your First Docker Container](#2-running-your-first-docker-container)  
3. [Using Docker Compose](#3-using-docker-compose)  
4. [Connecting Services with Docker Compose](#4-connecting-services-with-docker-compose)  
5. [Expanding Docker Compose](#5-expanding-docker-compose)  
6. [Getting Started with Kubernetes](#6-getting-started-with-kubernetes)  
7. [Connecting Two Docker Stacks](#7-connecting-two-docker-stacks)  
8. [Local Development with Docker](#8-local-development-with-docker)  
9. [Deploying Apps on Kubernetes](#9-deploying-apps-on-kubernetes)  

---

## 1. Installing Docker on Ubuntu

### Step 1: Install Dependencies

```bash
sudo apt-get update
sudo apt-get install \
  ca-certificates \
  curl \
  gnupg
```

### Step 2: Add Docker Repository and GPG Key

```bash
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Step 3: Install Docker Engine

```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Step 4: Add User to Docker Group

```bash
sudo usermod -aG docker $USER
```

> Log out and back in for the changes to take effect.

### Step 5: Verify Installation

```bash
docker run hello-world
```

---

## 2. Running Your First Docker Container

To run a Minecraft server in a Docker container:

```bash
docker run -it -e EULA=true -p 25565:25565 --name mc itzg/minecraft-server
```

> Press `CTRL+C` to stop the server.

---

## 3. Using Docker Compose

Docker Compose allows you to define and run multi-container applications.

### Step 1: Create `docker-compose.yaml`

```yaml
version: "3"
services:
  mc:
    image: itzg/minecraft-server
    ports:
      - 25565:25565
    environment:
      EULA: "TRUE"
    tty: true
    stdin_open: true
    restart: unless-stopped
    volumes:
      - ./minecraft-data:/data
```

### Step 2: Start the Minecraft Server

```bash
docker compose up         # Run in foreground
docker compose up -d      # Run in background
docker compose stop       # Stop the server
docker compose ps         # List running services
```

---

## 4. Connecting Services with Docker Compose

Example: WordPress + MySQL

```yaml
services:
  db:
    image: mariadb:10.6.4-focal
    command: '--default-authentication-plugin=mysql_native_password'
    volumes:
      - db_data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
    expose:
      - 3306
    networks:
      - local

  wordpress:
    image: wordpress:latest
    ports:
      - 80:80
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
    networks:
      - local

volumes:
  db_data:

networks:
  local:
    external: true
```

Start the stack:

```bash
docker compose up
```

Visit `http://localhost` in your browser.

---

## 5. Expanding Docker Compose

Add phpMyAdmin to manage MySQL:

```yaml
phpmyadmin:
  image: phpmyadmin
  ports:
    - 8080:80
  environment:
    PMA_ARBITRARY: 1
    PMA_HOST: db
    PMA_USER: wordpress
    PMA_PASSWORD: wordpress
  networks:
    - local
```

Run the updated stack:

```bash
docker compose up
```

Access phpMyAdmin at `http://localhost:8080`.

---

## 6. Getting Started with Kubernetes

Follow [Kubernetes setup guide](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) for installation.

### Quick Deployment Example:

```bash
kubectl create deployment mysql --image=mysql:5.7
kubectl expose deployment mysql --port=3306 --target-port=3306
kubectl get services
```

---

## 7. Connecting Two Docker Stacks

### Step 1: Create a Shared Network

In each stack's `docker-compose.yaml`:

```yaml
networks:
  local:
    external: true
```

### Step 2: Example - Metabase Stack

```yaml
version: '3'
services:
  metabase:
    image: metabase/metabase
    ports:
      - 3000:3000
    networks:
      - local
networks:
  local:
    external: true
```

> Make sure each container uses unique ports.

### Step 3: Start Each Stack

```bash
docker compose up
```

Visit `http://localhost:3000` to configure Metabase.

---

## 8. Local Development with Docker

### Step 1: Setup

```yaml
version: '3'
services:
  dev:
    image: nginx
    volumes:
      - ./code:/usr/share/nginx/html
    ports:
      - 80:80
```

### Step 2: Create HTML File

```html
<!-- ./code/index.html -->
<html>
  <head><title>Docker Test</title></head>
  <body><h2>Hello from Docker</h2></body>
</html>
```

### Step 3: Run

```bash
docker compose up
```

---

## 9. Deploying Apps on Kubernetes

### MySQL Deployment (`mysql.yaml`)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: mysql
spec:
  ports:
    - port: 3306
  selector:
    app: mysql
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  namespace: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - image: mysql:5.7
          name: mysql
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: "myrootpassword"
            - name: MYSQL_DATABASE
              value: wordpress
            - name: MYSQL_USER
              value: wordpress
            - name: MYSQL_PASSWORD
              value: password
          ports:
            - containerPort: 3306
```

```bash
kubectl create namespace mysql
kubectl apply -f mysql.yaml
kubectl get all -n mysql
```

### WordPress Deployment (`wordpress.yaml`)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  namespace: wordpress
spec:
  ports:
    - port: 80
  selector:
    app: wordpress
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  namespace: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
        - image: wordpress:6.2.0-apache
          name: wordpress
          env:
            - name: WORDPRESS_DB_HOST
              value: mysql.mysql.svc
            - name: WORDPRESS_DB_USER
              value: wordpress
            - name: WORDPRESS_DB_NAME
              value: wordpress
            - name: WORDPRESS_DB_PASSWORD
              value: password
          ports:
            - containerPort: 80
```

```bash
kubectl create namespace wordpress
kubectl apply -f wordpress.yaml
kubectl get all -n wordpress
```

### Access WordPress

```bash
minikube service wordpress -n wordpress
```

This opens the WordPress setup page in your browser.

---

## About Kubernetes

Kubernetes automates deployment, scaling, and management of containerized apps. It can run locally using Minikube or in the cloud via managed services.

### Install Minikube

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

### Start

 Minikube

```bash
minikube start             # Docker driver
minikube start --driver=kvm2  # KVM driver
```

### Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

### Verify Cluster

```bash
kubectl get nodes
```

---

You're now ready to build and scale modern apps using Docker and Kubernetes 🚀
```

