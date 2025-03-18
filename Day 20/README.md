# Day 20 - Autoscaling in Kubernetes (HPA & VPA) | CKA Course 2025

## Video reference for Day 19 is the following:
[![Watch the video](https://img.youtube.com/vi/Bu4RocrMx0g/maxresdefault.jpg)](https://www.youtube.com/watch?v=Bu4RocrMx0g&ab_channel=CloudWithVarJosh)
 
![Alt text](/images/20.png)

## 📌 What is Scaling?

Scaling refers to **adjusting the resources** available to an application based on its **demand/load**. The key objective is to ensure the app has **enough resources** to handle current traffic, **without wasting resources** when demand is low.

- **Elastic Scaling**: We want the ability to **scale up** when load increases and **scale down** when load decreases.

There are two primary ways to scale:

1. **Adding more servers/pods/instances** (Horizontal Scaling)
2. **Increasing resources (CPU/Memory) of existing servers/pods** (Vertical Scaling)

> **Scaling can be done:**
> - **Manually** (e.g., increasing/decreasing replicas in your Deployment manually)
> - **Automatically** (via Kubernetes objects like HPA & VPA)

---

## 🌐 Types of Scaling

### 1. Horizontal Scaling (Scaling Out/In)

- **Scaling Out**: Adding more instances/pods.
  - In Kubernetes: Increasing the number of pod replicas.
  - Called **Scale-Out**.
  
- **Scaling In**: Removing instances/pods when load decreases.
  - Called **Scale-In**.

💡 Horizontal scaling is **stateless-friendly**, as requests can be distributed across multiple pods.


### 2. Vertical Scaling (Scaling Up/Down)

- **Scaling Up**: Increasing CPU/Memory resources of the existing pod or VM.
- **Scaling Down**: Reducing CPU/Memory.

> ⚠️ **Important Note:**  
> Vertical scaling **often requires a restart** of the resource (pod/VM) because:
> - **VM Example**: The OS needs to recognize new resources.
> - **Pod Example**: Kubernetes needs to recreate the pod with updated resource requests/limits.

---

## 🚀 Manual Scaling Example

Manually scaling Kubernetes Deployments:

```bash
kubectl scale deployment nginx-deploy --replicas=5
```

This increases replicas manually; however, it's not efficient when traffic varies unpredictably (e.g., daytime peaks, nighttime lows). That's where **autoscaling** comes into play.

---

Absolutely, here’s the **clean, structured theory section** for **HPA** as per your requirements—ready for you to paste directly into your `README.md` file.

---

# Horizontal Pod Autoscaler (HPA)

The **Horizontal Pod Autoscaler (HPA)** is a Kubernetes feature that **automatically scales the number of pods in a deployment or workload based on observed resource usage or custom metrics**.

---

## How HPA Works:

- HPA continuously monitors the workload’s resource usage.
- It **adjusts (scales up or scales down) the number of pod replicas** based on observed metrics.
- It operates as a **control loop**, checking metrics at a regular interval (by default, **every 15 seconds**).

---

## Supported Workloads:

HPA supports:

| Kubernetes Object | Supported |
|-------------------|------------|
| **Deployment**    | Yes (most common usage) |
| **StatefulSet**   | Yes |
| **ReplicaSet (RS)** | Yes |
| **ReplicationController (RC)** | Yes |

While HPA works with ReplicaSets and ReplicationControllers, **Deployments and StatefulSets are preferred in modern Kubernetes setups.**

---

## Metrics Used by HPA:

### 1. **Default Resource Metrics (CPU & Memory)**

- The most common use-case is scaling based on **CPU utilization**.
- **Memory utilization** can also be used, though it's less common because memory usage patterns are harder to predict.

These metrics are provided by the **Metrics Server** (explained below).


### 2. **Custom Metrics Support**

HPA also supports scaling based on **custom metrics**—useful when CPU or memory is **not the right indicator of load** (e.g., request rate, queue length).

#### Why Custom Metrics?

Applications might have **business-specific or app-specific metrics** that reflect actual load better than CPU/memory. For example:

- Number of active HTTP requests
- Queue size in a messaging system
- Database connection count

#### Custom Metrics Categories:

| Type | Description | API Used |
|------|-------------|---------|
| **Internal Custom Metrics** | Metrics tied to Kubernetes objects (like pods or deployments) | **Custom Metrics API** |
| **External Metrics** | Metrics not related to Kubernetes objects (external systems, APIs, etc.) | **External Metrics API** |

To utilize custom metrics:

- You need additional components like **Prometheus + Prometheus Adapter** or other monitoring tools like **Datadog**, **InfluxDB**, etc..
- The adapter exposes these metrics to the HPA via the appropriate API.

#### Flow with Custom Metrics


1. **Metric Collection**:
   - Internal and external systems send metrics to the **Prometheus server** running in the cluster.

2. **Prometheus Adapter**:
   - The **Prometheus Adapter** queries the Prometheus server for relevant metrics.  
   - It translates these metrics into a Kubernetes-compatible format.

3. **API Exposure**:
   - The Prometheus Adapter exposes **internal metrics** via the **Custom Metrics API**.  
   - Similarly, it exposes **external metrics** via the **External Metrics API**.

4. **HPA Query and Decision**:
   - The **HPA Controller** queries the appropriate API (Custom or External) to retrieve the metrics.  
   - Using the retrieved metrics, the HPA Controller makes scaling decisions and adjusts the number of pods accordingly.

---

## Metrics Server: Prerequisite for HPA

To enable HPA to fetch CPU and memory utilization, the **Metrics Server** must be installed and running in the cluster.

Metrics Server collects real-time metrics from the kubelets on each node and exposes them through the **metrics.k8s.io API**, which HPA queries.

📚 **Refer to Day 19 to install Metrics Server:**

- [YouTube Video: Day 19](https://www.youtube.com/watch?v=Bu4RocrMx0g&ab_channel=CloudWithVarJosh)
- [GitHub Repo: Day 19](https://github.com/CloudWithVarJosh/CKA-Certification-Course-2025/tree/main/Day%2019)

---

## Demo: Horizontal Pod Autoscaler (HPA)

### Step 1️⃣: **Create Nginx Deployment**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        resources:
          requests:
            cpu: "100m"
          limits:
            cpu: "200m"
        ports:
        - containerPort: 80
```

**Explanation:**

| Field | Description |
|------|-------------|
| `replicas: 1` | Starts with 1 pod |
| `requests.cpu: "100m"` | Minimum CPU resources requested (0.1 core) |
| `limits.cpu: "200m"` | Maximum CPU resources allowed (0.2 core) |
| `labels: app: nginx` | Important for Service selector! |

**Apply:**

```bash
kubectl apply -f nginx-deployment.yaml
```

---

### Step 2️⃣: **Create Kubernetes Service to Expose Pods**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

**Explanation:**

| Field | Description |
|------|-------------|
| `selector: app: nginx` | Matches pods with label `app: nginx` |
| `type: ClusterIP` | Exposes service **internally** inside the cluster |
| `port: 80` | Service listens on port 80 |
| `targetPort: 80` | Forwards to container port 80 |

**Apply:**

```bash
kubectl apply -f nginx-service.yaml
```

---

### Step 3️⃣: **Create HPA Object**

```bash
kubectl autoscale deployment nginx-deploy --cpu-percent=50 --min=1 --max=5
```

**Explanation:**

| Parameter | Description |
|------|-------------|
| `deployment nginx-deploy` | Applies to the nginx deployment |
| `--cpu-percent=50` | Target **50% CPU utilization** |
| `--min=1` | Minimum 1 pod |
| `--max=5` | Maximum 5 pods |

📌 **What happens:**

- Kubernetes will monitor **average CPU usage** of all nginx pods.
- If usage > 50%, HPA **scales out** (adds pods).
- If usage < 50%, HPA **scales in** (removes pods, but maintains minimum 1).

---

### Step 4️⃣: **Check HPA Status**

```bash
kubectl get hpa
```

**Explanation:**

This will show:
- Current CPU utilization.
- Current replicas.
- Scaling status.

---

### Step 5️⃣: **Generate CPU Load Using BusyBox**

```bash
kubectl run -it --rm load-generator --image=busybox /bin/sh
```

**Explanation:**

| Command | Description |
|------|-------------|
| `kubectl run` | Launches a temporary pod |
| `-it` | Interactive shell |
| `--image=busybox` | Uses lightweight BusyBox image |
| `/bin/sh` | Opens shell |
| `--rm` | Delete pod upon exit |

Once inside:

```bash
while true; do wget -q -O- http://nginx-svc; done
```

**Explanation:**

| Part | Meaning |
|------|-------------|
| `while true; do ... done` | Infinite loop |
| `wget -q` | Quiet HTTP GET request |
| `-O-` | Output discarded |
| `http://nginx-svc` | Sends requests to nginx pods via **Service DNS name** |

**Single** command to achieve the above:

`kubectl run -it --rm load-generator --image=busybox -- /bin/sh -c "while true; do wget -q -O- http://nginx-svc; done"`

---

### Step 6️⃣: **Observe Auto-Scaling**

Monitor HPA and pods:

```bash
kubectl get hpa
kubectl get pods -w
```

**Behavior:**
- As load increases, **CPU utilization rises**.
- HPA will scale from 1 pod up to max 5 if needed.
- Once load decreases (stop load-generator), HPA scales back down.

---

## **Did You Know?**

> **Kubernetes defaults to targeting 80% CPU usage when no `--cpu-percent` is set!**  
>  
> It optimizes for high resource efficiency while keeping enough headroom for sudden spikes.  
>  
> You can explicitly set it, but Kubernetes **automatically sets it to 80%**!

---

### **Key Takeaways:**

- HPA automatically adjusts **replica count** based on **average CPU utilization**.
- It requires:
  - Metrics Server
  - CPU resource requests/limits
- We can simulate load using busybox + wget.
- HPA works as a control loop, not instant — runs at **15-second intervals**.
- Helps handle fluctuating traffic **without manual intervention**!

---

## Vertical Pod Autoscaler (VPA) 

**Vertical Pod Autoscaler (VPA)** in Kubernetes is used to **automatically adjust the CPU and memory requests and limits of your pods based on their actual usage**.

It helps you ensure:

- Pods **always have enough resources to perform efficiently**.
- **Wasted resources are minimized**, avoiding over-provisioning.

---

### **Key Behavior:**

1. **VPA focuses on vertical scaling:**
   - It **increases or decreases the CPU & memory reservations of pods**.
   - Unlike HPA, it does **not change the number of replicas**.

2. **Pod Restarts:**
   - In order to apply updated resource requests/limits, **VPA will restart pods.**
   - This is crucial because Kubernetes **cannot update CPU/Memory requests on a running pod** — the pod must be recreated.

---

## **Why use VPA?**

- **Removes guesswork.** No need to manually tune CPU/memory requests.
- Ideal for **stateful or single-replica apps** where HPA isn’t helpful.
- Useful in **dynamic workloads** where resource demands frequently change.
- Helps maintain **performance stability without wasting resources**.

---

## **VPA Operating Modes:**

| Mode | Behavior |
|------|--------|
| **Off** | VPA only gives recommendations; it does NOT change pod resources automatically. |
| **Initial** | VPA sets recommended CPU/memory when pod is created. **No changes after creation.** |
| **Auto** | VPA **actively updates pod resources and restarts pods when needed** based on usage patterns. |

---

## Components of VPA:

1. ### **Recommender**
   - Continuously **analyzes historical and current resource usage**.
   - Predicts **optimal CPU/memory requests** to avoid under-provisioning or over-provisioning.
   - Uses statistical models (machine learning behind the scenes) to adapt recommendations over time.

2. ### **Updater**
   - **Identifies pods needing resource adjustments.**
   - Evicts & restarts them to apply new CPU/memory settings.
   - Respects **Pod Disruption Budgets (PDBs)** to maintain application availability and prevent too many disruptions at once.
     - Pod Disruption Budgets (PDBs) ensure a minimum number of pods remain available during voluntary disruptions, like upgrades or scaling actions. 

3. ### **Admission Controller**
   - **Intercepts pod creation requests.**
   - Modifies CPU & memory requests at creation time based on the latest VPA recommendations.
   - Ensures pods **start with the right resource allocations immediately.**

---

## **Benefits of VPA:**

| Benefit | Explanation |
|--------|------------|
| **Cost Efficiency & Stability** | VPA prevents over-provisioning (wasting resources) and under-provisioning (performance degradation), saving cloud costs while ensuring stable app performance. |
| **Cluster Resource Optimization** | Continuously right-sizes pods based on real usage, freeing up cluster resources for other workloads. |
| **Elimination of Manual Benchmarking** | No need to manually analyze or guess CPU/memory needs for every application; VPA automates it based on real data. |

**VPA as a Resource Recommendation Tool**

Even if you decide **not to use VPA in Auto mode in production**—to avoid pod restarts or potential downtime—**VPA can still be extremely valuable for recommending accurate CPU and memory requests and limits.**  
It is particularly useful when developers are unable to provide precise resource requirements for their applications. By running VPA in **"Off" mode**, you can safely collect real-world usage data and let VPA suggest optimal resource settings without automatically applying them.

---

# Demo: Vertical Pod Autoscaler (VPA)

## **Step 1️⃣ - Install VPA Components**

To install VPA components, follow the official installation process:

📄 **Reference:**
[VPA Installation Guide](https://github.com/kubernetes/autoscaler/blob/master/vertical-pod-autoscaler/docs/installation.md#install-command)

### Commands:

```bash
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler
./hack/vpa-up.sh
```

✔️ This installs:

- **VPA Recommender**
- **VPA Updater**
- **VPA Admission Controller**

### Optional Uninstall:

Later, to uninstall VPA:

```bash
./hack/vpa-down.sh
```

---

## **Step 2️⃣ - Create Nginx Deployment & Service**

### **Deployment YAML (`nginx-vpa.yaml`):**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        resources:
          requests:
            cpu: "100m"
            memory: "100Mi"
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```
---

### **Apply the YAML:**

```bash
kubectl apply -f nginx-vpa.yaml
```

**Why 2 replicas?**

- **Ensures zero-downtime** when VPA restarts pods.
- Demonstrates how VPA safely rolls out changes.

---

## **Step 3️⃣ - Create VPA Object**

**VPA YAML (`vpa.yaml`):**

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: nginx-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind:       Deployment
    name:       nginx-deploy
  updatePolicy:
    # updateMode options:
    # "Off"     - VPA only recommends resources, does NOT apply them.
    # "Initial" - VPA sets recommended resources at pod creation, no changes after.
    # "Auto"    - VPA automatically updates resources and restarts pods as needed.
    updateMode: "Auto"
```

**Apply:**

```bash
kubectl apply -f nginx-vpa.yaml
```

---

## **Step 4️⃣ - Check VPA Recommendations**

Run:

```bash
kubectl describe vpa nginx-vpa
```

### **Explaining Key Output:**

```bash
Recommendation:
  Container Recommendations:
    Container Name:  nginx
    Lower Bound:
      Cpu:     25m
      Memory:  262144k
    Target:
      Cpu:     63m
      Memory:  262144k
    Uncapped Target:
      Cpu:     63m
      Memory:  262144k
    Upper Bound:
      Cpu:     1711m
      Memory:  312499697
```

| Field | Explanation |
|------|-------------|
| **Lower Bound** | Minimum safe CPU/memory requests. Pod should not be sized below this. |
| **Target** | Optimal CPU/memory requests based on observed usage. |
| **Uncapped Target** | Target recommendation without considering any max limits. Often same as Target unless limits exist. |
| **Upper Bound** | Maximum safe CPU/memory requests. Resources above this are unnecessary. |

👉 **Pod CPU/Memory will be adjusted toward the Target values.**

---

## **Step 5️⃣ - Generate Load to Trigger VPA Adjustments**

Run a **BusyBox pod** to generate continuous load:

```bash
kubectl run -it --rm load-generator --image=busybox /bin/sh
```

Inside:

```bash
while true; do wget -q -O- http://nginx-svc; done
```

### 📌 **Explanation:**

| Command | Description |
|------|-------------|
| `kubectl run` | Launches a temporary pod |
| `-it` | Interactive shell |
| `--image=busybox` | Uses lightweight BusyBox image |
| `/bin/sh` | Opens shell |
| `--rm` | Delete pod upon exit |


**Single** command to achieve the above:

`kubectl run -it --rm load-generator --image=busybox -- /bin/sh -c "while true; do wget -q -O- http://nginx-svc; done"`

---
## **Step 6️⃣ - Observe VPA Actions**

### 1️⃣ **Describe VPA to Check Updates:**

```bash
kubectl describe vpa nginx-vpa
```

Check:

- Updated recommendations
- Actions taken (pod evictions)

---

### 2️⃣ **Check Events:**

```bash
kubectl get events
```

**Explanation:**

- VPA-related events (like pod evictions) are logged here.
- Useful to monitor when VPA applies changes.

---

## References:
  - [HPA Documentation](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
  - [VPA Documentation](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler)
  - [Latest CKA Exam Curriculam](https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/)
  - [VPA Installation Guide](https://github.com/kubernetes/autoscaler/blob/master/vertical-pod-autoscaler/docs/installation.md#install-command)

---
