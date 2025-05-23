# 🚀 Kube-Prometheus Stack Installation and Configuration Guide (with Ingress)

The **kube-prometheus-stack** provides a complete **Kubernetes monitoring solution**, including **Prometheus**, **Alertmanager**, **Grafana**, and **Kube-State-Metrics**. This guide covers **installation**, **ETCD monitoring**, **kube-proxy configuration**, and **Ingress setup** for easy access through custom domains.

---

## 📦 **1. Prerequisites**

- **Kubernetes cluster** up and running.
- **Helm** installed (`helm version` to verify).
- **Ingress Controller** (e.g., **NGINX Ingress**) installed and configured.
- **Kubernetes CLI (kubectl)** configured (`kubectl version` to verify).
- **Update `/etc/hosts`** with the following entries:

```plaintext
127.0.0.1 grafana.devops.local
127.0.0.1 alertmanager.devops.local
127.0.0.1 prometheus.devops.local
```

---

## ⚙️ **2. Set Up ETCD Monitoring**

### 🟢 **Step 1: Deploy Busybox Pod**

```sh
kubectl apply -f get-etcd-cert.yaml
```

- This **`get-etcd-cert.yaml`** file should define a **busybox pod** with **access to host certificates**.

---

### 🟢 **Step 2: Create ETCD Client Certificate Secret**

1. **Retrieve Busybox Pod Name**:

```sh
podname=$(kubectl get pods -o=jsonpath='{.items[0].metadata.name}' -n default | grep busybox)
```

2. **Create ETCD Client Secret**:

```sh
kubectl create secret generic etcd-client-cert -n monitoring \
--from-literal=etcd-ca="$(kubectl exec $podname -n default -- cat /etc/kubernetes/pki/etcd/ca.crt)" \
--from-literal=etcd-client="$(kubectl exec $podname -n default -- cat /etc/kubernetes/pki/apiserver-etcd-client.crt)" \
--from-literal=etcd-client-key="$(kubectl exec $podname -n default -- cat /etc/kubernetes/pki/apiserver-etcd-client.key)"
```

3. **Clean Up Busybox Pod**:

```sh
kubectl delete pod -n default busybox
```

---

## 🚀 **3. Install Kube-Prometheus Stack with Ingress**

### 🟢 **Step 1: Add Prometheus Community Helm Repository**

```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

---

### 🟢 **Step 2: Install Kube-Prometheus Stack with Ingress**

```sh
helm upgrade --install prometheus-stack prometheus-community/kube-prometheus-stack \
    --namespace monitoring \
    -f values.yaml \
    --create-namespace
```

- **Namespace:** `monitoring` (Created if not already present).
- **Values File (`values.yaml`):** Should include **Ingress configurations**:


---

## 🌐 **4. Access Monitoring Tools via Ingress**

- **Grafana:** [http://grafana.devops.local](http://grafana.devops.local)  
- **Prometheus:** [http://prometheus.devops.local](http://prometheus.devops.local)  
- **Alertmanager:** [http://alertmanager.devops.local](http://alertmanager.devops.local)  

### 🟢 **Default Grafana Credentials:**
- **Username:** `admin`
- **Password:** `prom-operator`

---

## 🔍 **5. Configure Kube-Proxy Metrics Exposure**

### **Problem: Cannot Access Kube-Proxy Metrics**

1. **Edit Kube-Proxy ConfigMap:**

```sh
kubectl edit -n kube-system cm kube-proxy
```

2. **Update `metricsBindAddress`:**

```yaml
metricsBindAddress: "0.0.0.0:10249"
```

3. **Restart Kube-Proxy DaemonSet:**

```sh
kubectl rollout restart daemonset -n kube-system kube-proxy
```

4. **Verify Access:**

```sh
curl http://<kube-proxy-ip>:10249/metrics
```

---

## 📈 **6. Validate Monitoring Setup**

### **Verify All Pods Are Running:**

```sh
kubectl get pods -n monitoring
```

### **Check Ingress Status:**

```sh
kubectl get ingress -n monitoring
```

---

## 📋 **7. Troubleshooting Tips**

### **1. Check Prometheus Logs:**

```sh
kubectl logs -n monitoring -l app.kubernetes.io/name=prometheus
```

### **2. Check Grafana Logs:**

```sh
kubectl logs -n monitoring -l app.kubernetes.io/name=grafana
```

### **3. Validate ETCD Certificates:**

```sh
kubectl describe secret etcd-client-cert -n monitoring
```

### **4. Verify Ingress Connectivity:**

```sh
kubectl describe ingress -n monitoring
```

---

## 🔄 **8. Cleanup Resources**

### **Uninstall the Prometheus Stack:**

```sh
helm uninstall prometheus-stack -n monitoring
```

### **Delete ETCD Secret:**

```sh
kubectl delete secret etcd-client-cert -n monitoring
```

---

## 🚦 **9. Next Steps**

- Set up **Alerting Rules** in **Alertmanager**.
- Configure **Grafana Dashboards** with **custom metrics**.
- Implement **Persistent Storage** for **Prometheus** and **Grafana**.
- Test **Autoscaling** using **Metrics Server** and **HPA**.

---

## 📂 **References**

- [Kube-Prometheus Stack Documentation](https://artifacthub.io/packages/helm/prometheus-community/kube-prometheus-stack)
- [Prometheus Documentation](https://prometheus.io/docs/introduction/overview/)
- [Grafana Documentation](https://grafana.com/docs/)
- [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/)
