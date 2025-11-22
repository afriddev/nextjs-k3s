## Deploying a Next.js Frontend with HTTPS on Custom NodePort in Kubernetes
This tutorial walks you through containerizing your Next.js frontend and deploying it on a Kubernetes cluster under a dedicated namespace. Learn how to configure an NGINX sidecar proxy inside the pod to handle SSL termination using a self-signed certificate, enabling secure HTTPS traffic over a custom port (30001). Includes best practices for environment variables, TLS secret management, and applying Kubernetes manifests for deployment and service exposure.

#### Create Namespace
```bash
kubectl apply -f k8s/namespace.yaml
```
--- 
#### Create TLS Secret (run on K3s host)
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=104.154.187.72"
kubectl create secret tls nextjs-deployment-tls --key tls.key --cert tls.crt -n nextjs-app
```
---
#### Apply ConfigMap (NGINX SSL Proxy Configuration)
```bash
kubectl apply -f k8s/nginx.yaml
```
---
#### Apply Deployment
```bash
kubectl apply -f k8s/deployment.yaml
```
---
#### Apply ClusterIP Service
```bash
kubectl apply -f k8s/clusterip-service.yaml
```
---
#### Apply NodePort Service
```bash
kubectl apply -f k8s/nodeport-service.yaml
```
---
#### Build the Docker image locally
```bash
docker build -t his-registration:latest .
```
---
#### Load Docker image into K3s container runtime
```bash
docker save his-registration:latest | sudo k3s ctr images import -
```
---
#### Restart the deployment to use the new image
```bash
kubectl rollout restart deployment nextjs-deployment -n nextjs-app
```