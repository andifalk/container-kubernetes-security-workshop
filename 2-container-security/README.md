# Container Security

## Prerequisites

* Linux VM with docker engine installed or docker desktop on other operating systems
* Basic knowledge of the docker commands

## Table of Contents

* [Linux Users, Groups and File Permissions](#linux-file-permissions)
* [Linux Capabilities and Dropping Privileges](#linux-capabilities)
* [Linux Namespaces](#linux-namespaces)
* [Linux Control Groups (cgroups) and Resource Limits](#linux-cgroups)
* [Seccomp and System Call Filtering](#seccomp-and-system-call-filtering)

## Understanding Linux Capabilities with Docker

### 🎯 Objective
Learn what Linux capabilities are, how containers use them, and how to manage them securely using Docker’s `--cap-add` and `--cap-drop` options.

---

### 🧰 Prerequisites

- Docker installed (`docker version` should work)
- Root or sudo access to run containers
- Tools: `capsh`, `iputils`, `libcap2-bin`

```bash
sudo apt update
sudo apt install libcap2-bin iputils-ping
```

---

### 🔹 Lab 1: View Capabilities Inside a Docker Container

#### Step 1: Start a default container

```bash
docker run --rm -it debian bash
```

#### Step 2: Install tools and check capabilities

```bash
apt update && apt install -y libcap2-bin
capsh --print
```

✅ **Expected:** Default set of capabilities shown (`cap_net_raw`, `cap_chown`, etc.)

---

### 🔹 Lab 2: Drop All Capabilities

#### Step 1: Run a container with no capabilities

```bash
docker run --rm -it --cap-drop=ALL debian bash
```

#### Step 2: Try actions that need privileges

```bash
apt update && apt install -y iputils-ping
ping -c 1 8.8.8.8
```

❌ **Expected:** `ping` fails because `cap_net_raw` is missing.

---

### 🔹 Lab 3: Add a Specific Capability

#### Step 1: Add back `CAP_NET_RAW`

```bash
docker run --rm -it --cap-drop=ALL --cap-add=NET_RAW debian bash
```

#### Step 2: Try `ping` again

```bash
apt update && apt install -y iputils-ping
ping -c 1 8.8.8.8
```

✅ **Expected:** `ping` works with `CAP_NET_RAW` restored.

---

### 🔹 Lab 4: Compare with Privileged Containers

#### Step 1: Run a privileged container

```bash
docker run --rm -it --privileged debian bash
```

```bash
capsh --print
```

✅ **Expected:** All capabilities are available. The container can do nearly anything the host can.

---

### 🔹 Lab 5: Test Filesystem Capabilities

#### Step 1: Try mounting inside a container

```bash
docker run --rm -it --cap-add=SYS_ADMIN debian bash
mkdir /mnt/test
mount -t tmpfs none /mnt/test
```

✅ **Expected:** Works with `CAP_SYS_ADMIN`; fails without it.

---

### 🔹 Lab 6: Inspect Running Containers

#### Step 1: Run a container in the background

```bash
docker run -d --cap-drop=ALL --name minimal alpine sleep 300
```

#### Step 2: Inspect the container

```bash
docker inspect minimal | grep Cap
```

✅ **Expected:** `CapDrop` should list all dropped caps, `CapAdd` should be empty.

---

### ✅ Wrap-Up

- You explored capabilities in Docker and saw how they restrict or allow actions.
- You ran containers with minimal privileges (`--cap-drop=ALL`).
- You added capabilities only as needed (`--cap-add=...`).
- You compared privilege escalation using `--privileged`.

---

## Container Escape with Docker (Educational Use Only)

> ⚠️ WARNING: These labs are for educational and defensive purposes only.  
> Run them **only in isolated, non-production environments** like VMs or lab systems.

### 🎯 Objective
Learn how insecure Docker configurations can lead to container escapes and host compromise — and how to prevent them.

---

### 🧰 Prerequisites

- A Linux VM (never your main host!)
- Docker installed
- Root access
- Tools: `gcc`, `make`, `procps`, `util-linux`

```bash
sudo apt update
sudo apt install gcc make procps util-linux -y
```

---

### 🔹 Lab 1: Host Mount Escape via `-v /:/host`

#### Step 1: Run a container with full host access

```bash
docker run -it --rm --privileged -v /:/host ubuntu bash
```

#### Step 2: Simulate host access

```bash
chroot /host bash
passwd root  # Example, do not use on real systems
```

✅ **Expected:** Full host access.

🧠 **Defense:** Never mount `/`, use strict volume policies.

---

### 🔹 Lab 2: Escape via `--privileged` and `/proc`

#### Step 1: Start a privileged container

```bash
docker run -it --rm --privileged ubuntu bash
```

#### Step 2: Enter host namespaces

```bash
nsenter --target 1 --mount --uts --ipc --net --pid
```

✅ **Expected:** You are now in the host’s namespaces.

🧠 **Defense:** Avoid `--privileged`, use minimal capabilities.

---

### 🔹 Lab 3: Simulated Kernel Escape via `CAP_SYS_ADMIN`

> This does **not** run a real exploit, but shows how dangerous capabilities can be.

#### Step 1: Launch container with high privileges

```bash
docker run -it --rm --cap-add=SYS_ADMIN ubuntu bash
```

#### Step 2: Try sensitive operations

```bash
mount -t proc proc /mnt
```

✅ **Expected:** You have low-level kernel control.

🧠 **Defense:** Drop all unnecessary capabilities.

---

### 🔹 Lab 4: Escape via Host Devices

#### Step 1: Run container with device access

```bash
docker run -it --rm --privileged --device /dev/kmsg ubuntu bash
```

#### Step 2: Write to kernel logs

```bash
echo "MESSAGE FROM CONTAINER" > /dev/kmsg
```

✅ **Expected:** Writes to host kernel logs.

🧠 **Defense:** Do not expose host devices unless absolutely needed.

---

### ✅ Wrap-Up

- **What you learned:**
    - How misconfigurations can lead to escapes
    - Common escape vectors: privileged mode, host mounts, capabilities
    - Defense-in-depth: drop caps, limit mounts, audit configs

- **Best practices:**
    - Never use `--privileged` unless critical
    - Use security profiles (AppArmor, seccomp)
    - Use namespaces and rootless containers
    - Audit Dockerfiles and runtime flags

---

## Secure Use of Docker Containers

### 🎯 Objective
Learn how to use Docker containers securely by applying best practices and reducing attack surfaces.

---

### 🧰 Prerequisites

- Docker installed
- Root or `sudo` access
- Terminal and Docker basics

---

### 🔹 Lab 1: Use Minimal Base Images

#### Step 1: Compare image sizes

```bash
docker pull ubuntu
docker pull alpine
docker images | grep -E 'ubuntu|alpine'
```

✅ **Expected:** Alpine is much smaller.

🧠 Smaller images = fewer vulnerabilities and faster deploys.

---

### 🔹 Lab 2: Run Containers as Non-Root User

#### Step 1: Create a Dockerfile

```Dockerfile
# Dockerfile
FROM alpine
RUN adduser -D secureuser
USER secureuser
CMD ["sh"]
```

#### Step 2: Build and run

```bash
docker build -t secureuser-container .
docker run --rm -it secureuser-container
```

✅ **Expected:** You are not root inside the container.

🧠 Principle of least privilege.

---

### 🔹 Lab 3: Drop Unnecessary Capabilities

#### Step 1: Run a minimal-capabilities container

```bash
docker run --rm -it --cap-drop=ALL alpine sh
```

✅ **Expected:** Privileged actions fail.

🧠 Capabilities like `CAP_NET_RAW` are dangerous if not required.

---

### 🔹 Lab 4: Use Seccomp to Restrict Syscalls

#### Step 1: Run with Docker’s default seccomp profile

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

### 🔹 Lab 5: Apply AppArmor Profiles

#### Step 1: View active profiles

```bash
sudo aa-status
```

#### Step 2: Run with a profile

```bash
docker run --rm -it --security-opt apparmor=unconfined alpine sh
```

🧠 Use `docker-default` or custom profiles for app restrictions.

---

### 🔹 Lab 6: Secure Secrets Handling

#### Step 1: Avoid baking secrets into images

**Bad (insecure):**

```Dockerfile
ENV DB_PASSWORD=supersecret
```

#### Step 2: Use runtime secrets

```bash
echo "supersecret" > db_password.txt
docker run --rm -it -v $(pwd)/db_password.txt:/run/secrets/db_pass alpine sh
```

✅ **Expected:** Secret is not in the image, only on runtime mount.

---

### 🔹 Lab 7: Scan Images for Vulnerabilities

#### Step 1: Use Trivy or Docker Scout

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

### ✅ Wrap-Up

- ✅ Minimal base images
- ✅ Run as non-root
- ✅ Drop unnecessary caps
- ✅ Use seccomp/AppArmor
- ✅ Handle secrets securely
- ✅ Scan images

---

## Next

[Next: Application Security](../step1-hello-spring-boot)
