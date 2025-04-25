# 🧪 Learning Linux cgroups

## 🎯 Objective
Understand how Linux cgroups isolate and limit process resources. These labs use native Linux tools (no Docker) to demonstrate:
- CPU and memory limits
- Monitoring cgroup behavior
- Cgroups v1 and systemd-based cgroups (v2)

---

## 🧰 Prerequisites
- Linux system (Ubuntu/Debian/CentOS)
- Root or `sudo` access
- Tools: `cgroup-tools`, `stress`

```bash
sudo apt update
sudo apt install cgroup-tools stress
```

Check the `cgroups` version:
```bash
mount | grep cgroup
```

---

## 🔹 Lab 1: CPU Limit with cgroups (v1)

### 1.1 Create a CPU cgroup
```bash
sudo cgcreate -g cpu:/mycg
```

### 1.2 Limit CPU usage to 25%
```bash
echo 25000 | sudo tee /sys/fs/cgroup/cpu/mycg/cpu.cfs_quota_us
echo 100000 | sudo tee /sys/fs/cgroup/cpu/mycg/cpu.cfs_period_us
```

### 1.3 Run a CPU-intensive task
```bash
sudo cgexec -g cpu:mycg stress --cpu 1 --timeout 20
```

### 1.4 Monitor with `top` or `htop`

✅ Expected: CPU usage is throttled.

---

## 🔹 Lab 2: Memory Limit with cgroups (v1)

### 2.1 Create a memory cgroup
```bash
sudo cgcreate -g memory:/memcg
```

### 2.2 Set the memory limit to 100 MB
```bash
echo $((100 * 1024 * 1024)) | sudo tee /sys/fs/cgroup/memory/memcg/memory.limit_in_bytes
```

### 2.3 Run a memory-hungry task
```bash
sudo cgexec -g memory:memcg stress --vm 1 --vm-bytes 150M --timeout 20
```

✅ Expected: Process is killed once it exceeds memory limit.

---

## 🔹 Lab 3: Monitor cgroup Stats

Check CPU stats:
```bash
cat /sys/fs/cgroup/cpu/mycg/cpu.stat
```

Check memory usage:
```bash
cat /sys/fs/cgroup/memory/memcg/memory.usage_in_bytes
```

---

## 🔹 Lab 4: systemd-based cgroups (v2-style)

### 4.1 Start a transient systemd unit with CPU limit
```bash
sudo systemd-run --unit=mycpulimit --property=CPUQuota=25% stress --cpu 1 --timeout 20
```

### 4.2 Check status
```bash
systemctl status mycpulimit.service
```

✅ Expected: The stress task runs with CPU limited by systemd.

---

## 🔹 Bonus: Inspect Process Membership

Get PID of a process:
```bash
pgrep stress
```

Check its cgroup:
```bash
cat /proc/<pid>/cgroup
```

---

## ✅ Wrap-Up

- You applied CPU & memory limits to processes using cgroups.
- Explored both **cgroups v1** (manually) and **systemd-based cgroups** (v2).
- Observed how system resources can be controlled at the kernel level.

---