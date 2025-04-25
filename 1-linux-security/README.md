# Linux Security Basics for Containers

## Prerequisites

* Linux VM or container host (Ubuntu recommended)
* Basic knowledge of the Linux commandline
* Basic commands like `ps`, `(h)top`, `ls`, `chmod`, `chown` and `chroot`
* Further prerequisites are listed in the individual labs

## Table of Contents

* [Linux Users, Groups and File Permissions](#linux-file-permissions)
  * [Introduction: Understand Linux file permissions](#introduction-understand-linux-file-permissions)
  * [Lab: Using special permissions using setuid and setgid](#lab-using-special-permissions-using-setuid-and-setgid)
* [Linux Capabilities and Dropping Privileges](#linux-capabilities)
* [Linux Namespaces](#linux-namespaces)
* [Linux Control Groups (cgroups) and Resource Limits](#learning-linux-cgroups)
* [Seccomp and System Call Filtering](#seccomp-and-system-call-filtering)

## Labs

### Linux File permissions

#### Introduction: Understand Linux file permissions

In Linux, everything is a file, like:

* Binary application code
* Data
* Configuration
* Logs
* Devices

Permissions on such files determine which users are allowed to access those files and what actions they
can perform on the files.

Each file and directory has three user-based permission groups:

* __u - owner__ – The _Owner_ permissions apply only to the owner of the file or directory, they will not impact the actions of other users.
* __g - group__ – The _Group_ permissions apply only to the group that has been assigned to the file or directory, they will not affect the actions of other users.
* __all users__ – The _All Users_ permissions apply to all other users on the system, this is the permission group that you want to watch the most.

The Permission Types that are used are:

* __r__ – Read
* __w__ – Write
* __x__ – Execute

The permissions are displayed as: `-rwxrwxrwx 1 owner:group`.
Using `ls -l test.txt` would result in the following:

```bash
-rw-r--r-- 1 myuser mygroup test.txt
```

1. The first character is the special permission flag 
2. The following set of three characters (rwx) is for the _owner_ permissions
3. The second set of three characters (rwx) is for the _group_ permissions
4. The third set of three characters (rwx) is for the _all users_ permissions
5. Following the grouping the number displays the number of hard links to the file
6. The last piece is the _owner_ and _group_ assignment

The file owner and group can be changed using the `chown` command. By performing `chown myuser:mygroup test.txt` the owner of the file _test.txt_ would be _myuser_ and the group would be set to _mygroup_.

The file permissions are edited by using the command `chmod`. You can assign the permissions explicitly or by using a binary reference.
You may add the _read_ and _write_ permission to the group using `chmod g+rw text.txt`. To remove the same permissions for all other users you would type `chmod o-rw text.txt`.

You may also specify the complete file permissions using a binary reference instead:
The numbers are a binary representation of the rwx string.

* r (read) = 4
* w (write) = 2
* x (execute) = 1

So you could also perform `chmod 644 test.txt` instead.

### Lab: Using special permissions using setuid and setgid

When executing a file, usually the process that gets started inherits your user ID.
If the file has the setuid/setgid bit set, the process will have the user/group ID of the file’s owner/group instead.

We will try that using the `sleep` command. Because we will change permissions first, we will copy the binary to our own one experiment with. To check the installation path of the `sleep` file perform a `which sleep`. 
With this path perform the copy command:

```bash
cp /bin/sleep ./mysleep
```

Now let's check the file permissions for the _mysleep_ file:

```bash
ls -l mysleep
```

This should return something like this:

```bash
-rwxr-xr-x 1 afa afa 39256 Nov 27 18:15 mysleep
```

Normally, when you execute a file, the process that gets started inherits your user ID.
If you now execute it as a root user in a terminal with

```bash
sudo ./mysleep 20
```

And then execute this in another terminal:

```bash
ps ajf
```

Then this will run with the root user id:

```bash
PPID     PID    PGID     SID TTY        TPGID STAT   UID   TIME COMMAND
 735453  810647  810647  735453 pts/0     810647 S+       0   0:00  \_ sudo ./mysleep 20
 810647  810648  810647  735453 pts/0     810647 S+       0   0:00      \_ ./mysleep 20
```

Now with setuid bit set:

```bash
chmod +s mysleep
```

Check again with 

```bash
ls -l mysleep
```

```bash
-rwsr-sr-x 1 afa afa 39256 Nov 27 18:15 mysleep
```

If you now execute this in one terminal:

```bash
sudo ./mysleep 20
```

And execute this in another terminal:

```bash
ps ajf
```

Then you will see that even when executing as root, the command is run using the other user id:

```bash
PPID     PID    PGID     SID TTY        TPGID STAT   UID   TIME COMMAND
 735453  810637  810637  735453 pts/0     810637 S+       0   0:00  \_ sudo ./mysleep 20
 810637  810638  810637  735453 pts/0     810637 S+    1000   0:00      \_ ./mysleep 20
```

This bit is typically used to give a program privilege that it needs but is not
usually extended to regular users.
Because _setuid_ provides a dangerous pathway to privilege escalation, some container
image security scanners report on the presence of files with the _setuid_ bit set.

### Linux capabilities

Back in the old days the only way in Linux has been to either execute a process in privileged (_root_) or unprivileged mode (all other users).
 
With linux capabilities you can now break down privileges used by executing processes/threads to just grant the least possible privileges required to successfully run a thread.

Look up the detailed docs for linux capabilities by

```shell
man capabilities
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

If you want to query capabilities for a process, use this command

```shell
getpcaps <pid>
```

You should use this for root processes. Processes for non-root users usually do not have 
any capability set.

You may query capabilities for a file:

```bash
getcap /usr/bin/ping
```

This will return

```bash
/usr/bin/ping = cap_net_raw+ep
```

As you can see the `ping` command requires the `net_raw` capability to access the network socket.

You may also set capabilities for a file:

```shell
sudo setcap 'cap_net_raw+p' ./myping
```

You will see in the container security part how docker uses capabilities to run containers with the fewest privileges required.

## Linux Namespaces

### 🎯 Objective

Learn how Linux namespaces isolate system resources by creating isolated environments using the unshare and nsenter tools.

### 🧰 Prerequisites
- Linux system (Ubuntu/Debian/CentOS)
- Tools: `util-linux` (`unshare`, `nsenter`), `procps`, `iproute2`, `coreutils`
- Root or `sudo` access recommended

### Install required tools:

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

### 🔹 Lab 1: Isolating the hostname using a UTS namespace

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

### Lab 2: Isolating the process id using a PID namespace

You can also use the _PID_ namespace to isolate the process id.

```shell
sudo unshare -fp --mount-proc bash
ps aux
```

Only a few processes show now, starting from PID 1. This simulates what a container would see: a trimmed-down process view.

✅ Simulates container-like PID isolation.

---

### 🔹 Lab 3: Isolating the network using a network namespace

```shell
sudo unshare -n bash
ip a
```

This should show only the `lo` interface (loopback). Network is isolated. You can’t reach the outside world.

✅ Network isolation achieved.

---

### 🔹 Lab 4: Inspect Namespace Links in `/proc`

```shell
echo $$
ls -l /proc/$$/ns/
```

You’ll see symbolic links to namespace descriptors like mnt, uts, pid, etc.

✅ Observe `mnt`, `uts`, `pid`, `net`, and other namespace descriptors.

---

### 🔹 Lab 5: Isolating the mount namespace

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

### 🔹 Lab 6: Combine multiple Namespaces

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

### 🔹 Lab 7: Enter Another Process’s Namespace

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

### ✅ Wrap-Up

* Namespaces are the foundation of container isolation.
* Learned to use `unshare` and `nsenter` for namespace exploration
* Explored UTS, PID, MNT, NET, and combined namespaces
* Simulated container isolation mechanisms using native Linux features

---

## Learning Linux cgroups

### 🎯 Objective
Understand how Linux cgroups isolate and limit process resources. These labs use native Linux tools (no Docker) to demonstrate:
- CPU and memory limits
- Monitoring cgroup behavior
- Cgroups v1 and systemd-based cgroups (v2)

---

### 🧰 Prerequisites
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

### 🔹 Lab 1: CPU Limit with cgroups (v1)

#### 1.1 Create a CPU cgroup
```bash
sudo cgcreate -g cpu:/mycg
```

#### 1.2 Limit CPU usage to 25%
```bash
echo 25000 | sudo tee /sys/fs/cgroup/cpu/mycg/cpu.cfs_quota_us
echo 100000 | sudo tee /sys/fs/cgroup/cpu/mycg/cpu.cfs_period_us
```

#### 1.3 Run a CPU-intensive task
```bash
sudo cgexec -g cpu:mycg stress --cpu 1 --timeout 20
```

#### 1.4 Monitor with `top` or `htop`

✅ Expected: CPU usage is throttled.

---

### 🔹 Lab 2: Memory Limit with cgroups (v1)

#### 2.1 Create a memory cgroup
```bash
sudo cgcreate -g memory:/memcg
```

#### 2.2 Set the memory limit to 100 MB
```bash
echo $((100 * 1024 * 1024)) | sudo tee /sys/fs/cgroup/memory/memcg/memory.limit_in_bytes
```

#### 2.3 Run a memory-hungry task
```bash
sudo cgexec -g memory:memcg stress --vm 1 --vm-bytes 150M --timeout 20
```

✅ Expected: Process is killed once it exceeds memory limit.

---

### 🔹 Lab 3: Monitor cgroup Stats

Check CPU stats:
```bash
cat /sys/fs/cgroup/cpu/mycg/cpu.stat
```

Check memory usage:
```bash
cat /sys/fs/cgroup/memory/memcg/memory.usage_in_bytes
```

---

### 🔹 Lab 4: systemd-based cgroups (v2-style)

#### 4.1 Start a transient systemd unit with CPU limit
```bash
sudo systemd-run --unit=mycpulimit --property=CPUQuota=25% stress --cpu 1 --timeout 20
```

#### 4.2 Check status
```bash
systemctl status mycpulimit.service
```

✅ Expected: The stress task runs with CPU limited by systemd.

---

### 🔹 Bonus: Inspect Process Membership

Get PID of a process:
```bash
pgrep stress
```

Check its cgroup:
```bash
cat /proc/<pid>/cgroup
```

---

### ✅ Wrap-Up

- You applied CPU & memory limits to processes using cgroups.
- Explored both **cgroups v1** (manually) and **systemd-based cgroups** (v2).
- Observed how system resources can be controlled at the kernel level.


## 🧪 Seccomp and System Call Filtering

## 🎯 Objective
Understand how seccomp filters syscalls using profiles and explore how processes can be restricted from making unsafe system calls.

---

## 🧰 Prerequisites

- Linux system with kernel ≥ 4.4
- Tools: `gcc`, `make`, `strace`, `seccomp-tools`

### Install dependencies:

```bash
sudo apt update
sudo apt install gcc make strace seccomp-tools
```

---

## 🔹 Lab 1: Inspect syscalls with `strace`

### 1.1 Trace system calls of a command

```bash
strace ls
```

✅ **Expected:** You'll see many syscalls like `openat`, `read`, `write`, etc.

### 1.2 Trace network command

```bash
strace ping -c 1 8.8.8.8
```

✅ **Expected:** You’ll see calls like `socket`, `sendto`, `recvmsg`, etc.

---

### 🔹 Lab 2: Block syscalls with seccomp in C

#### 2.1 Create a C program that uses `write`

```c
// write_test.c
#include <unistd.h>
#include <stdio.h>

int main() {
    write(STDOUT_FILENO, "Hello, world!\n", 14);
    return 0;
}
```

Compile:

```bash
gcc -o write_test write_test.c
./write_test
```

✅ **Expected:** It prints: Hello, world!

---

### 🔹 Lab 3: Apply a seccomp filter to block `write`

#### 3.1 Modify the C code to add seccomp

```c
// write_seccomp.c
#include <stdio.h>
#include <unistd.h>
#include <seccomp.h>

int main() {
    scmp_filter_ctx ctx;

    ctx = seccomp_init(SCMP_ACT_ALLOW); // allow everything
    seccomp_rule_add(ctx, SCMP_ACT_KILL, SCMP_SYS(write), 0); // block write
    seccomp_load(ctx);

    write(STDOUT_FILENO, "This should fail\n", 17);
    return 0;
}
```

#### 3.2 Compile and run

```bash
gcc -o write_seccomp write_seccomp.c -lseccomp
./write_seccomp
```

❌ **Expected:** The program is killed by seccomp when calling `write`.

---

### 🔹 Lab 4: Use `seccomp-tools` to inspect a binary

Install seccomp-tools (if not installed):

```bash
sudo gem install seccomp-tools
```

Inspect:

```bash
seccomp-tools dump ./write_seccomp
```

✅ **Expected:** Shows a summary of syscall filtering rules.

---

### 🔹 Lab 5: Use `prctl` for strict mode filtering

```c
// write_prctl.c
#include <stdio.h>
#include <unistd.h>
#include <sys/prctl.h>
#include <linux/seccomp.h>
#include <linux/filter.h>
#include <errno.h>

int main() {
    prctl(PR_SET_SECCOMP, SECCOMP_MODE_STRICT);
    write(STDOUT_FILENO, "Hello (strict mode)\n", 21);
    return 0;
}
```

```bash
gcc -o write_prctl write_prctl.c
./write_prctl
```

✅ **Expected:** Works with only `read`, `write`, `_exit`, `sigreturn`.

---

### ✅ Wrap-Up

- You used `strace` to observe syscall activity
- You created seccomp profiles in C
- You used strict and filtered seccomp modes
- You understood how seccomp improves security by syscall whitelisting

---

## 🧪 Understanding SELinux

### 🎯 Objective
Learn how SELinux works, how to manage policies, and how to troubleshoot denials using native Linux systems.

---

### 🧰 Prerequisites

- Linux system with **SELinux enabled** (e.g., **Fedora**, **RHEL**, **CentOS**, **Rocky Linux**)
- Root or `sudo` access
- Basic Linux command-line knowledge

#### 🛠️ Confirm SELinux is enabled

```bash
sestatus
```

✅ **Expected Output:**
```
SELinux status:                 enabled
Current mode:                   enforcing
```

---

### 🔹 Lab 1: View SELinux Contexts

#### 1.1 Check context of files and processes

```bash
ls -Z /etc/passwd
ps -eZ | grep sshd
```

✅ **Expected:** SELinux contexts like `system_u:object_r:etc_t:s0`.

---

### 🔹 Lab 2: Switch to Permissive Mode

#### 2.1 Temporarily set SELinux to permissive

```bash
sudo setenforce 0
getenforce
```

✅ *SELinux logs denials but does not block access.*

#### 2.2 Set it back to enforcing

```bash
sudo setenforce 1
getenforce
```

---

### 🔹 Lab 3: Create a Policy Violation and Examine Logs

#### 3.1 Create a file and change its context

```bash
echo "secret" > /tmp/secret.txt
sudo chcon -t httpd_sys_content_t /tmp/secret.txt
ls -Z /tmp/secret.txt
```

#### 3.2 View denials (from a non-httpd process context)

```bash
sudo ausearch -m AVC,USER_AVC -ts recent
```

✅ *SELinux should block improper access attempts.*

---

### 🔹 Lab 4: Restore Default SELinux Contexts

#### 4.1 Reset file label using `restorecon`

```bash
sudo restorecon -v /tmp/secret.txt
```

✅ **Expected:** File context is restored to default.

---

### 🔹 Lab 5: View and Use Boolean Flags

#### 5.1 List SELinux booleans

```bash
getsebool -a | grep ftp
```

#### 5.2 Enable boolean for FTP access

```bash
sudo setsebool -P ftp_home_dir 1
getsebool ftp_home_dir
```

✅ *Allows FTP daemons to access user home directories.*

---

### 🔹 Lab 6: Analyze and Troubleshoot SELinux Denials

#### 6.1 Analyze recent denials

```bash
sudo ausearch -m AVC -ts recent | audit2why
```

✅ **Expected:** Explanation of SELinux blocks.

#### 6.2 Create and apply a custom policy (optional)

```bash
sudo ausearch -m AVC -ts recent | audit2allow -M mypol
sudo semodule -i mypol.pp
```

✅ *Creates and loads a custom module to allow specific actions.*

---

### ✅ Wrap-Up

- You viewed SELinux labels and processes.
- You worked with permissive and enforcing modes.
- You restored contexts and managed booleans.
- You analyzed and handled policy violations with custom rules.

---

## 🧪 Understanding AppArmor

### 🎯 Objective
Learn how AppArmor profiles work, how to inspect and apply them, and how to debug blocked actions on a real Linux system.

---

### 🧰 Prerequisites

- Linux system with **AppArmor support** (Ubuntu, Debian, openSUSE)
- Root or `sudo` access
- Tools: `apparmor-utils`, `auditd`

#### Install required packages

```bash
sudo apt update
sudo apt install apparmor-utils auditd
```

#### Confirm AppArmor is enabled

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

### 🔹 Lab 1: View Active AppArmor Profiles

```bash
sudo aa-status
```

✅ Shows which profiles are loaded, in enforce or complain mode.

---

### 🔹 Lab 2: Put an Application in Complain Mode

#### 2.1 Switch a known app to complain mode

```bash
sudo aa-complain /bin/ping
```

#### 2.2 Run the app and view logs

```bash
ping -c 1 8.8.8.8
sudo journalctl | grep apparmor
```

✅ *Logs show actions that would be denied in enforce mode.*

---

### 🔹 Lab 3: Enforce a Profile and Trigger a Denial

#### 3.1 Switch to enforce mode

```bash
sudo aa-enforce /bin/ping
```

#### 3.2 Trigger restricted behavior

```bash
ping -c 1 8.8.8.8 > /tmp/pinglog.txt
```

❌ **Expected:** Access denied or failure due to enforced policy.

---

### 🔹 Lab 4: Write and Load a Custom Profile

#### 4.1 Create a test script

```bash
echo -e '#!/bin/bash\ncat /etc/shadow' > ~/readshadow.sh
chmod +x ~/readshadow.sh
```

#### 4.2 Run it once (it should succeed with `sudo`)

```bash
~/readshadow.sh
```

#### 4.3 Generate a new profile

```bash
sudo aa-genprof ~/readshadow.sh
```

✅ Use guided prompts to define rules (deny reading /etc/shadow).

---

### 🔹 Lab 5: View and Edit AppArmor Profiles

Profiles live in:

```bash
ls /etc/apparmor.d/
sudo nano /etc/apparmor.d/home.<youruser>.readshadow.sh
```

✅ Profiles are human-readable and easily editable.

---

### 🔹 Lab 6: Monitor and Audit Denials

```bash
sudo journalctl -xe | grep apparmor
sudo ausearch -m APPARMOR
```

✅ Provides detailed trace of blocked actions.

---

### ✅ Wrap-Up

- You listed and managed AppArmor profiles.
- You enforced and monitored application restrictions.
- You created and tested a custom policy.
- You audited real-world access denials.

---

## Next Up

[Next: Container Security](../2-container-security)
