# 📸 Pixa Search App – Dockerized Deployment with Monitoring

This project is a Dockerized image search application named **Pixa Search App**, which is built, pushed to Docker Hub, and deployed with integrated monitoring using **cAdvisor**, **Prometheus**, and **Grafana**.

---

## 🐳 Docker Setup

### 🔨 Build the Docker Image

```bash
docker build -t pixa-search-app .
```

### 🚀 Run the Docker Container

```bash
docker run -d -p 8080:80 --name pixa-search-container pixa-search-app
```

### ☁️ Push to Docker Hub

```bash
docker login
docker build -t shreui/pixa-search-app:v1 .
docker push shreui/pixa-search-app:v1
```

---

## 📊 Monitoring Setup

We use the following tools for monitoring container metrics:

- 📦 **cAdvisor**
- 📈 **Prometheus**
- 📉 **Grafana**

---

### 🔍 cAdvisor (Container Advisor)

```powershell
docker run -d --name cadvisor -p 8081:8080 `
  --volume="C:/":/rootfs:ro `
  --volume="/var/run":/var/run:ro `
  --volume="/sys":/sys:ro `
  --volume="/var/lib/docker/":/var/lib/docker:ro `
  gcr.io/cadvisor/cadvisor:latest
```

📍 Visit: [http://localhost:8081](http://localhost:8081)

---

### 📈 Prometheus

Create a `prometheus.yml` file in your working directory with the following content:

```yaml
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['host.docker.internal:8081']
```

Run the Prometheus container using PowerShell:

```powershell
docker run -d --name prometheus -p 9090:9090 -v "${PWD}\prometheus.yml:/etc/prometheus/prometheus.yml" prom/prometheus
```

📍 Visit: [http://localhost:9090](http://localhost:9090)

🔎 Sample Query:

```
container_memory_usage_bytes
```

---

### 📉 Grafana

```bash
docker run -d --name=grafana -p 3000:3000 grafana/grafana
```

📍 Visit: [http://localhost:3000](http://localhost:3000)

🔑 **Login Credentials:**

- Username: `admin`
- Password: `admin`

---

### ➕ Add Prometheus as Data Source

1. Go to **Settings > Data Sources > Add data source**
2. Choose **Prometheus**
3. Set the URL to:

```
http://host.docker.internal:9090
```

4. Click **Save & Test**

---

### 📊 Import Docker Monitoring Dashboard

1. Go to ➕ > **Import**
2. Use **Dashboard ID**: `193`
3. Follow the prompts and link it to your Prometheus data source

---

## ✅ Summary

| Tool       | Port | URL                      |
|------------|------|--------------------------|
| Pixa App   | 8080 | http://localhost:8080    |
| cAdvisor   | 8081 | http://localhost:8081    |
| Prometheus | 9090 | http://localhost:9090    |
| Grafana    | 3000 | http://localhost:3000    |

---

## 📦 Directory Requirements

Make sure the following file exists before running Prometheus:

### `prometheus.yml`:

```yaml
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['host.docker.internal:8081']
```

---

## 📬 Notes

- ✅ Ensure Docker Desktop is running with access to your file system.
- 🖥️ Replace `C:/` with appropriate paths if using Linux/macOS.
- 🌐 Use `host.docker.internal` for container-to-host communication on Windows/macOS.
  - On Linux, you may need to use the host's IP or run with `--network=host`.

---

## 🧠 Happy Coding!
