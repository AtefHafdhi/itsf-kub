# 🚀 DevOps Kubernetes Assignment – Microservices with TLS & Non-Root Security

This project implements two Kubernetes-deployed microservices (`hello-risf` and `hello-itsf`), served securely using an NGINX Ingress with HTTPS. All containers run with a non-root user following best security practices.

---

## 📁 Project Structure

```bash
.
├── certs/
│   ├── hello-risf-ingress.yaml
│   ├── tls-risf.crt
│   ├── tls-risf.key
│   ├── tls-itsf.crt
│   └── tls-itsf.key
│
├── hello-risf/
│   ├── Dockerfile
│   ├── index.html
│   ├── deployments-risf.yaml
│   └── service-risf.yaml
│
├── hello-itsf/
│   ├── Dockerfile
│   ├── index.html
│   ├── deployment-itsf.yaml
│   ├── service-itsf.yaml
│   └── pv-pvc.yaml (optional/unused in final version)
└── README.md
```

---

## ✅ Objectives

- Deploy two microservices (`hello-risf`, `hello-itsf`) with custom HTML
- Secure traffic via Ingress and TLS
- Ensure **all containers run as non-root**
- Apply container security policies (read-only FS, no privilege escalation)

---

## ⚙️ Stack Used

- Kubernetes (Kind cluster on macOS)
- Docker
- NGINX
- Self-signed TLS certificates
- Ingress Controller (NGINX)
- ConfigMaps, initContainers, SecurityContext

---

## 🧪 How to Run

### 1. Start Kind Cluster

```bash
kind create cluster
```

### 2. Install NGINX Ingress Controller (Kind compatible)

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.4/deploy/static/provider/kind/deploy.yaml
```

### 3. Patch Ingress Controller (remove nodeSelector if pods are pending)

```bash
kubectl patch deployment ingress-nginx-controller -n ingress-nginx \
  --type='json' \
  -p='[{{"op": "remove", "path": "/spec/template/spec/nodeSelector"}}]'
```

### 4. Build and Load Images

```bash
# hello-risf
cd hello-risf/
docker build -t hello-risf:v1 .
kind load docker-image hello-risf:v1

# hello-itsf
cd ../hello-itsf/
docker build -t hello-itsf:v1 .
kind load docker-image hello-itsf:v1
```

### 5. Apply Kubernetes Resources

```bash
# Deploy hello-risf
kubectl apply -f hello-risf/deployments-risf.yaml
kubectl apply -f hello-risf/service-risf.yaml

# Deploy hello-itsf (uses initContainer + ConfigMap)
kubectl create configmap html-itsf --from-file=index.html
kubectl apply -f hello-itsf/deployment-itsf.yaml
kubectl apply -f hello-itsf/service-itsf.yaml

# Apply Ingress + TLS
kubectl create secret tls tls-risf --key=certs/tls-risf.key --cert=certs/tls-risf.crt
kubectl create secret tls tls-itsf --key=certs/tls-itsf.key --cert=certs/tls-itsf.crt
kubectl apply -f certs/hello-risf-ingress.yaml
```

### 6. Add Local DNS Entries

```bash
sudo nano /etc/hosts
```

```txt
127.0.0.1 hello-risf.local.domain
127.0.0.1 hello-itsf.local.domain
```

---

## 🌐 Access

- https://hello-risf.local.domain
- https://hello-itsf.local.domain

⚠️ Accept the TLS warning (self-signed certs).

---

## 🔐 Security Highlights

| Microservice   | User  | ReadOnly FS | Escalation Blocked | Init Container | ConfigMap HTML |
|----------------|-------|-------------|---------------------|----------------|----------------|
| hello-risf     | 1000  | ✅           | ✅                   | ❌             | ❌              |
| hello-itsf     | 1000  | ✅           | ✅                   | ✅             | ✅              |

---

## ⚠️ Key Challenges & Solutions

| Issue | Solution |
|-------|----------|
| NGINX failed to write to `/run` and `/var/cache/nginx` | Used `emptyDir` volumes for these paths |
| `initContainer` in `hello-itsf` couldn’t write to shared volume | Added `fsGroup: 1000` to the pod |
| HostPath PV not mountable via Kind | Switched to ConfigMap to inject `index.html` |
| Ingress stuck in Pending state | Removed Kind-specific `nodeSelector` from Ingress controller |

---

## 🙋 Author

**Atef Hafdhi**  
DevOps Engineer | Kubernetes | CI/CD | Cloud  
[LinkedIn](https://linkedin.com/in/atefhafdhi)

---

## 📜 License

MIT – Use freely for learning, testing, and demonstration.
