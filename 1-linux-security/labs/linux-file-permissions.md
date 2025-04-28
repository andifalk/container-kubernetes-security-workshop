# 🧪 Linux File permissions

## 🎯 Objective
Learn to manage Linux file permissions, understand setuid/setgid, and recognize associated security risks.

---

## 🧰 Prerequisites

- Linux system (Ubuntu/Debian/CentOS)
- `sudo` access
- Basic shell familiarity

---

In Linux, everything is a file, like:

* Binary application code
* Data
* Configuration
* Logs
* Devices

Permissions on such files determine which users are allowed to access those files and what actions they can perform on the files.

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

---

## 🔹 Lab 1: Basic File Permissions

### Step 1: Create a file and set permissions

```bash
touch myfile.txt
chmod o+rw myfile.txt
ls -l myfile.txt
```

✅ **Expected:** Only the owner can read/write.

---

## 🔹 Lab 2: Understanding Numeric Permissions

You may also specify the complete file permissions using a binary reference instead:
The numbers are a binary representation of the rwx string.

* r (read) = 4
* w (write) = 2
* x (execute) = 1

So you could also perform `chmod 644 test.txt` instead.

Set different permissions:

```bash
chmod 755 myfile.txt
ls -l myfile.txt
```

✅ **Expected:** Owner can read/write/execute; the group and others can read/execute.

---

## 🔹 Lab 3: Using `umask` to Set Default Permissions

Check and change `umask`:

```bash
umask
umask 027
touch newfile.txt
ls -l newfile.txt
```

✅ New files will have permissions limited by `umask`.

---

## 🔹 Lab 4: Using special permissions using setuid and setgid

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

---

## 🔹 Lab 5: Setgid and Directory Behavior

Create a directory and setgid:

```bash
mkdir setgid-dir
chmod 2775 setgid-dir
ls -ld setgid-dir
```

✅ `s` appears on group permissions.

Files created inside inherit the directory's group:

```bash
cd setgid-dir
touch file1.txt
ls -l
```

✅ Group ownership is inherited automatically.

---

### 🚨 Attack Potential:

- If writable by multiple users, can lead to group privilege escalation or unauthorized access.

---

## 🔹 Lab 6: Finding All Setuid and Setgid Binaries

Audit the system:

```bash
find / -perm -4000 -type f 2>/dev/null  # Setuid
find / -perm -2000 -type f 2>/dev/null  # Setgid
```

✅ Review carefully — attackers often target vulnerable setuid/setgid binaries.

---

## ✅ Wrap-Up

- ✅ Learned about Linux file permissions
- ✅ Created and used setuid/setgid binaries
- ✅ Understood real attack potential
- ✅ Practiced mitigation and auditing techniques

---