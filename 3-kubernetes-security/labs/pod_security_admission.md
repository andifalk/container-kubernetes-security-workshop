# 🧪 Using Pod Security Admission (PSA) and Security Levels in Kubernetes

## 🎯 Objective

Learn how to enforce security standards for Kubernetes workloads using Pod Security Admission (PSA) and Pod Security Standards.

---

## 🧰 Prerequisites

- Kubernetes 1.25+ cluster (PSA enabled by default)
- `kubectl` configured
- Privileges to set labels on user's namespace

---

## 🔹 Lab 1: Understand Pod Security Standards (PSS)

Kubernetes defines three levels:

- **Privileged**: No restrictions (dangerous for multi-tenant)
- **Baseline**: Minimally restrictive for common workloads
- **Restricted**: Heavily restricted, enforcing best practices

✅ PSA enforces these levels per namespace.

### Step 1: Check your current namespace for next labs

To get the name of your current namespace perform the following command and copy your namespace from there:

```bash
export MY_NS=$(kubectl config view --minify -o jsonpath={..namespace})
```

We will use this environment variable in the next labs.

---

## 🔹 Lab 2: Create Namespaces for Each Policy Level

```bash
kubectl create ns privileged-ns
kubectl create ns baseline-ns
kubectl create ns restricted-ns
```

Label namespaces:

```bash
kubectl label namespace privileged-ns   pod-security.kubernetes.io/enforce=privileged

kubectl label namespace baseline-ns   pod-security.kubernetes.io/enforce=baseline

kubectl label namespace restricted-ns   pod-security.kubernetes.io/enforce=restricted
```

✅ Labels control what pods are allowed.

---

## 🔹 Lab 3: Deploy a Privileged Pod

### Step 1: Try with privileged policy

First, label the current namespace as `privileged`:

```bash
kubectl label ns $MY_NS pod-security.kubernetes.io/enforce=privileged
```

You may check that the label has been added correctly using:

```bash
kubectl get ns $MY_NS --show-labels
```

```yaml
# privileged-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: privileged-pod
spec:
  containers:
  - name: nginx
    image: nginx
    securityContext:
      privileged: true
```

```bash
kubectl apply -f privileged-pod.yaml
```

✅ Works in `privileged-ns`.

#### Cleanup

Delete the pod for clean up:

```bash
kubectl delete pod privileged-pod --force=true
```

Also delete the label on the namespace:

```bash
kubectl label ns $MY_NS pod-security.kubernetes.io/enforce-
```

### Step 2: Try with baseline policy

First, label namespace with `baseline` policy

```bash
kubectl label ns $MY_NS pod-security.kubernetes.io/enforce=baseline
```

You may check that the label has been added correctly using:

```bash
kubectl get ns $MY_NS --show-labels
```

```bash
kubectl apply -f privileged-pod.yaml
```

❌ Expected: Rejected by PSA (`baseline` policy does not allow privileged containers).

#### Cleanup

Delete the pod for clean up:

```bash
kubectl delete pod privileged-pod --force=true
```

Also delete the label on the namespace:

```bash
kubectl label ns $MY_NS pod-security.kubernetes.io/enforce-
```

---

## 🔹 Lab 4: Deploy a Baseline-Compliant Pod

```yaml
# baseline-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: baseline-pod
spec:
  containers:
  - name: nginx
    image: nginx
    securityContext:
      allowPrivilegeEscalation: false
```

### Step 1: Try with baseline policy

First, label namespace with `baseline` policy

```bash
kubectl label ns $MY_NS pod-security.kubernetes.io/enforce=baseline
```

You may check that the label has been added correctly using:

```bash
kubectl get ns $MY_NS --show-labels
```

```bash
kubectl apply -f baseline-pod.yaml
```

✅ Works with `baseline` policy.

#### Cleanup

Delete the pod for clean up:

```bash
kubectl delete pod baseline-pod --force=true
```

Also delete the label on the namespace:

```bash
kubectl label ns $MY_NS pod-security.kubernetes.io/enforce-
```

### Step 2: Try with restricted policy

First, label namespace with `restricted` policy

```bash
kubectl label ns $MY_NS pod-security.kubernetes.io/enforce=restricted
```

You may check that the label has been added correctly using:

```bash
kubectl get ns $MY_NS --show-labels
```

```bash
kubectl apply -f baseline-pod.yaml
```

❌ Expected: Rejected by PSA (`restricted` policy requires more secure context).

#### Cleanup

Delete the pod for clean up:

```bash
kubectl delete pod baseline-pod --force=true
```

Also delete the label on the namespace:

```bash
kubectl label ns $MY_NS pod-security.kubernetes.io/enforce-
```

---

## 🔹 Lab 5: Deploy a Restricted-Compliant Pod

```yaml
# restricted-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: restricted-pod
spec:
  securityContext:
    runAsNonRoot: true
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: nginx
    image: nginxinc/nginx-unprivileged
    securityContext:
      capabilities:
            drop:
              - ALL
      allowPrivilegeEscalation: false
```

### Step 1: Try with baseline policy

First, label namespace with `baseline` policy

```bash
kubectl label ns $MY_NS pod-security.kubernetes.io/enforce=baseline
```

You may check that the label has been added correctly using:

```bash
kubectl get ns $MY_NS --show-labels
```

```bash
kubectl apply -f restricted-pod.yaml
```

✅ Still works with `baseline` policy.

#### Cleanup

Delete the pod for clean up:

```bash
kubectl delete pod restricted-pod --force=true
```

Also delete the label on the namespace:

```bash
kubectl label ns $MY_NS pod-security.kubernetes.io/enforce-
```

### Step 2: Try with restricted policy

First, label namespace with `restricted` policy

```bash
kubectl label ns $MY_NS pod-security.kubernetes.io/enforce=restricted
```

You may check that the label has been added correctly using:

```bash
kubectl get ns $MY_NS --show-labels
```

```bash
kubectl apply -f restricted-pod.yaml
```

✅ Now works as well with `restricted` policy.

#### Cleanup

Delete the pod for clean up:

```bash
kubectl delete pod restricted-pod --force=true
```

Also delete the label on the namespace:

```bash
kubectl label ns $MY_NS pod-security.kubernetes.io/enforce-
```

---

## 🔹 Lab 6: View PodSecurity Violations

```bash
kubectl describe ns $MY_NS
```

✅ Shows any PSA violations for that namespace.

---

## ✅ Wrap-Up

- ✅ Learned Kubernetes Pod Security Standards (Privileged, Baseline, Restricted)
- ✅ Enforced namespace-based pod security with PSA
- ✅ Practiced deploying compliant and non-compliant pods
- ✅ Strengthened workload isolation using native Kubernetes controls

---
