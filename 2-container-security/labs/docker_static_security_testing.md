# 🧪 Lab Series: Static Security Testing for Dockerfiles

## 🎯 Objective
Learn to perform static analysis on Dockerfiles to identify security issues before building images.

---

## 🧰 Prerequisites

- Dockerfile available for testing
- Tools: `hadolint`, `dockle`, `trivy`, `checkov`
- A system with Docker and internet access

---

## 🔹 Lab 1: Create a Sample Insecure Dockerfile

```Dockerfile
# Dockerfile
FROM ubuntu:latest

RUN apt update && apt install -y curl sudo

ADD secret.txt /root/secret.txt

RUN chmod 777 /root/secret.txt

CMD ["bash"]
```

Save it as `Dockerfile`.

---

## 🔹 Lab 2: Install `hadolint` (Dockerfile Linter)

### On Ubuntu:

```bash
sudo apt install hadolint
```

### Or via Docker:

```bash
docker run --rm -i hadolint/hadolint < Dockerfile
```

✅ Detects issues like use of `latest`, insecure permissions, and `ADD` instead of `COPY`.

---

## 🔹 Lab 3: Install and Use `dockle`

Dockle checks container image best practices and Dockerfile structure.

```bash
docker pull goodwithtech/dockle

docker run --rm -v /var/run/docker.sock:/var/run/docker.sock goodwithtech/dockle docker.io/library/ubuntu
```

✅ Report will include missing `HEALTHCHECK`, `USER`, writable files, etc.

---

## 🔹 Lab 4: Scan Dockerfile with `checkov`

Checkov is a static code analysis tool for infrastructure security including Dockerfiles.

### Install Checkov:

```bash
pip install checkov
```

### Run Checkov against a Dockerfile:

```bash
checkov -f Dockerfile
```

✅ Identifies misconfigurations based on best practices and compliance checks.

---

## 🔹 Lab 5: Use GitHub Actions for CI Security Testing (Optional)

```yaml
# .github/workflows/dockerfile-lint.yml
name: Dockerfile Lint

on: [push]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Run hadolint
      uses: hadolint/hadolint-action@v2
      with:
        dockerfile: Dockerfile
    - name: Check Dockerfile with Checkov
      run: |
        pip install checkov
        checkov -f Dockerfile
```

✅ Adds multi-tool static testing to your pipeline.

---

## ✅ Wrap-Up

- ✅ Used `hadolint`, `dockle`, and `checkov` for Dockerfile scanning
- ✅ Detected vulnerabilities and bad practices
- ✅ Integrated multiple tools into a CI pipeline

---