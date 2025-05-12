# 🧪 Secure Secrets Management in Kubernetes

## 🎯 Objective

Learn how to manage secrets securely in Kubernetes using best practices, sealed secrets, and volume-based injection.

---

## 🧰 Prerequisites

- Kubernetes cluster
- `kubectl` configured
- Optional tools: `kubeseal`, `helm`, `vault`

---

## 🔹 Lab 1: Create a Secret and Mount It as a Volume

### Step 1: Create the secret

```bash
kubectl create secret generic db-creds \
  --from-literal=username=admin \
  --from-literal=password='S3cureP@ss'
```

### Step 2: Mount secret as volume

```yaml
# secretMountPod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-vol-demo
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    volumeMounts:
    - name: secret-vol
      mountPath: "/etc/creds"
      readOnly: true
  volumes:
  - name: secret-vol
    secret:
      secretName: db-creds
```

### Step 3: Run the pod

```bash
kubectl apply -f secretMountPod.yaml
```

### Step 4: Check the secrets

```bash
kubectl exec secret-vol-demo -- ls /etc/creds
kubectl exec secret-vol-demo -- cat /etc/creds/username
kubectl exec secret-vol-demo -- cat /etc/creds/password
```

✅ Secrets are mounted as read-only files.

---

## 🔹 Lab 2: Avoid Using Environment Variables (But Here's How)

### Step 1: Mount secret as environment variables

```yaml
# secretEnvPod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-demo
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "env & sleep 3600"]
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

### Step 2: Run the pod

```bash
kubectl apply -f secretEnvPod.yaml
```

### Step 3: Expose secrets

This command does not expose secrets any more:

```bash
kubectl describe pod secret-env-demo
```

But the log does reveal the secrets:

```bash
kubectl logs secret-env-demo
```

❗️ Secrets exposed in env vars can be viewed via `kubectl logs`.

---

## 🔹 Lab 3: Use Sealed Secrets (Bitnami)

**Problem**: "I can manage all my K8s config in git, except Secrets."

**Solution**: Encrypt your Secret into a SealedSecret, which is safe to store - even inside a public repository. The SealedSecret can be decrypted only by the controller running in the target cluster and nobody else (not even the original author) is able to obtain the original Secret from the SealedSecret.

> Requires [kubeseal](https://github.com/bitnami-labs/sealed-secrets) and corresponding Kubernetes controller setup

### Step 1: Install the controller

```bash
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
helm install sealed-secrets -n kube-system sealed-secrets/sealed-secrets
```

### Step 2: Install kubeseal client

Install via brew:

```bash
brew install kubeseal
```

### Step 3: Use kubeseal

Create secrets:

```bash
kubectl create secret generic mysecret --from-literal password=secret --dry-run=client -o json > mysecret.json
```

If you do a `cat mysecret.json`:

```json
{
    "kind": "Secret",
    "apiVersion": "v1",
    "metadata": {
        "name": "mysecret",
        "creationTimestamp": null
    },
    "data": {
        "password": "c2VjcmV0"
    }
}
```

You may decode the secret using `echo c2VjcmV0 | base64 -d`.

No let us seal the secrets:

```bash
kubeseal -f mysecret.json -w mysealedsecret.json
```

✅ Safe for GitOps workflows.

Read secrets again:

```bash
kubectl get secret mysecret
```

---

## 🔹 Lab 4: Use External Secret Managers

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
