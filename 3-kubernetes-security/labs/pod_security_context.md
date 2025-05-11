# 🧪 Securing Workloads with Pod Security Context in Kubernetes

## 🎯 Objective

Learn to secure Kubernetes workloads using `securityContext` and `podSecurityContext` for least-privilege container execution.

---

## 🧰 Prerequisites

- Kubernetes cluster (Minikube, kind, etc.)
- `kubectl` configured
- Permissions to deploy workloads

---

## 🔹 Lab 1: Run a Pod as a Non-Root User

```yaml
# non-root-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: non-root-pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "id && sleep 3600"]
```

```bash
kubectl apply -f non-root-pod.yaml
kubectl logs non-root-pod
```

✅ Pod runs as a non-root user with specific UID/GID.

---

## 🔹 Lab 2: Set a Read-Only Root Filesystem

```yaml
# readonly-rootfs.yaml
apiVersion: v1
kind: Pod
metadata:
  name: readonly-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "touch /test || echo 'cannot write' && sleep 3600"]
    securityContext:
      readOnlyRootFilesystem: true
```

```bash
kubectl apply -f readonly-rootfs.yaml
kubectl logs readonly-pod
```

✅ Container cannot write to the root filesystem.

---

## 🔹 Lab 3: Drop Linux Capabilities

```yaml
# drop-capabilities.yaml
apiVersion: v1
kind: Pod
metadata:
  name: drop-caps-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "ping -c 1 127.0.0.1 || echo 'ping failed' && sleep 3600"]
    securityContext:
      capabilities:
        drop: ["ALL"]
```

```bash
kubectl apply -f drop-capabilities.yaml
kubectl logs drop-caps-pod
```

✅ Dropping capabilities prevents privileged operations like `ping`.

---

## 🔹 Lab 4: Disable Privilege Escalation

```yaml
# no-priv-escalation.yaml
apiVersion: v1
kind: Pod
metadata:
  name: no-priv-escalation
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    securityContext:
      allowPrivilegeEscalation: false
```

```bash
kubectl apply -f no-priv-escalation.yaml
```

✅ Prevents processes in the container from gaining additional privileges.

---

## 🔹 Lab 5: Clean Up

```bash
kubectl delete pod non-root-pod readonly-pod drop-caps-pod no-priv-escalation
```

---

## ✅ Wrap-Up

- ✅ Ran containers as non-root users
- ✅ Made root filesystems read-only
- ✅ Dropped unnecessary Linux capabilities
- ✅ Prevented privilege escalation
- ✅ Followed best practices for secure workload execution

---
