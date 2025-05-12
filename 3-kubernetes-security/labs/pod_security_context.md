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

## 🔹 Lab 5: A really Pod Security Context

### Key Security Fields

| Field                          | Default | Recommended     | Reason |
|-------------------------------|---------|------------------|---------|
| `automountServiceAccountToken`| `true`  | `false`          | Avoid exposing API token |
| `hostNetwork`                 | `false` | `false`          | Prevent network namespace escape |
| `hostPID`                     | `false` | `false`          | Prevent process visibility |
| `hostIPC`                     | `false` | `false`          | Prevent shared memory abuse |
| `readOnlyRootFilesystem`      | N/A     | `true`           | Block writes to `/` |
| `seccompProfile`              | N/A     | `RuntimeDefault` | Prevent unwanted syscalls |

✅ Even if some defaults are `false`, **you should set them explicitly** to:

- Document intent
- Satisfy policies (e.g., PodSecurityAdmission `restricted`)
- Prevent accidental future changes

### Step 1: Define a really Secure Pod Specification

```yaml
# secure-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  automountServiceAccountToken: false
  hostNetwork: false
  hostPID: false
  hostIPC: false
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: secure-container
    image: busybox
    command: ["sh", "-c", "id && sleep 3600"]
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]
```

✅ This Pod enforces:

- Disable token mounting as the application does not need to call the Kubernetes API
- Ensures pod is not sharing the host network namespace
- Prevents access to host process namespace
- Prevents access to host IPC namespace
- Non-root user (UID 1000)
- Sets the group ID (GID 3000) for the primary group of the container process
- No privilege escalation
- No added capabilities
- Read-only root filesystem
- Ensures that the container has group access (GID 2000) to the mounted files
- Seccomp isolation

#### Step 2: Apply and Verify the Secure Pod

```bash
kubectl apply -f secure-pod.yaml
kubectl get pod secure-pod
kubectl exec secure-pod -- id
```

✅ The output should show non-root UID/GID.

#### Step 3: Test File System Protection

Try writing to the root filesystem:

```bash
kubectl exec secure-pod -- touch /testfile
```

❌ Expected: Operation not permitted (due to readOnlyRootFilesystem)

### ✅ Summary

- ✅ Deployed the most secure possible Pod using native features
- ✅ Blocked privilege escalation and system-level access
- ✅ Followed all best practices explicitly for compliance and clarity

## 🔹 Lab 6: Clean Up

```bash
kubectl delete pod non-root-pod readonly-pod drop-caps-pod no-priv-escalation pod secure-pod
```

---

## ✅ Wrap-Up

- ✅ Ran containers as non-root users
- ✅ Made root filesystems read-only
- ✅ Dropped unnecessary Linux capabilities
- ✅ Prevented privilege escalation
- ✅ Followed best practices for secure workload execution

---
