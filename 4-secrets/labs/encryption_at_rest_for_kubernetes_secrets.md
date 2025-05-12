# 🧪 Enabling Encryption at Rest for Kubernetes Secrets

## 🎯 Objective

Learn how to encrypt Kubernetes Secrets at rest in etcd using AES-CBC encryption.

---

## 🧰 Prerequisites

- Kubernetes cluster with admin access
- SSH access to the control plane (master node)
- `kubectl` configured

---

## ⚠️ Important

Always backup your cluster before modifying critical configurations!

---

## 🔹 Lab 1: SSH into the Control Plane

```bash
ssh user@<control-plane-node-ip>
```

✅ You need access to `/etc/kubernetes/manifests/kube-apiserver.yaml`.

---

## 🔹 Lab 2: Create an Encryption Configuration File

Generate a 32-byte base64 key:

```bash
head -c 32 /dev/urandom | base64
```

Save it securely. Then create `/etc/kubernetes/encryption-config.yaml`:

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
        secret: <paste-your-base64-key-here>
  - identity: {}
```

✅ This configuration encrypts secrets using AES-CBC encryption.

---

## 🔹 Lab 3: Edit kube-apiserver to Use Encryption

Edit `/etc/kubernetes/manifests/kube-apiserver.yaml` and add the following flag under `spec.containers.command`:

```bash
--encryption-provider-config=/etc/kubernetes/encryption-config.yaml
```

Example fragment:

```yaml
- --advertise-address=...
- --encryption-provider-config=/etc/kubernetes/encryption-config.yaml
```

✅ Save and exit.

Kubelet will automatically restart the `kube-apiserver` pod with the new config.

---

## 🔹 Lab 4: Verify Encryption Provider is Active

Create a new secret:

```bash
kubectl create secret generic encrypted-secret --from-literal=mykey=myvalue
```

✅ Now secrets should be encrypted in etcd!

---

## 🔹 Lab 5: (Optional) Simulate Inspecting etcd Storage

If you can access etcd directly (dangerous in production!):

```bash
ETCDCTL_API=3 etcdctl get /registry/secrets/default/encrypted-secret --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key --cacert /etc/kubernetes/pki/etcd/ca.crt
```

✅ Instead of base64, you'll see unreadable ciphertext.

---

## 🔹 Lab 6: Clean Up

```bash
kubectl delete secret encrypted-secret
```

---

## ✅ Wrap-Up

- ✅ Generated an encryption key and configuration
- ✅ Modified the kube-apiserver to enable encryption
- ✅ Verified secrets are encrypted at rest in etcd
- ✅ Improved cluster security by protecting sensitive data

---
