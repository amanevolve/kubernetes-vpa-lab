
# 🚀 Kubernetes Vertical Pod Autoscaler (VPA) Lab

This project demonstrates Kubernetes **Vertical Pod Autoscaler (VPA)** with step-by-step installation, deployment, and monitoring.

---

# 🎯 Objective

- Setup Kubernetes cluster (Kind / Minikube)
- Install kubectl
- Install Metrics Server
- Install VPA (Vertical Pod Autoscaler)
- Deploy sample application
- Observe automatic CPU & Memory scaling

---

# 🧠 What is VPA?

VPA automatically:
- Adjusts CPU & Memory requests
- Gives recommendations for pods
- Restarts pods to apply new resources

---

# ⚙️ Step 1: System Setup

```bash
sudo apt-get update -y
sudo apt-get upgrade -y
sudo apt-get install -y curl git wget apt-transport-https ca-certificates
````

---

# ☸️ Step 2: Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

---

# 🐳 Step 3: Install Docker

```bash
sudo apt-get install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
docker --version
```

---

# 📦 Step 4: Install Kind Cluster

```bash
curl -Lo kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x kind
sudo mv kind /usr/local/bin/
kind version
```

Create cluster:

```bash
kind create cluster --name vpa-cluster
kubectl get nodes
```

---

# 📊 Step 5: Install Metrics Server

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

If using Kind, patch metrics server:

```bash
kubectl patch deployment metrics-server -n kube-system \
--type='json' \
-p='[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'
```

Verify:

```bash
kubectl top nodes
kubectl get pods -n kube-system
```

---

# 📦 Step 6: Install VPA

```bash
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler/
./hack/vpa-up.sh
```

Verify:

```bash
kubectl get pods -n kube-system | grep vpa
```

---

# 🚀 Step 7: Create Application

Create folder:

```bash
mkdir manifests
cd manifests
```

---

## deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "200m"
            memory: "256Mi"
```

Apply:

```bash
kubectl apply -f deployment.yaml
kubectl get pods
```

---

# 📈 Step 8: Create VPA

## vpa.yaml

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: nginx-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-app
  updatePolicy:
    updateMode: "Auto"
```

Apply:

```bash
kubectl apply -f vpa.yaml
kubectl get vpa
kubectl describe vpa nginx-vpa
```

---

# 📊 Step 9: Monitor Resources

```bash
kubectl top nodes
kubectl top pods
kubectl describe vpa nginx-vpa
``
