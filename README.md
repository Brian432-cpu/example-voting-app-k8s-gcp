# Example Voting App

A simple distributed application running across multiple Docker containers.

## Getting started

Download [Docker Desktop](https://www.docker.com/products/docker-desktop) for Mac or Windows. [Docker Compose](https://docs.docker.com/compose) will be automatically installed. On Linux, make sure you have the latest version of [Compose](https://docs.docker.com/compose/install/).

This solution uses Python, Node.js, .NET, with Redis for messaging and Postgres for storage.

Run in this directory to build and run the app:

```shell
docker compose up
```

The `vote` app will be running at [http://localhost:8080](http://localhost:8080), and the `results` will be at [http://localhost:8081](http://localhost:8081).

Alternately, if you want to run it on a [Docker Swarm](https://docs.docker.com/engine/swarm/), first make sure you have a swarm. If you don't, run:

```shell
docker swarm init
```

Once you have your swarm, in this directory run:

```shell
docker stack deploy --compose-file docker-stack.yml vote
```

## Run the app in Kubernetes

The folder k8s-specifications contains the YAML specifications of the Voting App's services.

Run the following command to create the deployments and services. Note it will create these resources in your current namespace (`default` if you haven't changed it.)

```shell
kubectl create -f k8s-specifications/
```

The `vote` web app is then available on port 31000 on each host of the cluster, the `result` web app is available on port 31001.

To remove them, run:

```shell
kubectl delete -f k8s-specifications/
```

## Architecture

![Architecture diagram](architecture.excalidraw.png)

* A front-end web app in [Python](/vote) which lets you vote between two options
* A [Redis](https://hub.docker.com/_/redis/) which collects new votes
* A [.NET](/worker/) worker which consumes votes and stores them in…
* A [Postgres](https://hub.docker.com/_/postgres/) database backed by a Docker volume
* A [Node.js](/result) web app which shows the results of the voting in real time

## Notes

The voting application only accepts one vote per client browser. It does not register additional votes if a vote has already been submitted from a client.

# 🚀 Ultimate Kubernetes Project on GKE

In this Kubernetes project, we will learn how to deploy a **multi-microservice architectured application** on **Google Kubernetes Engine (GKE)**.

---

## 📦 Application Code

👉 [Example Voting App Repository](https://github.com/dockersamples/example-voting-app)

---

## 🧱 Architecture
You can visualize the project architecture using **Excalidraw** (or draw your own).

---

## ⚙️ Step 1: Enable the Container API

```bash
gcloud services enable container.googleapis.com
```

---

## 🏗️ Step 2: Create a GKE Cluster

### Option 1: Create a High Availability (HA) GKE Cluster with 3 Nodes
```bash
gcloud container clusters create my-cluster   --region us-central1   --num-nodes 3   --machine-type e2-standard-4
```

### Option 2: Create a GKE Cluster with Autopilot Mode
*(Google manages the nodes for you)*

```bash
gcloud container clusters create-auto my-autopilot-cluster   --region us-central1
```

---

## 🔗 Step 3: Connect to the Cluster

From Cloud Shell or your VM:

```bash
gcloud container clusters get-credentials autopilot-cluster --region us-central1
```

---

## 🧰 Step 4: Clone the Git Repository

```bash
git clone https://github.com/dockersamples/example-voting-app.git
cd example-voting-app
```

---

## 🚀 Step 5: Run the App on the Cluster

```bash
kubectl apply -f k8s-specifications/
```

### ✅ Verify Deployment
```bash
kubectl get all
```

---

## 🌐 Step 6: Access the App Using NodePort

```bash
kubectl port-forward svc/vote 8080:8080
```

Access the app at:  
➡️ [http://localhost:8080](http://localhost:8080)

---

## 🌍 Step 7: Expose the App Using Ingress

### Step 7.1: Create Ingress Resource

```bash
kubectl create namespace ingress-nginx

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx   --namespace ingress-nginx   --set controller.service.type=LoadBalancer
```

---

### Step 7.2: Deploy Voting App Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: vote-ingress
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: vote
            port:
              number: 8080
```

---

### Step 7.3: Deploy Result App Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: result-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /result
        pathType: Prefix
        backend:
          service:
            name: result
            port:
              number: 8081
```

---

### Step 7.4: Verify Ingress and Access the Apps

```bash
kubectl get ingress
```

Once the LoadBalancer IP is available, open the app in your browser 🎉

---

## 🏁 Conclusion

You have successfully deployed a **multi-microservice application** on **Google Kubernetes Engine** using **Ingress** and **Helm**.

---

👨‍💻 **Author:** Brian Sumba  
📂 **Repository:** [Helpful Python Scripts](https://github.com/Brian432-cpu/Helpful-python-scripts)


This isn't an example of a properly architected perfectly designed distributed app... it's just a simple
example of the various types of pieces and languages you might see (queues, persistent data, etc), and how to
deal with them in Docker at a basic level.
