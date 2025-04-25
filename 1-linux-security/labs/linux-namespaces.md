# 🧪 Linux Namespaces

## 🎯 Objective

Learn how Linux namespaces isolate system resources by creating isolated environments using the unshare and nsenter tools.

## 🧰 Prerequisites
- Linux system (Ubuntu/Debian/CentOS)
- Tools: `util-linux` (`unshare`, `nsenter`), `procps`, `iproute2`, `coreutils`
- Root or `sudo` access recommended

## Install required tools:

```bash
sudo apt update
sudo apt install util-linux procps iproute2 -y
```

---

Linux namespaces control what a process can see. CGroups (see next section [Linux CGroups](#linux-cgroups)) control the resources that a process can use;

Linux currently provides the following namespaces:
* Unix Timesharing System (UTS): This namespace is responsible for the hostname and domain names.
* Process IDs
* Mount points
* Network
* User and group IDs
* Inter-process communications (IPC)
* Control groups (cgroups)

You can see all namespaces on your machine using the `lsns` command.
Try also to run this command using root, then you are able to see more details.

By using the tool _unshare_ you may run a process with some namespaces unshared from the parent
(i.e., simulating a linux container).

---

## 🔹 Lab 1: Isolating the hostname using a UTS namespace

So let's try to use the UTS namespace to isolate the hostname:

```shell
sudo unshare --uts bash
hostname
hostname isolatedhost
hostname
``` 

In the current shell we have our own hostname isolated by the _UTS_ namespace.
Now open a new terminal and check the hostname. You will notice that the host still has its original name.

```shell
hostname
``` 

✅ Hostname change is isolated.

---

## Lab 2: Isolating the process id using a PID namespace

You can also use the _PID_ namespace to isolate the process id.

```shell
sudo unshare -fp --mount-proc bash
ps aux
```

Only a few processes show now, starting from PID 1. This simulates what a container would see: a trimmed-down process view.

✅ Simulates container-like PID isolation.

---

## 🔹 Lab 3: Isolating the network using a network namespace

```shell
sudo unshare -n bash
ip a
```

This should show only the `lo` interface (loopback). Network is isolated. You can’t reach the outside world.

✅ Network isolation achieved.

---

## 🔹 Lab 4: Inspect Namespace Links in `/proc`

```shell
echo $$
ls -l /proc/$$/ns/
```

You’ll see symbolic links to namespace descriptors like mnt, uts, pid, etc.

✅ Observe `mnt`, `uts`, `pid`, `net`, and other namespace descriptors.

---

## 🔹 Lab 5: Isolating the mount namespace

```shell
sudo unshare --mount bash
mkdir /tmp/mylab
mount -t tmpfs tmpfs /tmp/mylab
mount | grep mylab  # Should show your new mount
```

Now, in another terminal, run:

```shell
mount | grep mylab
```

Our new mount doesn’t appear outside the namespace.

✅ Mount is isolated from the host view.

---

## 🔹 Lab 6: Combine multiple Namespaces

```shell
sudo unshare -u -n -m -p -i -f --mount-proc bash
```

Now check in the same terminal:

```shell
hostname
ps aux
ip a
mount
```

All outputs reflect isolation from the host.
You will see that the hostname is isolated, the process id starts at 1, the network is isolated and the mount namespace is also isolated.

✅ Fully isolated namespace context.

---

## 🔹 Lab 7: Enter Another Process’s Namespace

Another option is the _nsenter_ tool that basically is intended
to run a program with namespaces of other processes.

```shell
sudo unshare -u -p -f --mount-proc bash
```

In another terminal run:

```shell
pid=$(pgrep -f unshare)
sudo nsenter -t $pid -u hostname
```

Now you’ve entered another process’s UTS namespace.

✅ Join and inspect the other process's namespace.

---

## ✅ Wrap-Up

* Namespaces are the foundation of container isolation.
* Learned to use `unshare` and `nsenter` for namespace exploration
* Explored UTS, PID, MNT, NET, and combined namespaces
* Simulated container isolation mechanisms using native Linux features

---