# 🧪 Learning Linux cgroups

🚧 Under Construction!!

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

This should either print cgroup1 or cgroup2:

```bash
cgroup2 on /sys/fs/cgroup type cgroup2 (rw,nosuid,nodev,noexec,relatime,nsdelegate,memory_recursiveprot)
```

---

## 🔹 Lab 1: CPU Limit with cgroups

### Step 1: Create a CPU cgroup

```bash
sudo cgcreate -g cpu:/mycg
```

You can check the current settings of this new cgroup with:

```bash
cgget -g cpu:/mycg
```

This should return this information:

```bash
/mycg:
cpu.weight: 100
cpu.stat: usage_usec 0
 user_usec 0
 system_usec 0
 core_sched.force_idle_usec 0
 nr_periods 0
 nr_throttled 0
 throttled_usec 0
 nr_bursts 0
 burst_usec 0
cpu.weight.nice: 0
cpu.pressure: some avg10=0.00 avg60=0.00 avg300=0.00 total=0
 full avg10=0.00 avg60=0.00 avg300=0.00 total=0
cpu.idle: 0
cpu.stat.local: throttled_usec 0
cpu.max.burst: 0
cpu.max: max 100000
cpu.uclamp.min: 0.00
cpu.uclamp.max: max
```

### Step 2: Limit CPU usage to 25%

```bash
echo 25000 | sudo tee /sys/fs/cgroup/cpu/mycg/cpu.cfs_quota_us
echo 100000 | sudo tee /sys/fs/cgroup/cpu/mycg/cpu.cfs_period_us
```

### Step 3: Run a CPU-intensive task

```bash
sudo cgexec -g cpu:mycg stress --cpu 1 --timeout 20
```

### Step 4: Monitor with `top` or `htop`

✅ Expected: CPU usage is throttled.

---

## 🔹 Lab 2: Memory Limit with cgroups (v1)

### Step 1: Create a memory cgroup

```bash
sudo cgcreate -g memory:/memcg
```

### Step 2: Set the memory limit to 100 MB

```bash
echo $((100 * 1024 * 1024)) | sudo tee /sys/fs/cgroup/memory/memcg/memory.limit_in_bytes
```

### Step 3: Run a memory-hungry task

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

### Step 1: Start a transient systemd unit with CPU limit

```bash
sudo systemd-run --unit=mycpulimit --property=CPUQuota=25% stress --cpu 1 --timeout 20
```

### Step 2: Check status

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
