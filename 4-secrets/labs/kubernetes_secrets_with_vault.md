# 🧪 Using an external HashiCorp Vault Instance with Kubernetes

## 🎯 Objective
Use an existing, externally managed HashiCorp Vault instance to inject secrets securely into Kubernetes pods.

---

## 🧰 Prerequisites

- A running Vault server outside Kubernetes
- Kubernetes cluster access
- Vault CLI (`vault`) installed
- Kubernetes `kubectl` configured

---

## 🔹 Lab 1: Prepare Vault for Kubernetes Authentication

### Step 1: Enable Kubernetes auth in Vault

```bash
vault auth enable kubernetes
```

### Step 2: Configure Kubernetes auth with your cluster details

```bash
vault write auth/kubernetes/config   token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)"   kubernetes_host="https://<KUBERNETES_API_ENDPOINT>"   kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
```

Replace `<KUBERNETES_API_ENDPOINT>` with your cluster's API server address.

---

## 🔹 Lab 2: Create Vault Policies and Roles

### Step 1: Create a policy

```bash
vault policy write myapp-policy - <<EOF
path "secret/data/myapp/*" {
  capabilities = ["read"]
}
EOF
```

### Step 2: Create a role for the Kubernetes service account

```bash
vault write auth/kubernetes/role/myapp-role   bound_service_account_names=vault-sa   bound_service_account_namespaces=default   policies=myapp-policy   ttl=24h
```

---

## 🔹 Lab 3: Store a Secret in Vault

```bash
vault kv put secret/myapp/config username="admin" password="s3cr3t"
```

---

## 🔹 Lab 4: Deploy Vault Agent Injector (if needed)

If using Vault Agent Injector:

```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
helm install vault-agent-injector hashicorp/vault   --set injector.externalVaultAddr="http://<VAULT_SERVER_ADDRESS>:8200"
```

✅ Connects to external Vault instead of in-cluster Vault.

---

## 🔹 Lab 5: Create Kubernetes ServiceAccount

```bash
kubectl create serviceaccount vault-sa
```

---

## 🔹 Lab 6: Annotate Pod to Use External Vault

```yaml
# vault-inject-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: vault-demo
  annotations:
    vault.hashicorp.com/agent-inject: "true"
    vault.hashicorp.com/role: "myapp-role"
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
```

✅ Secret from external Vault is injected as a file inside the pod.

---

## 🔹 Lab 7: Verify Secret Injection

```bash
kubectl logs vault-demo
```

✅ Should print Vault secret contents from `/vault/secrets/config.txt`.

---

## ✅ Wrap-Up

- ✅ Integrated Kubernetes with an external Vault server
- ✅ Used Kubernetes auth method for secure authentication
- ✅ Injected secrets into pods using Vault Agent
- ✅ No in-cluster Vault server needed

---