# How to Run Simple Web App With Docker Using nginx

## Add Simple TypeScript File

```ts
alert("Hello World!");
```

## Install esbuild

```
npm i --save-dev esbuild
```

## Add build Script

The build script below outputs to `static/js/app.js`.

```json
{
  "name": "02-react-kube",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "build": "esbuild src/index.ts --bundle --minify --outfile=static/js/app.js"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "esbuild": "^0.15.10"
  }
}

```

## Create static/index.html

```html
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <script src="/js/app.js"></script>
</head>
<body>
  
</body>
</html>
```

## Create Dockerfile

```Dockerfile
# Dockerfile
# Starts with an alpine linux image with node pre-installed
FROM node:18-alpine3.15 as fe

# All future commands will operate out of this directory (in the image)
WORKDIR /fe

# First copy package.json
COPY package*.json ./

# Any change to package.json will now ensure that this install step is run
# Otherwise the install won't need to re-run
RUN npm install

# Copy over the src files
COPY src ./src

# Copy over the index.html file in the static folder
COPY static ./static

# Run the build script
RUN npm run build

# Starts a new image with nginx pre-configured
FROM nginx:1.23.1-alpine

COPY nginx/nginx.conf /etc/nginx/conf.d/default.conf

WORKDIR /usr/share/nginx/html

# Removes default nginx files
RUN rm -rf ./*

EXPOSE 80

# Copies over built files from the fe image defined at the start of this file
COPY --from=fe /fe/static .
```

## Build Docker Image Tagged as `webapp`

docker build . -t webapp

## Add deployment yaml file

```yaml
# deployment.yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: webapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: webapp
        ports:
        - containerPort: 80
```

## Start Local Kubernetes Cluster

```
minikube start
```

## Create namespace called webapp

```
kubectl create namespace webapp
```

## Set `webapp` as Default Context

```
kubectl config set-context --current --namespace=webapp
```

## Create New Deployment

```
kubectl apply -f deployment.yaml 
```

## Monitor Status of Deployment

```
kubectl get deployment -w
```

## Open MiniKube Dashboard

```
minikube dashboard
```

Select the `webapp` namespace in the top left dropdown.

Under Workflows > Deployments you should see a red circle next to `webapp` that indicates the deployment did not succeed.

Click on the deployment and then click on the link under the "New Replica Set" heading.  This will take you to a page where you can now "View Logs" (top right of the dashboard). Click this and you should see: "container "webapp" in pod "webapp-5f5c7cd6c5-54gtp" is waiting to start: image can't be pulled"

This is because it is trying to pull the image from the public registry.  You could set things up so that you push to Docker hub, etc.  But I found it easier to enable local images to be used instead.

## Add `imagePullPolicy: Never` to deployment.yaml

```diff
kind: Deployment
apiVersion: apps/v1
metadata:
  name: webapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
      - name: webapp
        image: webapp
+       imagePullPolicy: Never
        ports:
        - containerPort: 80
```

This ensures that the image won't be pulled from the public registry.

## Configure Current Terminal to Run Against docker inside minikube Cluster

```
eval $(minikube docker-env)
```

This command will cause docker commands in this terminal to be run within the minikube cluster.

## Re-run docker build

Now that docker is configured to build within minikube, we need to rebuild the `webapp` image.

```
docker build . -t webapp
```

## Deploy deployment.yaml

```
kubectl apply -f deployment.yaml
```

## Check Status

```
kubectl get deployment -w
```

Or alternatively you can go back to the minikube dashboard and see that the deployment now shows up as green.

## Create a LoadBalancer Service

```yaml
# load-balancer.yaml
apiVersion: v1
kind: Service
metadata:
  name: load-balancer
  labels:
    app: webapp
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    nodePort: 31000
  selector:
    app: webapp
```

## Open Browser via minikube service

```
minikube service load-balancer -n webapp
```

This starts a tunnel that is required in order to actually access the deployment.

## Wrap-up

It took me nearly a full day to wrap my head around this much.  The next step is to have a look at how this integrates with a kubernetes cluster within Digital Ocean.

# Resources

https://blog.logrocket.com/deploy-react-app-kubernetes-using-docker/