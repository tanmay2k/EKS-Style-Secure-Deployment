# Simulated EKS-Style Secure Deployment Using Minikube

## 📦 Project Overview

This project simulates a basic microservices architecture in Kubernetes using Prometheus and Grafana for monitoring, Kyverno for policy enforcement, and MinIO for object storage. It includes tasks around deploying services, simulating IAM, securing pods, and visualizing metrics.

---

## 🚀 Setup Instructions

### 1. Prerequisites

* Minikube or any local Kubernetes cluster
* kubectl
* Helm
* Prometheus & Grafana
* Kyverno

### 2. Clone the Repo & Deploy Helm Chart

```bash
cd microservices
helm install microservices . --namespace app --create-namespace
```

### 3. Deploy MinIO (Mock S3)

```bash
helm repo add minio https://charts.min.io/
helm install minio minio/minio \
  --namespace minio \
  --create-namespace \
  --set mode=standalone \
  --set replicas=1 \
  --set accessKey=minioadmin \
  --set secretKey=minioadmin \
  --set persistence.enabled=false \
  --set resources.requests.memory=512Mi \
  --set resources.requests.cpu=250m
```

### 4. Create MinIO Secret in `app` Namespace

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: minio-creds
  namespace: app
type: Opaque
stringData:
  MINIO_ACCESS_KEY: minioadmin
  MINIO_SECRET_KEY: minioadmin
```

Apply it:

```bash
kubectl apply -f minio/minio-creds.yaml
```

### 5. Deploy Services

```bash
kubectl apply -f templates/data-service.yaml
kubectl apply -f templates/auth-service.yaml
```

### 6. Install Monitoring Stack

```bash
helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace
```

Forward Grafana:

```bash
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80
```

Login → user: `admin` / pass: `prom-operator` or check secret.

---

## 🗺️ Architecture Diagram (ASCII)

```
          +--------------+
          |   Gateway    |
          +------+-------+
                 |
       +---------+----------+
       |                    |
+------+-----+       +------+-----+
| Auth Service|       | Data Service|
+------------+       +------------+
       |                    |
       |      +-------------+-------------+
       |      |                           |
       v      v                           v
  Prometheus  --> MinIO (mock S3)     Grafana
       ^                                ^
       |                                |
    Kyverno (Policies)         kube-prometheus-stack
```

---

## 💡 Design Decisions

* **MinIO as mock S3**: Simplifies IAM simulation.
* **ServiceAccount Binding**: `data-service` is the only one with access to `minio-creds`.
* **Kyverno**: Used to restrict which pods can mount the secret.
* **Prometheus Operator**: Provides service monitoring and alerting.
* **Lightweight containers**: `httpbin` and `http-echo` to minimize resource overhead.

---

## 🔐 Security Incident (Simulated)

**Incident:** Unauthorized access attempt to `minio-creds` by `auth-service`.

**Detection:** Kyverno policy `restrict-minio-secret` denied the deployment.

**Fix:**

* Removed `envFrom` mount of `minio-creds` from `auth-service`.
* Policy enforced to only allow `data-service` to mount that secret.

---

## ⚠️ Assumptions & Known Issues

* HTTP request metrics (`http_requests_total`) are **not emitted** because the app containers are uninstrumented.
* These panels are kept in the dashboard for structure but may show "no data".
* Resource metrics may show flat lines unless workload generates load (use `busybox` CPU loop to simulate).
* Alerts are only set up for **pod restarts**.

---

## ✅ Features Implemented

* [x] Prometheus + Grafana setup
* [x] MinIO mock S3 with secret-based access
* [x] Kyverno IAM policy
* [x] Network policies
* [x] CPU/Memory/Restart metrics visualized
* [x] Restart alerting

---

