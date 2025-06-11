### Example Basic for Dapr ### 

# 🛍️ Dapr 1.14.5 Installation on k0s with Scheduler Support

This guide walks you through installing Dapr **v1.14.5** on a **k0s** single-node cluster, configuring the scheduler with a persistent volume.

---

## ✅ 1. Install k0s

```bash
curl -sSLf https://get.k0s.sh | sudo sh
k0s install controller --enable-worker
k0s start
```

Configure `kubectl` access:

```bash
mkdir -p /root/.kube
cp /var/lib/k0s/pki/admin.conf /root/.kube/config
```

---

## ✅ 2. Install Helm

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

---

## ✅ 3. Install Dapr using Helm

Install Dapr with Scheduler enabled and configured with persistent storage:

```bash
helm repo add dapr https://dapr.github.io/helm-charts/
helm repo update

helm install dapr dapr/dapr \
  --version 1.14.5 \
  --namespace dapr-system \
  --create-namespace \
  --set global.ha.enabled=false \
  --set dapr_scheduler.cluster.storageSize=16Gi \
  --set dapr_scheduler.etcdSpaceQuota=16Gi
```

Initialize Dapr runtime:

```bash
dapr init -k --runtime-version=1.14.5
```

---

## ✅ 4. Validate Dapr Pods

Check if all Dapr components are running:

```bash
kubectl get pods -n dapr-system
```

You should see `dapr-scheduler-server-0` among other core components.

---

## ✅ 5. Build and Deploy Your App

### 🐳 Build and Push Docker Image

```bash
docker build -t <YOUR_REPO>/dapr-api:0.1 .
docker push <YOUR_REPO>/dapr-api:0.1
```

### 📁 Prepare Persistent Volume Directory

Create the host directory for the Dapr scheduler persistent volume:

```bash
mkdir -p /mnt/data/dapr-scheduler-0
chmod 777 /mnt/data/dapr-scheduler-0
```

### 🛆 Apply the PersistentVolume

Ensure you have a `pv.yml` like:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: dapr-scheduler-data-dir-pv
spec:
  capacity:
    storage: 16Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/data/dapr-scheduler-0
```

Apply it:

```bash
kubectl apply -f pv.yml
```

---

## 🚀 6. Deploy Your Application

Check Dapr pods again:

```bash
kubectl get pods -n dapr-system
```

Then apply your deployment file:

```bash
kubectl apply -f deploy.yml
```

---

## ✅ Final Checks

Ensure that the scheduler pod is running:

```bash
kubectl get pod -n dapr-system -l app.kubernetes.io/name=dapr-scheduler
```

Check logs if needed:

```bash
kubectl logs -n dapr-system dapr-scheduler-server-0
```

