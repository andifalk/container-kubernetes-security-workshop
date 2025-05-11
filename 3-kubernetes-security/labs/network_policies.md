# 🧪 Using Network Policies in Kubernetes

## 🎯 Objective

Learn how to secure pod-to-pod communication using Kubernetes Network Policies.

---

## 🧰 Prerequisites

- Kubernetes cluster (Minikube, kind, etc.)
- `kubectl` configured
- A CNI that supports NetworkPolicies (e.g., Calico, Cilium, or Weave)

### Test for a suitable CNI

```bash
kubectl get pods -n kube-system -o wide | grep -E 'calico|cilium|weave|flannel'
```

Note: The Kubernetes inside Docker Desktop does not enforce Network Policies! 

---

## 🔹 Lab 1: Set Up Test Environment in Current Namespace

### Step 1: Create two deployments

```bash
kubectl run client --image=busybox --command -- sleep 3600
kubectl run nginx --image=nginx
```

✅ Two pods: `client` and `nginx`.

### Step 2: Expose nginx as a service

```bash
kubectl expose pod nginx --port=80 --name=nginx-service
```

✅ Service: `nginx-service`.

---

## 🔹 Lab 2: Verify Open Communication

```bash
kubectl exec client -- wget --timeout=5 -qO- http://nginx-service
```

✅ Should return the nginx default HTML page.

---

## 🔹 Lab 3: Apply a Default-Deny Policy

```yaml
# default-deny.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

```bash
kubectl apply -f default-deny.yaml
```

✅ All ingress traffic is now blocked in the current namespace.

---

## 🔹 Lab 4: Allow Traffic to Nginx Only from Client Pod

```yaml
# allow-client.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-client
spec:
  podSelector:
    matchLabels:
      run: nginx
  ingress:
  - from:
    - podSelector:
        matchLabels:
          run: client
```

```bash
kubectl apply -f allow-client.yaml
```

✅ Only `client` can now reach `nginx`.

---

## 🔹 Lab 5: Test Access Again

```bash
kubectl exec client -- wget --timeout=5 -qO- http://nginx-service
```

✅ Should work.

Test from another pod:

```bash
kubectl run tester --image=busybox --command -- sleep 3600
kubectl exec client -- wget --timeout=5 -qO- http://nginx-service
```

❌ Should fail due to policy.

---

## 🔹 Lab 6: Clean Up

```bash
kubectl delete pod client nginx tester
kubectl delete networkpolicy deny-all allow-client
```

---

## ✅ Wrap-Up

- ✅ Created a default-deny NetworkPolicy
- ✅ Allowed access to a pod from a specific source
- ✅ Verified restricted and allowed traffic
- ✅ Practiced Kubernetes network security in the default namespace

---
