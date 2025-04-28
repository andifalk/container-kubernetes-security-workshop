# 🧪 Using RBAC in Kubernetes

## 🎯 Objective
Learn how to create and manage Kubernetes RBAC (Role-Based Access Control) rules using `Roles`, `ClusterRoles`, `RoleBindings`, and `ClusterRoleBindings`.

---

## 🧰 Prerequisites

- Kubernetes cluster (Minikube, kind, GKE, etc.)
- `kubectl` configured
- Admin access to the cluster

---

## 🔹 Lab 1: Create a Service Account

```bash
kubectl create serviceaccount dev-user
```

✅ Service account `dev-user` created in the default namespace.

---

## 🔹 Lab 2: Create a Role to View Pods

```yaml
# view-pods-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

```bash
kubectl apply -f view-pods-role.yaml
```

✅ Role `pod-reader` created.

---

## 🔹 Lab 3: Bind the Role to the Service Account

```yaml
# view-pods-rolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
subjects:
- kind: ServiceAccount
  name: dev-user
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

```bash
kubectl apply -f view-pods-rolebinding.yaml
```

✅ `dev-user` can now read pods in the default namespace.

---

## 🔹 Lab 4: Test Access with Impersonation

```bash
kubectl auth can-i list pods --as=system:serviceaccount:default:dev-user
```

✅ Should return `yes`.

```bash
kubectl auth can-i create deployments --as=system:serviceaccount:default:dev-user
```

❌ Should return `no`.

---

## 🔹 Lab 5: Create a ClusterRole to Access Nodes (Cluster-Wide)

```yaml
# node-reader-clusterrole.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list"]
```

```bash
kubectl apply -f node-reader-clusterrole.yaml
```

---

## 🔹 Lab 6: Bind ClusterRole to a ServiceAccount

```yaml
# node-reader-clusterrolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: node-reader-binding
subjects:
- kind: ServiceAccount
  name: dev-user
  namespace: default
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```

```bash
kubectl apply -f node-reader-clusterrolebinding.yaml
```

✅ `dev-user` can now read cluster-wide node resources.

---

## 🔹 Lab 7: Clean Up

```bash
kubectl delete serviceaccount dev-user
kubectl delete role pod-reader
kubectl delete rolebinding read-pods-binding
kubectl delete clusterrole node-reader
kubectl delete clusterrolebinding node-reader-binding
```

---

## ✅ Wrap-Up

- ✅ Created namespace-specific and cluster-wide roles
- ✅ Granted and verified permissions with service accounts
- ✅ Practiced secure, least-privilege access control

---