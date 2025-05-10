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

Check where `ping`is installed first with `which ping`.

```bash
getcap /bin/ping
```

✅ **Expected Output:**

```bash
/bin/ping cap_net_raw=ep
```

✅ cap_net_raw=ep Breakdown

**cap_net_raw**: This is a Linux capability that allows a process to:

- Open raw sockets
- Send and receive ICMP (ping), which is normally a privileged operation
- **e** = Effective: The capability is active when the binary is run
- **p** = Permitted: The capability is allowed for the binary

So, when you run `/bin/ping`, it has permission to use raw sockets without needing to be _setuid root_.

---

## 🔹 Lab 2: Remove Capability from a Binary

```bash
sudo setcap -r /bin/ping
ping -c 1 8.8.8.8
```

❌ This should fail with Operation not permitted message

```bash
ping: socktype: SOCK_RAW
ping: socket: Operation not permitted
ping: => missing cap_net_raw+p capability or setuid?
```

---

## 🔹 Lab 3: Restore the Capability

```bash
sudo setcap cap_net_raw+ep /bin/ping
getcap /bin/ping
ping -c 1 8.8.8.8 
```

✅ This should work again

```bash
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=115 time=3.22 ms

--- 8.8.8.8 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 3.223/3.223/3.223/0.000 m
```

---

## 🔹 Lab 4: Drop Capabilities from a Shell (Using `capsh`)

First try if ping works with standard capabilities set:

```bash
sudo capsh -- -c "ping -c 1 8.8.8.8"
```

Now try again but with dropping the _cap_net_raw_ capability:

```bash
sudo capsh --drop=cap_net_raw -- -c "ping -c 1 8.8.8.8"
```

❌ **Expected:** Ping fails due to missing capability.

---

## 🔹 Lab 5: Run a Root Process with Minimal Capabilities

### Step 1: Write a simple C program

Open up an editor with `nano showuid.c` and copy the following content into the editor: 

```c
// showuid.c
#include <stdio.h>
#include <unistd.h>

int main() {
    printf("UID: %d\n", getuid());
    sleep(2000);
    return 0;
}
```

Write the file using `Ctrl-W` and exit the editor using `Ctrl-X`.

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

```bash
getpcaps <pid>
```

Execute the binary `./showuid` from previous lab.

Then open a new shell and check for the process id:

```bash
ps ax | grep showuid
```

```bash
999985 pts/1    S+     0:00 ./showuid
1000912 pts/0    S+     0:00 grep --color=auto showuid
```

With the corresponding process id check for effective capabilities of the process:

```bash
getpcaps 999985
```

✅ Expected output

```bash
999985: cap_net_raw=ep
```

You should mainly use this for root processes. Processes for non-root users usually do not have
any capability set.

---

## ✅ Wrap-Up

- ✅ Used `getcap`, `setcap`, and `capsh` to inspect and modify capabilities
- ✅ Demonstrated capability-based privilege separation
- ✅ Reinforced the principle of least privilege

---
