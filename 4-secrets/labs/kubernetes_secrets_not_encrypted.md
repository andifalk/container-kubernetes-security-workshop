# 🧪 Demonstrating Kubernetes Secrets Are Base64 Encoded (Not Encrypted)

## 🎯 Objective

Show that Kubernetes Secrets are only base64-encoded by default and not securely encrypted at rest without additional configuration.

---

## 🧰 Prerequisites

- Kubernetes cluster
- `kubectl` configured
- Admin access to the cluster

---

## 🔹 Lab 1: Create a Simple Kubernetes Secret

```bash
kubectl create secret generic demo-secret   --from-literal=username=admin   --from-literal=password='SuperSecret123'
```

✅ A secret named `demo-secret` is created.

---

## 🔹 Lab 2: View the Secret Using `kubectl`

```bash
kubectl get secret demo-secret -o yaml
```

✅ Output:

```yaml
apiVersion: v1
data:
  password: U3VwZXJTZWNyZXQxMjM=
  username: YWRtaW4=
kind: Secret
metadata:
  name: demo-secret
type: Opaque
```

🔍 The `password` and `username` fields are **base64 encoded**, not encrypted.

---

## 🔹 Lab 3: Decode the Secret Locally

Decode the username:

```bash
echo "YWRtaW4=" | base64 --decode
```

Decode the password:

```bash
echo "U3VwZXJTZWNyZXQxMjM=" | base64 --decode
```

✅ You retrieve the original credentials easily with `base64`.

---

## 🔹 Lab 4: Check How Secrets Are Stored in `etcd`

⚠️ **Warning**: Direct access to etcd is needed for real inspection; we simulate this.

### Simulate reading etcd storage

If you had direct etcd access (example from `/var/lib/etcd`), you'd see that the stored secrets are still base64-encoded.

This proves that **without encryption at rest enabled**, secrets are only encoded — **not securely encrypted**.

---

## 🔹 Lab 5: How to Enable Secret Encryption (for Real Protection)

Edit the Kubernetes API server manifest (usually `/etc/kubernetes/manifests/kube-apiserver.yaml`) to include:

```bash
--encryption-provider-config=/etc/kubernetes/encryption-config.yaml
```

Sample `encryption-config.yaml`:

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
- resources:
  - secrets
  providers:
  - aescbc:
      keys:
      - name: key1
        secret: <base64-encoded-aes-key>
  - identity: {}
```

✅ This encrypts Secrets at rest using AES encryption.

---

## 🔹 Lab 6: Clean Up

```bash
kubectl delete secret demo-secret
```

---

## ✅ Wrap-Up

- ✅ Created and inspected a Kubernetes Secret
- ✅ Verified Secrets are only base64-encoded by default
- ✅ Learned how to enable true encryption for Secrets at rest

---
