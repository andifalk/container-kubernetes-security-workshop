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

| Capability           | Description                                                                                          |
|----------------------|------------------------------------------------------------------------------------------------------|
| CAP_CHOWN            | Make arbitrary changes to file UIDs and GIDs                                                         |
| CAP_NET_ADMIN        | Perform network operations like modify routing tables                                                |
| CAP_NET_BIND_SERVICE | Bind a socket to Internet domain privileged ports (port numbers less than 1024)                      |
| CAP_NET_RAW          | use RAW and PACKET sockets                                                                           |
| CAP_SETUID           | Make arbitrary manipulations of process UIDs                                                         |
| CAP_SYS_ADMIN        | Perform system admin operations like _mount_, _swapon_, _sethostname_ or perform privileged _syslog_ |
| CAP_SYS_BOOT         | Use reboot                                                                                           |
| CAP_SYS_CHROOT       | Use chroot                                                                                           |
| CAP_SYS_TIME         | Set system clock                                                                                     |
| CAP_SYSLOG           | Perform privileged syslog operations                                                                 |

🧠 Useful capabilities:
- `CAP_NET_RAW`
- `CAP_NET_BIND_SERVICE`
- `CAP_SYS_ADMIN` (powerful, use cautiously)

If you want to query capabilities for a process, use this command

```shell
getpcaps <pid>
```

You should use this for root processes. Processes for non-root users usually do not have
any capability set.

---

## ✅ Wrap-Up

- ✅ Used `getcap`, `setcap`, and `capsh` to inspect and modify capabilities
- ✅ Demonstrated capability-based privilege separation
- ✅ Reinforced the principle of least privilege

---