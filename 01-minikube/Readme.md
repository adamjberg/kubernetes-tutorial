# How to Setup a Local Kubernetes Cluster with minikube

## Installation

Install [kubectl](https://kubernetes.io/releases/download/#kubectl)
Install [minikube](https://minikube.sigs.k8s.io/docs/start/)

## Start minikube cluster

```bash
minikube start
```

## Deploy Simple nginx Container

```
kubectl create deployment balanced --image=docker.io/nginx:1.23
kubectl expose deployment balanced --type=LoadBalancer --port=80
```

## Start minikube tunnel

In order to access the deployment, the following command must be run.

```
minikube tunnel
```

Now you should be able to access the default "Welcome to nginx!" page by navigating to [http://localhost](http://localhost)

## Delete Everything

Now that we've successfully tested our installation, run:

```
minikube delete --all
```

Which will delete all minikube clusters.