# 🧪 Understanding SELinux

## 🎯 Objective

Learn how SELinux works, how to manage policies, and how to troubleshoot denials using native Linux systems.

---

## 🧰 Prerequisites

- Linux system with **SELinux enabled** (e.g., **Fedora**, **RHEL**, **CentOS**, **Rocky Linux**)
- Root or `sudo` access
- Basic Linux command-line knowledge

### 🛠️ Confirm SELinux is enabled

```bash
sestatus
```

✅ **Expected Output:**

```bash
SELinux status:                 enabled
Current mode:                   enforcing
```

---

## 🔹 Lab 1: View SELinux Contexts

### 1.1 Check the context of files and processes

```bash
ls -Z /etc/passwd
ps -eZ | grep sshd
```

✅ **Expected:** SELinux contexts like `system_u:object_r:etc_t:s0`.

---

## 🔹 Lab 2: Switch to Permissive Mode

### 2.1 Temporarily set SELinux to permissive

```bash
sudo setenforce 0
getenforce
```

✅ *SELinux logs denials but does not block access.*

### 2.2 Set it back to enforcing

```bash
sudo setenforce 1
getenforce
```

---

## 🔹 Lab 3: Create a Policy Violation and Examine Logs

### 3.1 Create a file and change its context

```bash
echo "secret" > /tmp/secret.txt
sudo chcon -t httpd_sys_content_t /tmp/secret.txt
ls -Z /tmp/secret.txt
```

### 3.2 View denials (from a non-httpd process context)

```bash
sudo ausearch -m AVC,USER_AVC -ts recent
```

✅ *SELinux should block improper access attempts.*

---

## 🔹 Lab 4: Restore Default SELinux Contexts

### 4.1 Reset file label using `restorecon`

```bash
sudo restorecon -v /tmp/secret.txt
```

✅ **Expected:** File context is restored to default.

---

## 🔹 Lab 5: View and Use Boolean Flags

### 5.1 List SELinux booleans

```bash
getsebool -a | grep ftp
```

### 5.2 Enable boolean for FTP access

```bash
sudo setsebool -P ftp_home_dir 1
getsebool ftp_home_dir
```

✅ *Allows FTP daemons to access user home directories.*

---

## 🔹 Lab 6: Analyze and Troubleshoot SELinux Denials

### 6.1 Analyze recent denials

```bash
sudo ausearch -m AVC -ts recent | audit2why
```

✅ **Expected:** Explanation of SELinux blocks.

### 6.2 Create and apply a custom policy (optional)

```bash
sudo ausearch -m AVC -ts recent | audit2allow -M mypol
sudo semodule -i mypol.pp
```

✅ *Creates and loads a custom module to allow specific actions.*

---

## ✅ Wrap-Up

- You viewed SELinux labels and processes.
- You worked with permissive and enforcing modes.
- You restored contexts and managed booleans.
- You analyzed and handled policy violations with custom rules.

---
