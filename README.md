## 🧱 Part 1: Microservice Stack

### ✅ Objective

Deploy a three-service microservice architecture using Kubernetes and Minikube. This stack mimics AWS EKS production practices with a focus on service isolation, resource management, and ingress exposure.

---

### 📦 Services Deployed

| Service        | Purpose                | Docker Image               | Access Scope     |
|----------------|------------------------|-----------------------------|------------------|
| `gateway`      | API Gateway            | `nginxdemos/hello`          | Public via Ingress |
| `auth-service` | Mock Auth Logic        | `kennethreitz/httpbin`      | Internal only    |
| `data-service` | Mock Business Logic    | `hashicorp/http-echo`       | Internal only    |

---
### 🧰 Helm Setup

I used a single Helm chart for deploying all three services (`gateway`, `auth-service`, `data-service`) with their manifests placed in the `templates/` directory.

