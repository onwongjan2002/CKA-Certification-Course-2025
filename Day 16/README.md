Day 16: Mastering Kubernetes Taints & Tolerations | Essential Scheduling Control | CKA Course 2025

## Video reference for Day 16 is the following:

[![Watch the video](https://img.youtube.com/vi/moZNbHD5Lxg/maxresdefault.jpg)](https://www.youtube.com/watch?v=moZNbHD5Lxg&ab_channel=CloudWithVarJosh)


## Table of Contents  
- [Introduction: Why Taints and Tolerations?](#introduction-why-taints-and-tolerations)  
- [Understanding Taints](#understanding-taints)  
  - [How to Apply a Taint](#how-to-apply-a-taint)  
  - [Effects of a Taint](#effects-of-a-taint)  
- [Understanding Tolerations](#understanding-tolerations)  
  - [How to Apply a Toleration](#how-to-apply-a-toleration)  
  - [Toleration Operators](#toleration-operators)  
- [How Taints and Tolerations Work Together](#how-taints-and-tolerations-work-together)  
- [Demonstration](#demonstration)  
  - [Applying Taints](#applying-taints)  
  - [Verifying Taints](#verifying-taints)  
  - [Checking Pod Behavior Without Tolerations](#checking-pod-behavior-without-tolerations)  
  - [Scheduling Pods with Tolerations](#scheduling-pods-with-tolerations)  
  - [Using the Exists Operator](#using-the-exists-operator)  
  - [Applying Multiple Taints and Tolerations](#applying-multiple-taints-and-tolerations)  
- [Key Takeaways](#key-takeaways)  

---

## Introduction: Why Taints and Tolerations?  

In Kubernetes, **Taints and Tolerations** help **control which pods can be scheduled on which nodes**. They allow cluster administrators to **prevent certain workloads from running on specific nodes** while allowing exceptions when necessary.  

### **When Do We Need Taints & Tolerations?**  
Taints and tolerations are useful in several scenarios:  

| **Use Case** | **Explanation** |
|-------------|----------------|
| **Dedicated Nodes** | Taint nodes with a GPU to restrict them to **GPU-intensive workloads** only. |
| **Isolating Critical Workloads** | Ensure **critical workloads** run separately from **non-critical** ones. |
| **Preventing Scheduling on Control Plane Nodes** | Control plane nodes should only run **Kubernetes system workloads**. |
| **Maintenance Mode** | Taint a node under **maintenance** to stop new pods from being scheduled. |
| **Node Resource Constraints** | Prevent pods from being scheduled on nodes with **low memory or CPU**. |

### **Important:**  
Taints and tolerations **only apply during scheduling**. If a pod is already running on a node, adding a taint **will not remove the existing pod** (unless you use the **NoExecute** effect).  

---

## Understanding Taints  

A **Taint** is applied to a **node** to indicate that it **should not accept certain pods** unless they explicitly tolerate it.  

### **How to Apply a Taint?**  
Taints are applied to nodes using the following command:  
```sh
kubectl taint nodes <node-name> <key>=<value>:<effect>
```
**Example:**  
```sh
kubectl taint nodes my-second-cluster-worker storage=ssd:NoSchedule
kubectl taint nodes my-second-cluster-worker2 storage=hdd:NoSchedule
```
- These commands taint `my-second-cluster-worker` to **only allow pods that tolerate `storage=ssd:NoSchedule`** and `my-second-cluster-worker2` to **only allow pods that tolerate `storage=hdd:NoSchedule`**.  

---

### **Effects of a Taint**  

| **Effect** | **Behavior** |
|------------|-------------|
| **NoSchedule** | New pods will **not be scheduled** unless they have a matching toleration. |
| **PreferNoSchedule** | Kubernetes **tries** to avoid scheduling pods but **doesn’t enforce it strictly**. |
| **NoExecute** | Existing pods **without a matching toleration will be evicted** from the node. |

---

## Understanding Tolerations  

A **Toleration** is applied to a **pod**, allowing it to **bypass a node’s taint** and be scheduled on it.  

### **How to Apply a Toleration?**  
Tolerations are defined in a pod’s YAML under `spec.tolerations`:  

```yaml
tolerations:
  - key: "storage"
    operator: "Equal"
    value: "ssd"
    effect: "NoSchedule"
```
- This **tolerates** nodes that have the taint `storage=ssd:NoSchedule`, allowing the pod to be scheduled on them.  
- The **Effect in the toleration must match the Effect in the taint** for it to take effect.  

---

### **Toleration Operators**  

| **Operator** | **Behavior** |
|-------------|-------------|
| **Equal** (default) | The key and value **must exactly match** the taint. |
| **Exists** | Only the key needs to match, and the value is ignored. |

**Example using Exists:**  
```yaml
tolerations:
  - key: "storage"
    operator: "Exists"
    effect: "NoSchedule"
```
- This toleration allows the pod to be scheduled on **any node that has a `storage` taint**, regardless of its value.  

---

## How Taints and Tolerations Work Together  

| **Node Taint** | **Pod Toleration** | **Effect** |
|---------------|-------------------|-----------|
| `storage=ssd:NoSchedule` | **Toleration exists** | ✅ Pod **can** be scheduled. |
| `storage=ssd:NoSchedule` | ❌ No toleration | ❌ Pod **cannot** be scheduled. |
| `storage=ssd:NoExecute` | **Toleration exists** | ✅ Pod **remains** on the node. |
| `storage=ssd:NoExecute` | ❌ No toleration | ❌ Pod **is evicted** from the node. |

---

## Demonstration  

### **Cluster Setup for Demonstration**  

Before we begin applying taints and tolerations, let's first review our Kubernetes cluster setup. We have a total of **three nodes** in our KIND cluster:  

1. **my-second-cluster-control-plane** → This is the **control-plane node** responsible for managing the cluster.  
2. **my-second-cluster-worker** → This is a **worker node** where application workloads can be scheduled.  
3. **my-second-cluster-worker2** → This is another **worker node** available for scheduling workloads.  


### **Applying Taints**  
```sh
kubectl taint nodes my-second-cluster-worker storage=ssd:NoSchedule
kubectl taint nodes my-second-cluster-worker2 storage=hdd:NoSchedule
```

### **Verifying Taints**  
```sh
kubectl describe node my-second-cluster-worker | grep -i taint
kubectl describe node my-second-cluster-worker2 | grep -i taint
```

### **Checking Pod Behavior Without Tolerations**  
```sh
kubectl run mypod --image=nginx
kubectl get pods
kubectl describe pod mypod
```
- The pod **remains in Pending state** because it does **not tolerate any taint**.

### **Why the Pod Is Not Scheduled on the Control Plane Node?**  

After applying taints to our worker nodes, we will attempt to create a pod **without any tolerations** using:  

```sh
kubectl run mypod --image=nginx
```

Since **both worker nodes are tainted**, the pod will remain in a **Pending** state. However, it **will not be scheduled on the control plane node either**.  

#### **Checking Why the Pod Is Not Scheduled**  

To investigate, we will describe the pod using:  

```sh
kubectl describe pod mypod
```

This will show that **no suitable nodes were found for scheduling** due to the applied taints. Additionally, the **control plane node is already tainted by default**, preventing general workloads from running on it.  

#### **Verifying the Taint on the Control Plane Node**  

We can check the taints applied to the control plane node using:  

```sh
kubectl describe node my-second-cluster-control-plane | grep Taints
```

Expected output:  

```plaintext
Taints: node-role.kubernetes.io/control-plane:NoSchedule
```

This confirms that the **control plane node has a taint that prevents regular workloads from being scheduled on it** unless the pod has a corresponding toleration.  

### **Why Were We Able to Manually Schedule a Pod on the Control Plane in the Previous Lecture?**  

In the **previous lecture**, we manually scheduled a pod on the control plane node by specifying the `nodeName` field in the pod definition. This **bypasses the Kubernetes scheduler entirely**.  

📌 **Key takeaway:**  
- **Taints and tolerations affect scheduling decisions made by the scheduler.**  
- **When using manual scheduling (`nodeName` field), the scheduler is not involved, so taints are ignored.**  

We will now proceed with applying tolerations to allow pods to be scheduled despite the taints.

### **Scheduling Pods with Tolerations**  

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      tolerations:
        - key: storage
          operator: Equal
          value: ssd
          effect: NoSchedule
      containers:
        - name: nginx-container
          image: nginx
```
- This deployment **will now schedule pods on `my-second-cluster-worker`**.  

### **Using the Exists Operator**  

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod2
spec:
  containers:
  - name: nginx
    image: nginx
  tolerations:
    - key: storage
      operator: Exists
      effect: NoSchedule
```
- This **pod can be placed on either `my-second-cluster-worker` or `my-second-cluster-worker2`**.  

### **Applying Multiple Taints and Tolerations**  
```sh
kubectl taint nodes my-second-cluster-worker env=prod:NoSchedule
kubectl taint nodes my-second-cluster-worker2 env=dev:NoSchedule
```

```yaml
tolerations:
  - key: storage
    operator: Exists
    effect: NoSchedule
  - key: env
    operator: Equal
    value: prod
    effect: NoSchedule
```
- This pod tolerates **any `storage` taint** and **only `env=prod:NoSchedule`**.  

---

## Key Takeaways  

- **Taints are applied to nodes**, **Tolerations are applied to pods**.  
- A **pod can only be scheduled** on a tainted node if it has a **matching toleration**.  
- **Three effects of taints:** `NoSchedule`, `PreferNoSchedule`, and `NoExecute`.  
- **Operator `Equal` (default)** requires an exact match, while **`Exists` ignores values**.  
- **Taints and tolerations work together** to control **pod placement and node access**.