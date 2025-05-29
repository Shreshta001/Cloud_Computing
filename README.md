# Cloud_Computing
DOCKER_KUBERNETES_DEVOPS
# 📸 Pixa Search App – Full DevOps Monitoring & Scaling Project

This is a complete Image Search Web App deployed using Docker & Kubernetes, monitored with cAdvisor, Prometheus, and Grafana. It features both Horizontal and Vertical Pod Auto-scaling.

---

## 📂 Project Structure

```
image-search/
├── index.html
├── style.css
├── script.js
├── Dockerfile
├── deployment.yaml
├── service.yaml
├── pixa-vpa.yaml
├── prometheus.yml
```

---

## 🐳 Step 1: Docker Build & Push

```bash
docker build -t pixa-search-app .
docker tag pixa-search-app shreui/pixa-search-app:v1
docker push shreui/pixa-search-app:v1
```

---

## ☸️ Step 2: Kubernetes Deployment

Start Minikube:

```bash
minikube start
```

Apply deployment and service:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

Access your app:

```bash
minikube service pixa-search-service --url
```

---

## 📊 Step 3: Monitoring Setup

### 🟩 cAdvisor

```bash
docker run -d --name=cadvisor -p 8081:8080 \
--volume=//:/rootfs:ro \
--volume=/var/run:/var/run:ro \
--volume=/sys:/sys:ro \
--volume=/var/lib/docker/:/var/lib/docker:ro \
gcr.io/cadvisor/cadvisor:latest
```

Visit: [http://localhost:8081](http://localhost:8081)

### 🟦 Prometheus

Create prometheus.yml:

```yaml
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: "cadvisor"
    static_configs:
      - targets: ["host.docker.internal:8081"]
```

Run Prometheus:

```bash
docker run -d --name=prometheus -p 9090:9090 \
-v ${PWD}/prometheus.yml:/etc/prometheus/prometheus.yml \
prom/prometheus
```

Visit: [http://localhost:9090](http://localhost:9090)

### 🟨 Grafana

```bash
docker run -d --name=grafana -p 3000:3000 grafana/grafana
```

Visit: [http://localhost:3000](http://localhost:3000) 
Login: admin / admin

Add Prometheus Data Source:
- URL: http://host.docker.internal:9090

Import Dashboard ID: 193

---

## 📈 Step 4: Horizontal Pod Autoscaling (HPA)

Install metrics-server:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Create HPA:

```bash
kubectl autoscale deployment pixa-search-deployment --cpu-percent=50 --min=2 --max=10
```

Simulate load:

```bash
kubectl run -i --tty load-generator --image=busybox /bin/sh
while true; do wget -q -O- http://pixa-search-service.default.svc.cluster.local; done
```

Check status:

```bash
kubectl get hpa
```

---

## 📏 Step 5: Vertical Pod Autoscaling (VPA)

Clone VPA repo:

```bash
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler
./hack/vpa-up.sh
```

Create pixa-vpa.yaml:

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: pixa-search-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: pixa-search-deployment
  updatePolicy:
    updateMode: "Auto"
```

Apply:

```bash
kubectl apply -f pixa-vpa.yaml
```

Monitor:

```bash
kubectl describe vpa pixa-search-vpa
```

---

## ✅ Final Checklist

| Component        | Status                          |
|------------------|----------------------------------|
| Docker Image     | ✅ Built & pushed to Docker Hub |
| Kubernetes App   | ✅ Deployed                     |
| cAdvisor         | ✅ Running on port :8081        |
| Prometheus       | ✅ Collecting metrics on :9090  |
| Grafana          | ✅ Dashboards on :3000          |
| HPA              | ✅ Active and scaling           |
| VPA              | ✅ Monitoring + adjusting pods  |

---

© 2025 – Pixa Search DevOps Project 🚀
