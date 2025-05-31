# 📸 Pixa Search App - Full DevOps Pipeline (Docker + Kubernetes + Monitoring + Autoscaling)

This guide walks through the full DevOps lifecycle for the `pixa-search-app`, including:

- Dockerization
- Deployment on Kubernetes via Minikube
- Monitoring via Prometheus + Grafana + cAdvisor
- Horizontal & Vertical Pod Autoscaling (HPA + VPA)

---

## 🐳 Docker Workflow

### 1. Build Docker Image
```powershell
docker build -t pixa-search-app .
```

### 2. Run Locally
```powershell
docker run -d -p 8080:80 --name pixa-search-container pixa-search-app
```

### 3. Push to DockerHub
```powershell
docker login
docker build -t shreui/pixa-search-app:v1 .
docker push shreui/pixa-search-app:v1
```

---

## 📊 Monitoring Setup

### ▶️ cAdvisor (Container Monitoring)
```powershell
docker run -d --name cadvisor -p 8081:8080 \
  --volume="C:/":/rootfs:ro \
  --volume="/var/run":/var/run:ro \
  --volume="/sys":/sys:ro \
  --volume="/var/lib/docker/":/var/lib/docker:ro \
  gcr.io/cadvisor/cadvisor:latest
```
🔗 Visit: [http://localhost:8081](http://localhost:8081)

---

### ▶️ Prometheus
Make sure `prometheus.yml` is in your working directory.

```powershell
docker run -d --name prometheus -p 9090:9090 \
  -v "${PWD}\prometheus.yml:/etc/prometheus/prometheus.yml" \
  prom/prometheus
```
🔗 Visit: [http://localhost:9090](http://localhost:9090)

📊 Query to Try:
```text
container_memory_usage_bytes
```

---

### ▶️ Grafana
```powershell
docker run -d --name=grafana -p 3000:3000 grafana/grafana
```

🔗 Visit: [http://localhost:3000](http://localhost:3000)  
👤 Login: `admin / admin`  
➕ Add Data Source:
- Type: **Prometheus**
- URL: `http://host.docker.internal:9090`

📅 Import Dashboard:
- ID: **193**

---

## ☘️ Kubernetes (Minikube) Setup

### ▶️ Start Minikube
```powershell
minikube start --driver=docker --force
```

### ▶️ Deploy App
```powershell
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

### ▶️ Check Resources
```powershell
kubectl get nodes
kubectl get pods
kubectl get deployments
kubectl get svc
```

🔗 Access via:
```powershell
minikube service pixa-search-service --url
```

---

## 📈 Horizontal Pod Autoscaler (HPA)

### ▶️ Enable Metrics Server
```powershell
minikube addons enable metrics-server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Check:
```powershell
kubectl get deployment metrics-server -n kube-system
kubectl get pods -n kube-system
```

### ▶️ Create HPA
```powershell
kubectl autoscale deployment pixa-search-deployment --cpu-percent=50 --min=1 --max=10
kubectl get hpa
```

### ▶️ Simulate Load
```powershell
kubectl run -i --tty load-generator --image=busybox -- /bin/sh
```

Inside BusyBox shell:
```sh
while true; do wget -O- http://pixa-search-service.default.svc.cluster.local; sleep 1; done
```

To manually spike CPU inside a pod:
```powershell
kubectl exec -it <pod-name> -- /bin/sh
# Inside:
while true; do :; done
```

### 🧪 Check HPA Status
```powershell
kubectl get hpa -w
kubectl top pod
kubectl top node
```

---

## 📊 Vertical Pod Autoscaler (VPA)

### ▶️ Clone and Setup VPA
```powershell
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler
./hack/vpa-up.sh
```

### ▶️ Apply VPA Policy
Create `pixa-vpa.yaml`, then:
```powershell
kubectl apply -f pixa-vpa.yaml
kubectl describe vpa pixa-search-vpa
```

---

## 📂 Sample YAML Files

### 📁 deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pixa-search-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: pixa-search
  template:
    metadata:
      labels:
        app: pixa-search
    spec:
      containers:
        - name: pixa-search
          image: shreui/pixa-search-app:v1
          resources:
            requests:
              cpu: "100m"
            limits:
              cpu: "200m"
          ports:
            - containerPort: 80
```

### 📁 service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: pixa-search-service
spec:
  selector:
    app: pixa-search
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: NodePort
```

### 📁 pixa-vpa.yaml
```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: pixa-search-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind:       Deployment
    name:       pixa-search-deployment
  updatePolicy:
    updateMode: "Auto"
```

---

## ✅ Final Checks

| Component       | URL                         |
|----------------|------------------------------|
| App            | `minikube service ...`       |
| cAdvisor       | http://localhost:8081        |
| Prometheus     | http://localhost:9090        |
| Grafana        | http://localhost:3000        |
| HPA Metrics    | `kubectl get hpa`            |
| VPA Status     | `kubectl describe vpa ...`   |

---

## 💡 Troubleshooting

- If HPA isn't scaling, ensure:
  - Metrics Server is healthy
  - CPU requests and limits are defined
  - Load is actually generated
- Run `kubectl top pod` to verify metrics
- If Grafana shows no data:
  - Check Prometheus data source
  - Use correct query like `container_cpu_usage_seconds_total`

---

## 📦 Cleanup
```powershell
minikube stop
minikube delete
docker stop $(docker ps -q)
docker system prune -a
```

---

> ✨ You're now running a production-like monitoring and scaling setup entirely in local development. Congrats!
