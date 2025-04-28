# 🧪 Using Pod Security Admission (PSA) and Security Levels in Kubernetes

## 🎯 Objective
Learn how to enforce security standards for Kubernetes workloads using Pod Security Admission (PSA) and Pod Security Standards.

---

## 🧰 Prerequisites

- Kubernetes 1.25+ cluster (PSA enabled by default)
- `kubectl` configured
- Cluster-admin privileges

---

## 🔹 Lab 1: Understand Pod Security Standards (PSS)

Kubernetes defines three levels:

- **Privileged**: No restrictions (dangerous for multi-tenant)
- **Baseline**: Minimally restrictive for common workloads
- **Restricted**: Heavily restricted, enforcing best practices

✅ PSA enforces these levels per namespace.

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

### Try in privileged namespace:

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
kubectl apply -f privileged-pod.yaml -n privileged-ns
```

✅ Works in `privileged-ns`.

### Try in baseline namespace:

```bash
kubectl apply -f privileged-pod.yaml -n baseline-ns
```

❌ Expected: Rejected by PSA.

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

Apply:

```bash
kubectl apply -f baseline-pod.yaml -n baseline-ns
kubectl apply -f baseline-pod.yaml -n restricted-ns
```

✅ Should work in both `baseline` and `restricted` namespaces.

---

## 🔹 Lab 5: Deploy a Non-Compliant Pod in Restricted Namespace

```yaml
# noncompliant-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: noncompliant-pod
spec:
  containers:
  - name: nginx
    image: nginx
```

Apply:

```bash
kubectl apply -f noncompliant-pod.yaml -n restricted-ns
```

❌ Should be **rejected** because required securityContext fields are missing.

---

## 🔹 Lab 6: View PodSecurity Violations

```bash
kubectl describe ns restricted-ns
```

✅ Shows any PSA violations for that namespace.

---

## 🔹 Lab 7: Clean Up

```bash
kubectl delete ns privileged-ns baseline-ns restricted-ns
```

---

## ✅ Wrap-Up

- ✅ Learned Kubernetes Pod Security Standards (Privileged, Baseline, Restricted)
- ✅ Enforced namespace-based pod security with PSA
- ✅ Practiced deploying compliant and non-compliant pods
- ✅ Strengthened workload isolation using native Kubernetes controls

---