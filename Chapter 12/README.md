
# Set Up a Kubernetes Cluster to Host Your Private Blog

## Introduction

In this guide, we will set up a Kubernetes cluster on a set of Ubuntu servers. We'll configure the necessary components, services for monitoring, logging, ingress/load balancing, and deploy a WordPress workload on the cluster. Kubernetes can be set up on various cloud platforms, and this guide also applies to those environments.

## Table of Contents
1. [Installing Kubernetes on Ubuntu](#installing-kubernetes-on-ubuntu)
2. [Setting up the Kubernetes Cluster](#setting-up-the-kubernetes-cluster)
3. [Deploying Kubernetes Base Service](#deploying-kubernetes-base-service)
4. [Installing Helm](#installing-helm)
5. [Adding Storage](#adding-storage)
6. [Monitoring the Cluster](#monitoring-the-cluster)
7. [Common kubectl Commands](#common-kubectl-commands)

---

## Installing Kubernetes on Ubuntu

We'll use **Kubeadm**, a tool provided by Kubernetes, to set up the cluster. We'll install it on both the master and worker nodes.

### Prerequisites

Run the following commands on both the master and worker nodes to install necessary packages.

```bash
# Update system packages
echo "First lets update this box"
apt-get update
apt-get upgrade -y

# Install Docker and dependencies
echo "Lets get docker"
apt-get install apt-transport-https ca-certificates curl gnupg lsb-release -y

# Setup cri-o container runtime
echo "Setup cri-o"
export OS=xUbuntu_22.04
export CRIO_VERSION=1.28
echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /"| sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
echo "deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$CRIO_VERSION/$OS/ /"|sudo tee /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$CRIO_VERSION.list

# Install CRI-O
curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$CRIO_VERSION/$OS/Release.key | sudo apt-key add -
apt update
sudo apt install cri-o cri-o-runc -y

# Configure Kubernetes modules
echo "Setup kdeadm"
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sudo sysctl --system

# Install Kubernetes components
apt-get update
apt-get install -y apt-transport-https ca-certificates curl
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl

# Disable swap
swapoff -a
```

### Install `kubectl` CLI

Install the `kubectl` command line interface on the master node.

```bash
apt-get install -y kubectl
```

### Turn Off Swap Permanently

To ensure Kubernetes works correctly, we need to disable swap.

```bash
swapoff -a
# Comment out the swap line in /etc/fstab
vi /etc/fstab
```

---

## Setting up the Kubernetes Cluster

### Initialize the Master Node

Use `kubeadm` to initialize the master node:

```bash
kubeadm init --cri-socket /var/run/crio/crio.sock --pod-network-cidr=10.244.0.0/16
```

This will initialize the master node. After the master is initialized, follow the instructions to configure access to the Kubernetes cluster.

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Join Worker Node to Cluster

On the worker node, run the following command to join it to the cluster:

```bash
kubeadm join 192.168.122.162:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

### Add a Network Plugin

After initializing the master node, we need to add a network plugin like Flannel:

```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

Verify that all pods are running correctly:

```bash
kubectl get pods -A
```

---

## Deploying Kubernetes Base Service

Now, let’s add workloads to our cluster using **Helm**. 

---

## Installing Helm

Helm is the package manager for Kubernetes. To install Helm, run:

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

---

## Adding Storage

Next, let's add a simple storage solution to our Kubernetes cluster using **OpenEBS**.

### Install OpenEBS

Run the following command to install OpenEBS:

```bash
kubectl apply -f https://openebs.github.io/charts/openebs-operator.yaml
```

### Set OpenEBS as Default Storage Class

```bash
kubectl patch storageclass openebs-hostpath -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

---

## Monitoring the Cluster

To monitor your cluster, we'll use **Prometheus** and **Grafana**.

### Install Prometheus

Add the Prometheus Helm repo and install it:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install monitoring prometheus-community/kube-prometheus-stack
```

### Verify Prometheus Installation

Check the status of Prometheus and other pods:

```bash
kubectl get pods -A
```

---

## Common kubectl Commands

- To get the nodes in the cluster:
  ```bash
  kubectl get nodes
  ```

- To see all pods across all namespaces:
  ```bash
  kubectl get pods -A
  ```

- To see the logs of a specific pod:
  ```bash
  kubectl logs <pod-name>
  ```

- To describe a pod:
  ```bash
  kubectl describe pod <pod-name>
  ```

---

# How to Set Up Ingress, Load Balancer, Logs, and WordPress in Kubernetes

This guide will walk you through setting up the necessary components for your Kubernetes cluster, including the ingress controller, load balancer, logging system, and the deployment of WordPress with a MySQL database.

---

## 1. Set Up Ingress Controller with Traefik

### Step 1: Add the Traefik Helm Repository

To install the Traefik ingress controller using Helm, follow these steps:

```bash
root@k8smaster:~# helm repo add traefik https://traefik.github.io/charts
"traefik" has been added to your repositories
```

### Step 2: Update the Helm Repositories

```bash
root@k8smaster:~# helm repo update
```

### Step 3: Install Traefik

```bash
root@k8smaster:~# helm install traefik traefik/traefik
```

This will deploy Traefik to the default namespace.

### Step 4: Verify Traefik Pods are Running

```bash
root@k8smaster:~# kubectl get pods
```

You should see the `traefik` pod running.

---

## 2. Set Up Load Balancer with MetalLB

### Step 1: Install MetalLB

MetalLB provides an external load balancer for your Kubernetes cluster. Install MetalLB with the following command:

```bash
root@k8smaster:~# kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.12/config/manifests/metallb-native.yaml
```

### Step 2: Verify MetalLB Pods are Running

```bash
root@k8smaster:~# kubectl get pods -A
```

You should see MetalLB components (`controller` and `speaker`) running.

---

## 3. Set Up Logs with Loki

### Step 1: Create `values.yaml` for Loki

Create a new file named `values.yaml` and add the following configuration:

```yaml
loki:
  commonConfig:
    replication_factor: 1
  storage:
    type: 'filesystem'
singleBinary:
  replicas: 1
```

### Step 2: Add the Grafana Helm Repository

```bash
root@k8smaster:~/loki# helm repo add grafana https://grafana.github.io/helm-charts
"grafana" has been added to your repositories
```

### Step 3: Update Helm Repositories

```bash
root@k8smaster:~/loki# helm repo update
```

### Step 4: Install Loki

```bash
root@k8smaster:~/loki# helm install --values values.yaml loki grafana/loki-stack
```

### Step 5: Verify Loki Pods are Running

```bash
root@k8smaster:~/loki# kubectl get pods -A
```

You should see Loki and Promtail pods running.

---

## 4. Install WordPress into Kubernetes

Now, we'll deploy WordPress along with a MySQL database.

### Step 1: Create MySQL Deployment

Create a file named `mysql.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql-pod
  replicas: 1
  template:
    metadata:
      labels:
        app: mysql-pod  
    spec:
      containers:
        - image: mysql:5.6
          imagePullPolicy: Always
          name: mysql-pod
          args: ["--default-authentication-plugin=mysql_native_password"]
          env:
            - name: MYSQL_USER
              value: mysql
            - name: MYSQL_PASSWORD
              value: 'password'
            - name: MYSQL_ROOT_PASSWORD
              value: rootpassword
            - name: MYSQL_DATABASE
              value: wordpress
          ports:
            - containerPort: 3306
              name: sql
          resources:
            requests:
              cpu: 100m
          volumeMounts:
            - name: mysql-persistent-storage
              mountPath: /var/lib/mysql
      volumes:
        - name: mysql-persistent-storage
          persistentVolumeClaim:
            claimName: mysql-disk
---
apiVersion: v1
kind: Service
metadata:
  name: mysql
  labels:
    app: mysql-pod
spec:
  type: ClusterIP
  ports:
    - port: 3306
      targetPort: 3306
      protocol: TCP
  selector:
    app: mysql-pod
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mysql-disk
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

### Step 2: Apply MySQL Manifest

Apply the MySQL manifest to your Kubernetes cluster:

```bash
kubectl apply -f mysql.yaml
```

### Step 3: Verify MySQL Pods are Running

```bash
kubectl get pods
```

You should see the MySQL pod running.

### Step 4: Create WordPress Deployment

Create a file named `wordpress.yaml` with the following content:

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: wordpress-storage
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
---
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress-pod
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
      name: http
  selector:
    app: wordpress-pod
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wordpress-pod
  template:
    metadata:
      labels:
        app: wordpress-pod  
    spec:
      containers:
        - image: wordpress
          imagePullPolicy: Always
          name: wordpress-pod
          env:
            - name: WORDPRESS_DB_HOST
              value: mysql
            - name: WORDPRESS_DB_PASSWORD
              value: password
            - name: WORDPRESS_DB_USER
              value: mysql
            - name: WORDPRESS_DB_NAME  
              value: wordpress
          ports:
            - containerPort: 80
              name: www
          resources:
            requests:
              memory: "64Mi"
              cpu: "250m"
            limits:
              memory: "256Mi"
              cpu: "500m"
          volumeMounts:
            - name: wordpress-storage
              mountPath: /var/www/html
      securityContext:
        fsGroup: 200
      volumes:
        - name: wordpress-storage
          persistentVolumeClaim:
            claimName: wordpress-storage
```

### Step 5: Apply WordPress Manifest

Apply the WordPress manifest to your Kubernetes cluster:

```bash
kubectl apply -f wordpress.yaml
```

### Step 6: Verify WordPress Pods are Running

```bash
kubectl get pods
```

You should see the WordPress pod running.

---
Here’s a How-to guide in markdown format, extracting commands and headings from the provided text:

```markdown
# How to Access WordPress and Set Up Ingress in Kubernetes

## Access WordPress

To access WordPress, you can use two different methods:

1. **Using NodePort**:
    - Modify the service in your WordPress YAML file to expose a `NodePort`.
    - Here’s how the updated service configuration should look:

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: wordpress
      labels:
        app: wordpress-pod
    spec:
      type: NodePort
      ports:
        - port: 80
          targetPort: 80
          protocol: TCP
          name: http
      selector:
        app: wordpress-pod
    ```

    Apply the changes to your WordPress deployment:

    ```bash
    kubectl apply -f wordpress.yaml
    ```

    Find the port assigned to your WordPress service:

    ```bash
    kubectl get service wordpress
    ```

    The output will show the NodePort, for example:

    ```bash
    wordpress   NodePort   10.96.52.119   <none>        80:31360/TCP   5m18s
    ```

    Now you can access your WordPress installation by navigating to:

    ```
    http://<IP_OF_WORKER>:31360
    ```

    Example:

    ```
    http://192.168.122.26:31360/
    ```

2. **Using Ingress with MetalLB**:
    - Set up MetalLB to provide an external IP.
    - Create a MetalLB configuration file `metallb.yaml`:

    ```yaml
    apiVersion: metallb.io/v1beta1
    kind: IPAddressPool
    metadata:
      name: pool
      namespace: metallb-system
    spec:
      addresses:
      - 192.168.122.133/32
    ---
    apiVersion: metallb.io/v1beta1
    kind: L2Advertisement
    metadata:
      name: pool
      namespace: metallb-system
    spec:
      ipAddressPools:
      - pool
    ```

    Apply the configuration:

    ```bash
    kubectl apply -f metallb.yaml
    ```

    Add the IP to the Traefik service:

    ```bash
    kubectl annotate service traefik metallb.universe.tf/address-pool=pool
    ```

    Check the service:

    ```bash
    kubectl get svc traefik
    ```

    You should see the external IP assigned to Traefik.

    Create an Ingress for WordPress with `wordpress-ingress.yaml`:

    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      annotations:
        kubernetes.io/ingress.class: traefik
      name: wordpress
    spec:
      rules:
      - host: "wordpress.home.lan"
        http:
          paths:
          - backend:
              service:
                name: wordpress
                port:
                  number: 80
            path: /
            pathType: Prefix
    ```

    Update `/etc/hosts` on your machine to point to the IP:

    ```
    192.168.122.133 wordpress.home.lan
    ```

    Access WordPress in your browser via:

    ```
    http://wordpress.home.lan
    ```

---

## Monitoring Kubernetes Cluster with Grafana

### Create Ingress for Grafana

Create the `grafana-ingress.yaml` file:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: traefik
  name: grafana
spec:
  rules:
  - host: "grafana.home.lan"
    http:
      paths:
      - backend:
          service:
            name: monitoring-grafana
            port:
              number: 80
        path: /
        pathType: Prefix
```

Update your `/etc/hosts` file to point to the correct IP:

```
192.168.122.133 grafana.home.lan
```

Now, access Grafana via:

```
http://grafana.home.lan
```

### Monitoring with Prometheus

To get Prometheus URL for Grafana integration, run:

```bash
kubectl get svc
```

The output will give you the URL:

```
monitoring-kube-prometheus-prometheus     ClusterIP      10.111.200.89    <none>            9090/TCP,8080/TCP              2d14h
```

Use the following URL to connect to Prometheus in Grafana:

```
http://monitoring-kube-prometheus-prometheus:9090
```

---

## Secrets Management in Kubernetes

To view and decode a secret, use the following command:

```bash
kubectl edit secret monitoring-grafana
```

It will show something like:

```yaml
apiVersion: v1
data:
  admin-password: cHJvbS1vcGVyYXRvcg==
  admin-user: YWRtaW4=
```

To decode the base64 encoded values:

```bash
echo cHJvbS1vcGVyYXRvcg== | base64 -d
```

The output will be:

```
prom-operator
```

The password for the admin user in Grafana is `prom-operator`.

---

## Useful `kubectl` Commands

### General Cluster Commands

- List all pods across all namespaces:

    ```bash
    kubectl get pods -A
    ```

- List all services:

    ```bash
    kubectl get svc -A
    ```

- List all deployments:

    ```bash
    kubectl get deployments -A
    ```

- List all ingress resources:

    ```bash
    kubectl get ingress -A
    ```

### Getting Details of a Pod

- Describe a specific pod:

    ```bash
    kubectl describe pod <POD_NAME>
    ```

- View logs for a pod:

    ```bash
    kubectl logs -f <POD_NAME>
    ```

- Edit a pod:

    ```bash
    kubectl edit pod <POD_NAME>
    ```

### Troubleshooting Commands

- Find and list namespaces:

    ```bash
    kubectl get ns
    ```

- Get pod details within a specific namespace:

    ```bash
    kubectl get pods -n <namespace_name>
    ```

---

### Conclusion

This guide shows you how to deploy and access WordPress in a Kubernetes environment, set up Grafana for monitoring, and manage secrets and troubleshooting commands effectively.