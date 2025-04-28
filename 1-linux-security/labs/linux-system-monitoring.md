# 🧪 Lab Series: System Monitoring on Linux (Ubuntu)

## 🎯 Objective
Learn how to monitor a Linux system’s performance and resource usage using built-in and common open-source tools.

---

## 🧰 Prerequisites

- Ubuntu 20.04+ system (VM or physical)
- `sudo` access

### Install essential tools:

```bash
sudo apt update
sudo apt install htop iotop sysstat net-tools dstat -y
```

---

## 🔹 Lab 1: Monitor CPU and Memory Usage with `top` and `htop`

```bash
top
```

✅ Shows real-time CPU, memory, and process usage.

```bash
htop
```

✅ A more interactive and colorful view.

---

## 🔹 Lab 2: Monitor Disk I/O with `iotop`

```bash
sudo iotop
```

✅ Shows which processes are reading from or writing to the disk.

---

## 🔹 Lab 3: Monitor Disk Usage with `df` and `du`

```bash
df -h
```

✅ View disk usage by filesystem.

```bash
du -sh /var/*
```

✅ See which directories are using the most space.

---

## 🔹 Lab 4: Monitor Network Usage with `netstat` and `ss`

```bash
netstat -tulnp
```

✅ Shows listening ports and the associated processes.

```bash
ss -tuln
```

✅ Modern, faster alternative to `netstat`.

---

## 🔹 Lab 5: Monitor System Load and Performance Over Time

```bash
uptime
```

✅ Displays system load averages.

```bash
sar -u 1 3
```

✅ CPU usage history (requires `sysstat`).

---

## 🔹 Lab 6: Monitor System Resources with `dstat`

```bash
dstat -cdngyt
```

✅ Combines CPU, disk, net, paging, and I/O stats in one view.

---

## 🔹 Lab 7: Identify High Resource Processes

```bash
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head
```

✅ Find top CPU and memory-consuming processes.

---

## ✅ Wrap-Up

- ✅ Used `top`, `htop`, `iotop`, `dstat` for real-time monitoring
- ✅ Monitored CPU, disk, memory, network, and load
- ✅ Learned to investigate and troubleshoot resource hogs

---