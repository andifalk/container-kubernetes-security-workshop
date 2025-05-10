# 🧪 Seccomp and System Call Filtering

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

## 🔹 Lab 2: Block syscalls with seccomp in C

### 2.1 Create a C program that uses `write`

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

## 🔹 Lab 3: Apply a seccomp filter to block `write`

### 3.1 Modify the C code to add seccomp

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

### 3.2 Compile and run

```bash
gcc -o write_seccomp write_seccomp.c -lseccomp
./write_seccomp
```

❌ **Expected:** The program is killed by seccomp when calling `write`.

---

## 🔹 Lab 4: Use `seccomp-tools` to inspect a binary

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

## 🔹 Lab 5: Use `prctl` for strict mode filtering

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

## ✅ Wrap-Up

- You used `strace` to observe syscall activity
- You created seccomp profiles in C
- You used strict and filtered seccomp modes
- You understood how seccomp improves security by syscall whitelisting

---
