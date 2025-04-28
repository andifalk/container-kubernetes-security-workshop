# 🧪 Lab Series: Secure Use of Docker Containers

## 🎯 Objective
Learn how to use Docker containers securely by applying best practices and reducing attack surfaces.

---

## 🧰 Prerequisites

- Docker installed
- Root or `sudo` access
- Terminal and Docker basics

---

## 🔹 Lab 1: Use Minimal Base Images

### Step 1: Compare image sizes

```bash
docker pull ubuntu
docker pull alpine
docker images | grep -E 'ubuntu|alpine'
```

✅ **Expected:** Alpine is much smaller.

🧠 Smaller images = fewer vulnerabilities and faster deploys.

---

## 🔹 Lab 2: Run Containers as Root User

### Step 1: Create a Dockerfile

```Dockerfile
# Dockerfile
FROM alpine
CMD ["sh"]
```

### Step 2: Build and run

```bash
docker build -t insecureuser-container .
docker run --rm -it insecureuser-container
```

✅ **Expected:** You are root inside the container.

🧠 Violates the principle of `Least Privilege`.

---

## 🔹 Lab 3: Run Containers as Non-Root User

### Step 1: Create a Dockerfile

```Dockerfile
# Dockerfile
FROM alpine
RUN adduser -D secureuser
USER secureuser
CMD ["sh"]
```

### Step 2: Build and run

```bash
docker build -t secureuser-container .
docker run --rm -it secureuser-container
```

✅ **Expected:** You are not root inside the container.

🧠 Follows the principle of `Least Privilege`.

---

## 🔹 Lab 4: Drop Unnecessary Capabilities

### Step 1: Run a minimal-capabilities container

```bash
docker run --rm -it --cap-drop=ALL alpine sh
```

✅ **Expected:** Privileged actions fail.

🧠 Capabilities like `CAP_NET_RAW` are dangerous if not required.

---

## 🔹 Lab 5: Use Seccomp to Restrict Syscalls

### Step 1: Run with Docker’s default seccomp profile

```bash
docker run --rm -it --security-opt seccomp=default.json alpine sh
```

Try:

```bash
apk add strace
strace -c ls
```

🧠 Seccomp blocks risky syscalls.

---

## 🔹 Lab 6: Apply AppArmor Profiles

### Step 1: View active profiles

```bash
sudo aa-status
```

### Step 2: Run with a profile

```bash
docker run --rm -it --security-opt apparmor=unconfined alpine sh
```

🧠 Use `docker-default` or custom profiles for app restrictions.

---

## 🔹 Lab 7: Secure Secrets Handling

### Step 1: Avoid baking secrets into images

**Bad (insecure):**

```Dockerfile
ENV DB_PASSWORD=supersecret
```

### Step 2: Use runtime secrets

```bash
echo "supersecret" > db_password.txt
docker run --rm -it -v $(pwd)/db_password.txt:/run/secrets/db_pass alpine sh
```

✅ **Expected:** Secret is not in the image, only on runtime mount.

---

## 🔹 Lab 8: Scan Images for Vulnerabilities

### Step 1: Use Trivy or Docker Scout

Install Trivy:

```bash
brew install aquasecurity/trivy/trivy
```

Scan:

```bash
trivy image alpine
```

✅ **Expected:** CVE results and recommendations.

🧠 Scan early, scan often.

---

## ✅ Wrap-Up

- ✅ Minimal base images
- ✅ Run as non-root
- ✅ Drop unnecessary caps
- ✅ Use seccomp/AppArmor
- ✅ Handle secrets securely
- ✅ Scan images

---