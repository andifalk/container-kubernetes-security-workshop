# 🧪 Secure Use of Containers in Kubernetes

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
# non-root-pod.yaml
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
# drop-caps-pod.yaml
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
# readonly-pod.yaml
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
# seccomp-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: seccomp-demo
spec:
  securityContext:
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    securityContext:
      allowPrivilegeEscalation: false
```

```bash
kubectl apply -f seccomp-pod.yaml
```

✅ Expected: Pod uses restricted syscall set.

---

## 🔹 Lab 5: Enforce Policies with Pod Security Admission (PSA)

### Step 1: Create secure namespace

Create a restricted policy in your own namespace.

First get the name of your namespace:

```bash
kubectl config get-contexts
```

If your namespace would be named `afa01-vm-0-ns` then this is the command for this (replace with your own namespace first)

```bash
kubectl label ns afa01-vm-0-ns pod-security.kubernetes.io/enforce=restricted
```

You may check that the label has been added correctly using:

```bash
kubectl get ns afa01-vm-0-ns --show-labels
```

### Step 2: Try applying an insecure pod

```yaml
# insecure-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: insecure
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

### Step 3: Clean up and remove label again

To be able to create unrestricted pods again just remove the label

```bash
kubectl label ns afa01-vm-0-ns pod-security.kubernetes.io/enforce-
```

You may check that the label has been removed correctly using:

```bash
kubectl get ns afa01-vm-0-ns --show-labels
```

---

## ✅ Wrap-Up

- ✅ Ran containers as non-root
- ✅ Dropped capabilities
- ✅ Used read-only filesystems
- ✅ Applied seccomp and AppArmor profiles
- ✅ Used Pod Security Admission to block insecure pods

---
