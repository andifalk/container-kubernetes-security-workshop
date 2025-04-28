# 🧪 Secure Secrets Management in Kubernetes

## 🎯 Objective
Learn how to manage secrets securely in Kubernetes using best practices, sealed secrets, and volume-based injection.

---

## 🧰 Prerequisites

- Kubernetes cluster (Minikube, kind, GKE, etc.)
- `kubectl` configured
- Optional tools: `kubeseal`, `helm`, `vault`

---

## 🔹 Lab 1: Create a Secret and Mount It as a Volume

```bash
kubectl create secret generic db-creds \
  --from-literal=username=admin \
  --from-literal=password='S3cureP@ss'
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-vol-demo
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "cat /etc/creds/* && sleep 3600"]
    volumeMounts:
    - name: secret-vol
      mountPath: "/etc/creds"
      readOnly: true
  volumes:
  - name: secret-vol
    secret:
      secretName: db-creds
```

✅ Secrets are mounted as read-only files.

---

## 🔹 Lab 2: Avoid Using Environment Variables (But Here's How)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-demo
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "env && sleep 3600"]
    env:
    - name: DB_USER
      valueFrom:
        secretKeyRef:
          name: db-creds
          key: username
    - name: DB_PASS
      valueFrom:
        secretKeyRef:
          name: db-creds
          key: password
```

❗️ Secrets exposed in env vars can be viewed via `kubectl describe`.

---

## 🔹 Lab 3: Avoid Leaking Secrets in Logs and CLI

### Safe inspection:

```bash
kubectl get secret db-creds -o json | jq '.data.username' | base64 -d
```

❌ Avoid echoing secrets in commands or logs.

---

## 🔹 Lab 4: Rotate Secrets Safely

Update and re-apply:

```bash
kubectl create secret generic db-creds \
  --from-literal=username=admin \
  --from-literal=password='N3wS3cr3t' \
  --dry-run=client -o yaml | kubectl apply -f -
```

✅ Mount-based secrets update on pod restart.

---

## 🔹 Lab 5: Use Sealed Secrets (Bitnami)

> Requires `kubeseal` and controller setup

```bash
kubeseal --cert pub-cert.pem < secret.yaml > sealedsecret.yaml
kubectl apply -f sealedsecret.yaml
```

✅ Safe for GitOps workflows.

---

## 🔹 Lab 6: Use External Secret Managers

- **HashiCorp Vault**
- **AWS Secrets Manager**
- **Azure Key Vault**

Use tools like:
- External Secrets Operator
- CSI Driver Secrets Store

🧠 Fetch secrets at runtime, not stored in etcd.

---

## ✅ Wrap-Up

- ✅ Use volume mounts over env vars
- ✅ Don’t log or echo secrets
- ✅ Rotate secrets regularly
- ✅ Use SealedSecrets for Git
- ✅ Use external secret managers for runtime safety

---