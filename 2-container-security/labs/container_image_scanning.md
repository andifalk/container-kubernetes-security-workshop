# 🧪 Lab Series: Scanning Container Images Using Trivy

## 🎯 Objective
Use Aqua Security's Trivy to scan container images for vulnerabilities, misconfigurations, and exposed secrets.

---

## 🧰 Prerequisites

- Docker installed
- Internet access
- Trivy installed (`brew install trivy` or `sudo apt install trivy`)

---

## 🔹 Lab 1: Install Trivy

### On Ubuntu:

```bash
sudo apt install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo gpg --dearmor -o /usr/share/keyrings/trivy.gpg
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb stable main" | sudo tee /etc/apt/sources.list.d/trivy.list
sudo apt update
sudo apt install trivy
```

---

## 🔹 Lab 2: Scan a Public Image

```bash
trivy image nginx:latest
```

✅ Trivy outputs known vulnerabilities (CVEs), their severity, and fixed versions.

---

## 🔹 Lab 3: Scan a Local Image

### Build your own image:

```Dockerfile
# Dockerfile
FROM ubuntu:latest
RUN apt update && apt install -y curl
```

```bash
docker build -t my-insecure-image .
trivy image my-insecure-image
```

✅ Scan custom images before pushing to production.

---

## 🔹 Lab 4: Scan for Secrets and Misconfigurations

```bash
trivy image --scanners vuln,config,secret my-insecure-image
```

✅ Trivy also finds secrets and insecure configs in layers.

---

## 🔹 Lab 5: Generate Reports in Different Formats

### JSON output:

```bash
trivy image -f json -o trivy-report.json my-insecure-image
```

### HTML output (requires Trivy >= 0.50):

```bash
trivy image -f template --template "@/contrib/html.tpl" -o trivy-report.html my-insecure-image
```

✅ Useful for automation and CI pipelines.

---

## 🔹 Lab 6: Integrate with CI/CD (GitHub Actions) — Optional

```yaml
# .github/workflows/trivy-scan.yml
name: Trivy Scan

on: [push]

jobs:
  trivy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Run Trivy Scan
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: 'my-insecure-image'
        format: 'table'
        ignore-unfixed: true
```

---

## ✅ Wrap-Up

- ✅ Installed and used Trivy to scan public and private images
- ✅ Detected CVEs, misconfigurations, and secrets
- ✅ Generated reports for review and CI use

---