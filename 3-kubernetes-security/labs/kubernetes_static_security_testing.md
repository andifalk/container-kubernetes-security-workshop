# 🧪 Lab Series: Static Security Testing for Kubernetes Workloads

## 🎯 Objective

Use tools like `kube-score`, `kubescape`, and `checkov` to statically analyze Kubernetes manifests for security and compliance issues.

---

## 🧰 Prerequisites

- Kubernetes manifest files (YAML)
- Tools: `kube-score`, `kubescape`, `checkov`
- A system with internet access

---

## 🔹 Lab 1: Create a Sample Kubernetes Deployment

```yaml
# insecure-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: insecure-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: insecure
  template:
    metadata:
      labels:
        app: insecure
    spec:
      containers:
      - name: app
        image: nginx:latest
        ports:
        - containerPort: 80
```

---

## 🔹 Lab 2: Install and Use `kube-score`

### Install:

```bash
brew install kube-score  # macOS
sudo snap install kube-score  # Linux
```

### Run against manifest:

```bash
kube-score score insecure-deployment.yaml
```

✅ Detects missing `readOnlyRootFilesystem`, `runAsNonRoot`, and more.

---

## 🔹 Lab 3: Install and Use `kubescape`

### Install:

```bash
curl -s https://raw.githubusercontent.com/kubescape/kubescape/master/install.sh | /bin/bash
```

### Run:

```bash
kubescape scan framework nsa -f insecure-deployment.yaml
```

✅ Scans manifest against NSA, MITRE, and custom frameworks.

---

## 🔹 Lab 4: Install and Use `checkov`

### Install:

```bash
pip install checkov
```

### Run:

```bash
checkov -f insecure-deployment.yaml
```

✅ Reports insecure configurations, missing security context, and risky settings.

---

## 🔹 Lab 5: Automate in CI/CD

### GitHub Actions Example:

```yaml
# .github/workflows/k8s-static-checks.yml
name: Kubernetes Static Checks

on: [push]

jobs:
  static-analysis:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Run Checkov
      run: |
        pip install checkov
        checkov -f insecure-deployment.yaml
    - name: Run Kube-Score
      run: |
        curl -L https://github.com/zegl/kube-score/releases/latest/download/kube-score-linux-amd64 -o kube-score
        chmod +x kube-score
        ./kube-score score insecure-deployment.yaml
```

---

## ✅ Wrap-Up

- ✅ Used `kube-score`, `kubescape`, and `checkov` to statically analyze workloads
- ✅ Identified common misconfigurations before deployment
- ✅ Integrated security checks into CI pipelines

---
