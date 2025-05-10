# 🧪 Lab Series: System Logging on Linux (Ubuntu)

## 🎯 Objective

Learn to manage and understand system logs using native Linux tools like `rsyslog`, `journald`, and `logrotate`.

---

## 🧰 Prerequisites

- Ubuntu 20.04+ system (VM recommended)
- Root or `sudo` access

---

## 🔹 Lab 1: Understanding the Logging Stack

Ubuntu uses:

- `journald` (systemd journal) for structured logging
- `rsyslog` for traditional syslog compatibility
- `logrotate` for rotating log files

---

## 🔹 Lab 2: View Logs with `journalctl`

### View the entire system log:

```bash
journalctl
```

### View logs for a specific service:

```bash
journalctl -u ssh
```

### Follow logs in real-time:

```bash
journalctl -f
```

✅ `journalctl` is the main tool for querying system logs.

---

## 🔹 Lab 3: Configure Persistent Journaling

By default, journald logs may be in memory only.

### Make logs persistent:

```bash
sudo mkdir -p /var/log/journal
sudo systemctl restart systemd-journald
```

✅ Logs will now survive reboots.

---

## 🔹 Lab 4: Explore Traditional Log Files with `rsyslog`

```bash
cat /var/log/syslog
cat /var/log/auth.log
```

✅ Use these for classic log file views.

---

## 🔹 Lab 5: Create a Custom rsyslog Rule

### Step 1: Create a custom log file

```bash
echo "local0.*  /var/log/mycustom.log" | sudo tee /etc/rsyslog.d/30-custom.conf
```

### Step 2: Restart rsyslog

```bash
sudo systemctl restart rsyslog
```

### Step 3: Test the rule

```bash
logger -p local0.info "This is a test log entry"
cat /var/log/mycustom.log
```

✅ Custom logs go to your specified file.

---

## 🔹 Lab 6: Configure Log Rotation with `logrotate`

View the config:

```bash
cat /etc/logrotate.conf
cat /etc/logrotate.d/rsyslog
```

Manually trigger rotation:

```bash
sudo logrotate -f /etc/logrotate.conf
```

✅ Helps prevent logs from filling the disk.

---

## 🔹 Lab 7: Secure Your Logs

### Ensure correct permissions:

```bash
sudo chmod -R go-rwx /var/log/*
```

✅ Only root can read/write logs.

---

## ✅ Wrap-Up

- ✅ Explored `journalctl` and `rsyslog`
- ✅ Created persistent journal and custom log rules
- ✅ Set up log rotation and secured logs

---
