# 🧪 Using Falco for Runtime Security in Kubernetes

## 🎯 Objective

Deploy and use Falco to detect suspicious activity and runtime security events inside a Kubernetes cluster.

---

## 🧰 Prerequisites

- Kubernetes cluster
- `kubectl` configured
- Helm installed

---

## 🔹 Lab 1: Install Falco using Helm

Add the Falco Helm repository:

```bash
helm repo add falcosecurity https://falcosecurity.github.io/charts
helm repo update
```

Install Falco:

```bash
helm install --replace falco --namespace falco --create-namespace --set tty=true falcosecurity/falco
```

✅ Falco daemonset is deployed to monitor syscalls on all cluster nodes.

---

## 🔹 Lab 2: Check Falco is Running

```bash
kubectl get pods -n falco
kubectl logs -n falco -l app=falco
```

✅ Falco should be logging system call events.

---

## 🔹 Lab 3: Trigger Suspicious Activities

### Step 1: Create a Pod to Simulate an Attacker

```bash
kubectl create deployment nginx --image=nginx
```

---

### Step 2: Touch a Sensitive File

```bash
kubectl exec -it $(kubectl get pods --selector=app=nginx -o name) -- cat /etc/shadow
```

✅ Expected: Falco alerts on sensitive file access.

---

### Step 3: Start a Shell Inside a Container (Unexpected Shell)

```bash
kubectl exec -it $(kubectl get pods --selector=app=nginx -o name) -- bash
```

✅ Expected: Falco may alert on a shell being spawned in a container.

---

### 3.4: Make a Network Connection from Pod

Inside the pod:

```bash
apt install wget
wget http://example.com
```

✅ Expected: Falco alerts on wget.

---

✅ Expected: Falco detects privileged container creation.

---

## 🔹 Lab 4: View Falco Alerts

Check Falco logs:

```bash
kubectl logs -l app.kubernetes.io/name=falco -n falco -c falco
```

✅ You should see alerts for file access, shell execution or network activity.

---

## 🔹 Lab 6: Clean Up

```bash
kubectl delete pod attacker privcontainer
helm uninstall falco
```

---

## ✅ Wrap-Up

- ✅ Installed Falco with Helm
- ✅ Triggered multiple runtime security events
- ✅ Detected suspicious activity like file tampering, shell spawning, network access, and privilege escalation
- ✅ Practiced real-world runtime threat detection

---
