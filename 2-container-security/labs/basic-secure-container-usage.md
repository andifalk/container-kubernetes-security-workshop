# 🧪 Secure Use of Docker Containers

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

```bash
ubuntu       latest    a0e45e2ce6e6   12 days ago    78.1MB
alpine       latest    aded1e1a5b37   2 months ago   7.83MB
```

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

### Step 1: Create a more secure Dockerfile

```Dockerfile
# Dockerfile
FROM alpine
RUN adduser -D secureuser
USER secureuser
CMD ["sh"]
```

### Step 2: Build and run again

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

Docker now runs by default using a default seccomp profile. By default this profile allows executing the potentially dangerous `strace` command.

First, get the default seccomp profile from github:

```bash
curl -O https://raw.githubusercontent.com/moby/moby/master/profiles/seccomp/default.json
```

Open the file with `nano default.json` and search for the entry `ptrace` (using Ctrl-W) and delete the corresponding line and exit nano using `Ctrl-X` and type `Y` to store the file.

Now run the following docker container using the modified seccomp profile:

```bash
docker run --rm -it --security-opt seccomp=default.json alpine sh
```

Inside the container try:

```bash
apk add strace
strace -c ls
```

🧠 Seccomp blocks risky syscalls.

```bash
strace: test_ptrace_get_syscall_info: PTRACE_TRACEME: Operation not permitted
strace: ptrace(PTRACE_TRACEME, ...): Operation not permitted
strace: PTRACE_SETOPTIONS: Operation not permitted
strace: cleanup: waitpid(-1, __WALL): No child process
```

See more details at https://docs.docker.com/engine/security/seccomp

---

## 🔹 Lab 6: Apply AppArmor Profiles

### Step 1: View active profiles

```bash
sudo aa-status
```

### Step 2: Run with a profile

When you run a container, it uses the `docker-default` AppArmor policy unless you override it with the security-opt option.

You can also specify the profile explicitly:

```bash
docker run --rm -it --security-opt apparmor=docker-default alpine sh
```

🧠 Use `docker-default` or custom profiles for app restrictions.

See more details at https://docs.docker.com/engine/security/apparmor

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

Check for secret in the container located at `/run/secrets/db_pass`.

Docker provides native secrets support only in swarm mode. See more details at https://docs.docker.com/engine/swarm/secrets. We will look more in depth into secrets in the correspondig section. 

---

## 🔹 Lab 8: Scan Images for Vulnerabilities

### Step 1: Use Trivy or Docker Scout

Install Trivy:

```bash
brew install aquasecurity/trivy/trivy
```

### Step 2: Scan the Alpine image

```bash
trivy image alpine
```

### Step 3: Scan the Ubuntu image

```bash
trivy image ubuntu
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
