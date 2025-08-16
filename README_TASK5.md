# Task 5 — Minikube on Azure VM (Ubuntu 22.04)

## Purpose
**Learn Kubernetes basics** by deploying and managing an app locally with **Minikube**: create a Deployment and expose it via a Service, verify Pods/Services, and scale the Deployment. (Tools required by the brief: **Minikube, kubectl, Docker**.)

## Prerequisites
- Same Azure VM you used earlier (Ubuntu 22.04). Docker should already be installed.
- Open inbound ports on the VM's NSG: **22 (SSH)** and **30080 (NodePort)**.

## 1) Install kubectl + minikube
```bash
# kubectl
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" |   sudo tee /etc/apt/sources.list.d/kubernetes.list > /dev/null
sudo apt-get update && sudo apt-get install -y kubectl

# minikube (binary)
curl -fsSLo minikube.deb https://storage.googleapis.com/minikube/releases/latest/minikube-latest.amd64.deb
sudo dpkg -i minikube.deb || sudo apt-get -f install -y && sudo dpkg -i minikube.deb
minikube version
kubectl version --client
```

## 2) Start Minikube (Docker driver)
> On small VMs, keep resources low. If start fails due to memory, stop other workloads or resize the VM.
```bash
sudo swapoff -a                 # kubelet prefers swap OFF
sudo systemctl enable --now docker
minikube start --driver=docker --cpus=1 --memory=1792
minikube status
kubectl get nodes -o wide
```

## 3) Apply the manifests
```bash
kubectl apply -f namespace.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

kubectl -n task5 get pods -o wide
kubectl -n task5 get svc -o wide
```

## 4) Verify and access
- From your PC/browser: `http://<PUBLIC_IP>:30080`
- Or from the VM: `curl -I http://localhost:30080`

You should see NGINX responding. If the port is blocked, add an inbound NSG rule for **TCP 30080**.

## 5) Scale and inspect
```bash
kubectl -n task5 scale deployment/hello-deploy --replicas=3
kubectl -n task5 get pods
kubectl -n task5 describe deployment/hello-deploy
kubectl -n task5 logs -l app=hello --tail=20
```

## 6) Clean up
```bash
kubectl delete -f service.yaml
kubectl delete -f deployment.yaml
kubectl delete -f namespace.yaml
# Optional: stop or delete minikube
minikube stop
# minikube delete
```

## Deliverables (what to upload in your Task5 repo)
- `namespace.yaml`, `deployment.yaml`, `service.yaml`
- Screenshots: `kubectl get pods -n task5`, `kubectl get svc -n task5`, browser hitting `http://<PUBLIC_IP>:30080`
- A README explaining steps (you can use this README)
