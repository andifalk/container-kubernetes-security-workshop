# 🧪 Lab Series: Secure Use of Containers in Kubernetes

## 🎯 Objective
Learn to secure containers in Kubernetes using `securityContext`, Pod Security Admission, and runtime profiles (seccomp, AppArmor).

---

## 🧰 Prerequisites

- Kubernetes cluster (Minikube, kind, or real)
- `kubectl` configured
- Cluster admin privileges
- Optional tools: `kube-bench`, `kubescape`, `trivy`

---

## 🔹 Lab 1: Run Containers as Non-Root

### Step 1: Create the pod spec

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: non-root-demo
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "id && sleep 3600"]
    securityContext:
      runAsUser: 1000
      allowPrivilegeEscalation: false
```

### Step 2: Apply and check logs

```bash
kubectl apply -f non-root-pod.yaml
kubectl logs non-root-demo
```

✅ Expected: User ID is 1000, not root.

---

## 🔹 Lab 2: Drop Linux Capabilities

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: drop-caps
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
kubectl apply -f drop-caps-pod.yaml
kubectl logs drop-caps
```

✅ Expected: `ping` fails due to dropped capabilities.

---

## 🔹 Lab 3: Use Read-Only Root Filesystem

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readonly-demo
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "touch /test || echo 'failed' && sleep 3600"]
    securityContext:
      readOnlyRootFilesystem: true
```

```bash
kubectl apply -f readonly-pod.yaml
kubectl logs readonly-demo
```

✅ Expected: Cannot write to root filesystem.

---

## 🔹 Lab 4: Apply Seccomp and AppArmor (if supported)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: seccomp-demo
  annotations:
    seccomp.security.alpha.kubernetes.io/pod: runtime/default
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
```

```bash
kubectl apply -f seccomp-pod.yaml
```

✅ Expected: Pod uses restricted syscall set.

---

## 🔹 Lab 5: Enforce Policies with Pod Security Admission (PSA)

### Step 1: Create secure namespace

```bash
kubectl create ns secure-ns
kubectl label ns secure-ns pod-security.kubernetes.io/enforce=restricted
```

### Step 2: Try applying an insecure pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: insecure
  namespace: secure-ns
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    securityContext:
      privileged: true
```

```bash
kubectl apply -f insecure-pod.yaml
```

❌ Expected: Pod creation denied by PSA.

---

## ✅ Wrap-Up

- ✅ Ran containers as non-root
- ✅ Dropped capabilities
- ✅ Used read-only filesystems
- ✅ Applied seccomp and AppArmor profiles
- ✅ Used Pod Security Admission to block insecure pods

---