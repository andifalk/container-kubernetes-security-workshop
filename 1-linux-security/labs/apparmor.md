# 🧪 Understanding AppArmor

## 🎯 Objective
Learn how AppArmor profiles work, how to inspect and apply them, and how to debug blocked actions on a real Linux system.

---

## 🧰 Prerequisites

- Linux system with **AppArmor support** (Ubuntu, Debian, openSUSE)
- Root or `sudo` access
- Tools: `apparmor-utils`, `auditd`

### Install required packages

```bash
sudo apt update
sudo apt install apparmor-utils auditd
```

### Confirm AppArmor is enabled

```bash
sudo aa-status
```

✅ **Expected Output:**
```
apparmor module is loaded.
<...>
profiles are in enforce mode.
```

---

## 🔹 Lab 1: View Active AppArmor Profiles

```bash
sudo aa-status
```

✅ Shows which profiles are loaded, in enforce or complain mode.

---

## 🔹 Lab 2: Put an Application in Complain Mode

### 2.1 Switch a known app to complain mode

```bash
sudo aa-complain /bin/ping
```

### 2.2 Run the app and view logs

```bash
ping -c 1 8.8.8.8
sudo journalctl | grep apparmor
```

✅ *Logs show actions that would be denied in enforce mode.*

---

## 🔹 Lab 3: Enforce a Profile and Trigger a Denial

### 3.1 Switch to enforce mode

```bash
sudo aa-enforce /bin/ping
```

### 3.2 Trigger restricted behavior

```bash
ping -c 1 8.8.8.8 > /tmp/pinglog.txt
```

❌ **Expected:** Access denied or failure due to enforced policy.

---

## 🔹 Lab 4: Write and Load a Custom Profile

### 4.1 Create a test script

```bash
echo -e '#!/bin/bash\ncat /etc/shadow' > ~/readshadow.sh
chmod +x ~/readshadow.sh
```

### 4.2 Run it once (it should succeed with `sudo`)

```bash
~/readshadow.sh
```

### 4.3 Generate a new profile

```bash
sudo aa-genprof ~/readshadow.sh
```

✅ Use guided prompts to define rules (deny reading /etc/shadow).

---

## 🔹 Lab 5: View and Edit AppArmor Profiles

Profiles live in:

```bash
ls /etc/apparmor.d/
sudo nano /etc/apparmor.d/home.<youruser>.readshadow.sh
```

✅ Profiles are human-readable and easily editable.

---

## 🔹 Lab 6: Monitor and Audit Denials

```bash
sudo journalctl -xe | grep apparmor
sudo ausearch -m APPARMOR
```

✅ Provides detailed trace of blocked actions.

---

## ✅ Wrap-Up

- You listed and managed AppArmor profiles.
- You enforced and monitored application restrictions.
- You created and tested a custom policy.
- You audited real-world access denials.

---
