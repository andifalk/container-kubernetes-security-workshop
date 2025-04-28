# 🧪 Using Falco for Runtime Security in Kubernetes

## 🎯 Objective
Deploy and use Falco to detect suspicious activity and runtime security events inside a Kubernetes cluster.

---

## 🧰 Prerequisites

- Kubernetes cluster (Minikube, kind, etc.)
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
helm install falco falcosecurity/falco
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

### 3.1: Create a Pod to Simulate an Attacker

```bash
kubectl run attacker --image=busybox --command -- sleep 3600
kubectl exec attacker -- sh
```

---

### 3.2: Touch a Sensitive File

Inside the pod:

```bash
touch /etc/passwd
```

✅ Expected: Falco alerts on sensitive file access.

---

### 3.3: Start a Shell Inside a Container (Unexpected Shell)

Inside the pod:

```bash
sh
```

✅ Expected: Falco may alert on a shell being spawned in a container.

---

### 3.4: Make a Network Connection from Pod

Inside the pod:

```bash
wget http://example.com
```

✅ Expected: Falco alerts on outbound network traffic if rules are configured.

---

### 3.5: Spawn a Privileged Container (Cluster-wide)

```bash
kubectl run privcontainer --image=busybox --overrides='
{
  "apiVersion": "v1",
  "spec": {
    "containers": [
      {
        "name": "busybox",
        "image": "busybox",
        "securityContext": {
          "privileged": true
        },
        "command": ["sleep", "3600"]
      }
    ]
  }
}' --restart=Never
```

✅ Expected: Falco detects privileged container creation.

---

## 🔹 Lab 4: View Falco Alerts

Check Falco logs:

```bash
kubectl logs -n falco -l app=falco
```

✅ You should see alerts for file access, shell execution, network activity, and privileged containers.

---

## 🔹 Lab 5: Customize Falco Rules (Optional)

Get Falco rules config:

```bash
kubectl get configmap falco-rules -n falco -o yaml
```

Edit or add new custom detection rules for your environment.

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