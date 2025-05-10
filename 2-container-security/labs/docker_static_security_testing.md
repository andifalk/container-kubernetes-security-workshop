# 🧪 Static Security Testing for Dockerfiles

## 🎯 Objective

Learn to perform static analysis on Dockerfiles to identify security issues before building images.

---

## 🧰 Prerequisites

- Dockerfile available for testing
- Tools: `hadolint`, `dockle`, `checkov`
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

## 🔹 Lab 2: Check with `hadolint` (Dockerfile Linter)

### Step 1: Install hadolint

Just install using `brew`:

```bash
brew install hadolint
```

or download the binary:

```bash
wget -O /usr/local/bin/hadolint https://github.com/hadolint/hadolint/releases/latest/download/hadolint-Linux-x86_64
chmod +x /usr/local/bin/hadolint
```

### Step 2: Check using hadolint

```bash
hadolint ./Dockerfile
```

✅ Detects issues like use of `latest`, insecure permissions, and `ADD` instead of `COPY`.

```bash
Dockerfile:1 DL3007 warning: Using latest is prone to errors if the image will ever update. Pin the version explicitly to a release tag
Dockerfile:3 DL3027 warning: Do not use apt as it is meant to be a end-user tool, use apt-get or apt-cache instead
Dockerfile:5 DL3020 error: Use COPY instead of ADD for files and folders
```

---

## 🔹 Lab 3: Check Image with `dockle`

Dockle checks container image best practices and Dockerfile structure.

### Step 1: Install dockle

```bash
brew install goodwithtech/r/dockle
```

### Step 2: Check with dockle

Build the container image using the Dockerfile from previous lab:

```bash
echo 'secret' > secret.txt
docker build -t check-container .
```

Now check the container image using dockle:

```bash
dockle check-container
```

✅ Report will include missing `HEALTHCHECK`, `USER`, writable files, etc.

```bash
FATAL - CIS-DI-0009: Use COPY instead of ADD in Dockerfile
 * Use COPY : ADD secret.txt /root/secret.txt # buildkit
FATAL - DKL-DI-0001: Avoid sudo command
 * Avoid sudo in container : RUN /bin/sh -c apt update && apt install -y curl sudo # buildkit
FATAL - DKL-DI-0005: Clear apt-get caches
 * Use 'rm -rf /var/lib/apt/lists' after 'apt-get install|update' : RUN /bin/sh -c apt update && apt install -y curl sudo # buildkit
WARN - CIS-DI-0001: Create a user for the container
 * Last user should not be root
WARN - DKL-DI-0006: Avoid latest tag
 * Avoid 'latest' tag
INFO - CIS-DI-0005: Enable Content trust for Docker
 * export DOCKER_CONTENT_TRUST=1 before docker pull/build
INFO - CIS-DI-0006: Add HEALTHCHECK instruction to the container image
 * not found HEALTHCHECK statement  
```

---

## 🔹 Lab 4: Scan Dockerfile with `checkov`

Checkov is a static code analysis tool for infrastructure security including Dockerfiles.

### Step 1: Install Checkov

Using `brew`:

```bash
brew install checkov
```

Or directly with pip:

```bash
pip install checkov
```

### Step 2: Run Checkov against a Dockerfile

```bash
checkov -f Dockerfile
```

✅ Identifies misconfigurations based on best practices and compliance checks.

```bash
dockerfile scan results:

Passed checks: 2, Failed checks: 5, Skipped checks: 0
<...>
```

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
