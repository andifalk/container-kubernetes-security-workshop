# Secure Secrets Management in Kubernetes

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
- Agent Injector

### Using HashiCorp Vault as an External Secret Manager in Kubernetes

#### 🎯 Objective
Learn how to integrate HashiCorp Vault with Kubernetes to inject secrets securely into Pods using Vault's Kubernetes auth and agent injector.

---

#### 🧰 Prerequisites

- Kubernetes cluster (minikube, kind, GKE, etc.)
- Helm installed
- Vault CLI installed
- `kubectl` configured
- Admin privileges in cluster

---

#### 🔹 Step 1: Install Vault in Kubernetes with Helm

```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
helm install vault hashicorp/vault --set "server.dev.enabled=true"
```

✅ Dev mode Vault pod (`vault-0`) runs.

---

#### 🔹 Step 2: Check Vault Status (Dev mode auto-unsealed)

```bash
kubectl exec -it vault-0 -- vault status
```

✅ Vault is initialized and unsealed.

---

#### 🔹 Step 3: Enable Kubernetes Auth Method

```bash
kubectl exec -it vault-0 -- /bin/sh
vault auth enable kubernetes

vault write auth/kubernetes/config \
  token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
  kubernetes_host="https://${KUBERNETES_PORT_443_TCP_ADDR}:443"
```

✅ Vault is configured to verify Kubernetes tokens.

---

#### 🔹 Step 4: Create Vault Policy and Role

```bash
vault policy write myapp - <<EOF
path "secret/data/myapp/*" {
  capabilities = ["read"]
}
EOF
```

```bash
vault write auth/kubernetes/role/myapp \
  bound_service_account_names=vault-sa \
  bound_service_account_namespaces=default \
  policies=myapp \
  ttl=24h
```

---

#### 🔹 Step 5: Store a Secret in Vault

```bash
vault kv put secret/myapp/config username="demo" password="vaultpass"
```

✅ Secret is stored at `secret/data/myapp/config`.

---

#### 🔹 Step 6: Deploy Vault Agent Injector

```bash
helm upgrade --install vault hashicorp/vault \
  --set "injector.enabled=true" \
  --set "server.dev.enabled=true"
```

✅ Vault Injector is deployed.

---

#### 🔹 Step 7: Inject Secrets into a Pod

```bash
kubectl create serviceaccount vault-sa
```

```yaml
# vault-inject-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: vault-demo
  annotations:
    vault.hashicorp.com/agent-inject: "true"
    vault.hashicorp.com/role: "myapp"
    vault.hashicorp.com/agent-inject-secret-config.txt: "secret/data/myapp/config"
spec:
  serviceAccountName: vault-sa
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "cat /vault/secrets/config.txt && sleep 3600"]
```

```bash
kubectl apply -f vault-inject-pod.yaml
kubectl logs vault-demo
```

✅ Vault Agent injects secrets as files into the pod.

---

#### ✅ Wrap-Up

🧠 Fetch secrets at runtime, not stored in etcd.

- ✅ Installed Vault via Helm
- ✅ Enabled Kubernetes auth
- ✅ Stored secrets securely
- ✅ Injected secrets into containers using Vault Agent Injector

---

## ✅ Wrap-Up

- ✅ Use volume mounts over env vars
- ✅ Don’t log or echo secrets
- ✅ Rotate secrets regularly
- ✅ Use SealedSecrets for Git
- ✅ Use external secret managers for runtime safety

---