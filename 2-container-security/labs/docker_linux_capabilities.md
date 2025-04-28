# 🧪 Understanding Linux Capabilities with Docker

## 🎯 Objective
Learn what Linux capabilities are, how containers use them, and how to manage them securely using Docker’s `--cap-add` and `--cap-drop` options.

---

## 🧰 Prerequisites

- Docker installed (`docker version` should work)
- Root or sudo access to run containers
- Tools: `capsh`, `iputils`, `libcap2-bin`

```bash
sudo apt update
sudo apt install libcap2-bin iputils-ping
```

---

## 🔹 Lab 1: View Capabilities Inside a Docker Container

### Step 1: Start a default container

```bash
docker run --rm -it debian bash
```

### Step 2: Install tools and check capabilities

```bash
apt update && apt install -y libcap2-bin
capsh --print
```

✅ **Expected:** Default set of capabilities shown (`cap_net_raw`, `cap_chown`, etc.)

---

## 🔹 Lab 2: Drop All Capabilities

### Step 1: Run a container with no capabilities

```bash
docker run --rm -it --cap-drop=ALL debian bash
```

### Step 2: Try actions that need privileges

```bash
apt update && apt install -y iputils-ping
ping -c 1 8.8.8.8
```

❌ **Expected:** `ping` fails because `cap_net_raw` is missing.

---

## 🔹 Lab 3: Add a Specific Capability

### Step 1: Add back `CAP_NET_RAW`

```bash
docker run --rm -it --cap-drop=ALL --cap-add=NET_RAW debian bash
```

### Step 2: Try `ping` again

```bash
apt update && apt install -y iputils-ping
ping -c 1 8.8.8.8
```

✅ **Expected:** `ping` works with `CAP_NET_RAW` restored.

---

## 🔹 Lab 4: Compare with Privileged Containers

### Step 1: Run a privileged container

```bash
docker run --rm -it --privileged debian bash
```

```bash
capsh --print
```

✅ **Expected:** All capabilities are available. The container can do nearly anything the host can.

---

## 🔹 Lab 5: Test Filesystem Capabilities

### Step 1: Try mounting inside a container

```bash
docker run --rm -it --cap-add=SYS_ADMIN debian bash
mkdir /mnt/test
mount -t tmpfs none /mnt/test
```

✅ **Expected:** Works with `CAP_SYS_ADMIN`; fails without it.

---

## 🔹 Lab 6: Inspect Running Containers

### Step 1: Run a container in the background

```bash
docker run -d --cap-drop=ALL --name minimal alpine sleep 300
```

### Step 2: Inspect the container

```bash
docker inspect minimal | grep Cap
```

✅ **Expected:** `CapDrop` should list all dropped caps, `CapAdd` should be empty.

---

## ✅ Wrap-Up

- You explored capabilities in Docker and saw how they restrict or allow actions.
- You ran containers with minimal privileges (`--cap-drop=ALL`).
- You added capabilities only as needed (`--cap-add=...`).
- You compared privilege escalation using `--privileged`.

---