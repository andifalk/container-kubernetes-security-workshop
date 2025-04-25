# 🧪 Understanding Linux Capabilities (No Docker)

## 🎯 Objective
Learn how to manage Linux capabilities using native tools, to understand how they provide fine-grained control over root privileges.

---

## 🧰 Prerequisites

- Linux system (Ubuntu/Debian recommended)
- Tools: `libcap2-bin`, `capsh`, `getcap`, `setcap`, `ping`, `bash`
- Root or sudo access

### Install required tools:

```bash
sudo apt update
sudo apt install libcap2-bin iputils-ping -y
```

---

## 🔹 Lab 1: View Capabilities on a Binary

```bash
getcap /bin/ping
```

✅ **Expected Output:**
```
/bin/ping = cap_net_raw+ep
```

---

## 🔹 Lab 2: Remove Capability from a Binary

```bash
sudo setcap -r /bin/ping
ping -c 1 8.8.8.8  # ❌ Should fail with permission denied
```

---

## 🔹 Lab 3: Restore the Capability

```bash
sudo setcap cap_net_raw+ep /bin/ping
getcap /bin/ping
ping -c 1 8.8.8.8  # ✅ Should work again
```

---

## 🔹 Lab 4: Drop Capabilities from a Shell (Using `capsh`)

```bash
sudo capsh --drop=cap_net_raw -- -c "ping -c 1 8.8.8.8"
```

❌ **Expected:** Ping fails due to missing capability.

---

## 🔹 Lab 5: Run a Root Process with Minimal Capabilities

### Step 1: Write a simple C program

```c
// showuid.c
#include <stdio.h>
#include <unistd.h>

int main() {
    printf("UID: %d\n", getuid());
    return 0;
}
```

### Step 2: Compile and assign capability

```bash
gcc -o showuid showuid.c
sudo setcap cap_net_raw+ep ./showuid
```

✅ `showuid` has only the `cap_net_raw` capability.

---

## 🔹 Lab 6: List of All Available Capabilities

```bash
man 7 capabilities
```

🧠 Useful capabilities:
- `CAP_NET_RAW`
- `CAP_NET_BIND_SERVICE`
- `CAP_SYS_ADMIN` (powerful, use cautiously)

---

## ✅ Wrap-Up

- ✅ Used `getcap`, `setcap`, and `capsh` to inspect and modify capabilities
- ✅ Demonstrated capability-based privilege separation
- ✅ Reinforced the principle of least privilege

---