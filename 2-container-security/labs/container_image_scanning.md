# 🧪 Scanning Container Images Using Trivy

## 🎯 Objective

Use Aqua Security's Trivy to scan container images for vulnerabilities, misconfigurations, and exposed secrets.

---

## 🧰 Prerequisites

- Docker installed
- Internet access
- Trivy installed (`brew install trivy` or `sudo apt install trivy`)

---

## 🔹 Lab 1: Install Trivy

```bash
brew install aquasecurity/trivy/trivy
```

---

## 🔹 Lab 2: Scan a Public Image

```bash
trivy image nginx:latest
```

✅ Trivy outputs known vulnerabilities (CVEs), their severity, and fixed versions.

---

## 🔹 Lab 3: Scan a Local Image

### Step 1: Build your own image

Create the `Dockerfile`:

```Dockerfile
# Dockerfile
FROM ubuntu:latest
RUN apt update && apt install -y curl
```

Build the container image:

```bash
docker build -t my-insecure-image .
```

### Step 2: Scan the local image

```bash
trivy image my-insecure-image
```

✅ Scan custom images before pushing to production.

---

## 🔹 Lab 4: Scan for Secrets and Misconfigurations

```bash
trivy image --scanners vuln,misconfig,secret my-insecure-image
```

✅ Trivy also finds secrets and insecure configs in layers.

---

## 🔹 Lab 5: Generate Reports in Different Formats

### Step 1: Use JSON output

```bash
trivy image -f json -o trivy-report.json my-insecure-image
```

### Step 2: Use CycloneDx output (for SBOM)

```bash
trivy image --scanners vuln -f cyclonedx -o trivy-sbom.json my-insecure-image
```

✅ Useful for automation in CI pipelines and to report SBOM files as part of supply chain security.

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
