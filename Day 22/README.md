# Day 23: Health probes | CKA Course 2025

## Video reference for Day 23 is the following:
[![Watch the video](https://img.youtube.com/vi/VEwP_wF67Tw/maxresdefault.jpg)](https://www.youtube.com/watch?v=VEwP_wF67Tw&ab_channel=CloudWithVarJosh)

![Alt text](/images/22.png)


## **Introduction**

"Hello everyone, and welcome to Day 23 of our Cloud/DevOps course! Today, we’re diving deep into the life of a Pod—how it behaves under different policies, what happens when you delete it, and how to diagnose common problems. We’ll cover termination handling, restart and image pull policies, the pod lifecycle phases, and a deep dive into common pod statuses. By the end of this session, you’ll understand not just what these terms mean but also how they come into play in real-world environments."

---

## **1. Pod Deletion and Termination Signals**

### **What Happens When You Run `kubectl delete pod`?**

- **Command Execution:**  
  When you run:
  ```bash
  kubectl delete pod <pod-name>
  ```
  Kubernetes initiates a graceful termination sequence rather than an abrupt shutdown.

- **Grace Period & Signals:**  
  - **SIGTERM:** As soon as the delete command is issued, Kubernetes sends the SIGTERM signal to the main process of each container inside the pod. This signal tells the application: “It’s time to shut down gracefully. Please wrap up any ongoing tasks.”
  - **Termination Grace Period:** By default, Kubernetes waits for 30 seconds (configurable via `terminationGracePeriodSeconds` in the pod spec) for the container to exit gracefully.
  - **SIGKILL:** If the container does not exit within this grace period, Kubernetes sends a SIGKILL signal to force an immediate shutdown.

### **Example: Graceful Shutdown of a Web Server**

Imagine you have a Pod running an NGINX server. With a graceful shutdown enabled in your container code, it might finish serving current requests and close connections cleanly when receiving a SIGTERM. If it takes too long—over the grace period—SIGKILL will terminate it immediately.

**YAML Snippet:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx- graceful
spec:
  terminationGracePeriodSeconds: 30
  containers:
  - name: nginx
    image: nginx:latest
```
In this YAML, if you issue a delete command, Kubernetes gives the NGINX container 30 seconds to wrap up before force-killing it.

---

## **2. Restart Policies**

Restart policies control what Kubernetes does when a container in a pod exits. There are three types:

### **a. Always**  
- **Use Case:** Commonly used in Deployments.  
- **Behavior:** The container is restarted regardless of its exit code.
- **Example:**  
  In a typical web application where continuous availability is paramount, using `Always` ensures the pod is perpetually running.

### **b. OnFailure**  
- **Use Case:** Often used in Jobs or batch processing.  
- **Behavior:** The container is restarted only if it exits with a non-zero (failure) status.
- **Example:**  
  Think of a data processing job; if it fails due to an error, Kubernetes will try to restart it. However, if the job completes successfully with exit code 0, it won’t be restarted.

### **c. Never**  
- **Use Case:** Used when restarting is undesirable.  
- **Behavior:** The container is not restarted even if it exits with a failure code.
- **Example:**  
  This policy is useful for one-off diagnostic pods where you want to see the result of a container execution without further restarts.

**Example in a Pod Spec:**
```yaml
spec:
  restartPolicy: OnFailure
  containers:
  - name: batch-processor
    image: my-batch-image:latest
```

---

## **3. Image Pull Policies**

Image pull policies determine whether Kubernetes should pull the container image from a registry when creating a container.

### **a. Always**  
- **Behavior:** Kubernetes always pulls the latest image.
- **Use Case:** Use this during active development when updates are frequent.
- **Example:**  
  ```yaml
  imagePullPolicy: Always
  ```

### **b. IfNotPresent**  
- **Behavior:** Kubernetes pulls the image only if it is not already present on the node.
- **Use Case:** Often used in production to reduce startup time and bandwidth usage.
- **Example:**  
  ```yaml
  imagePullPolicy: IfNotPresent
  ```

### **c. Never**  
- **Behavior:** Kubernetes never pulls the image; it must already exist on the node.
- **Use Case:** Rarely used—good for air-gapped environments or when you’re absolutely sure the image is local.
- **Example:**  
  ```yaml
  imagePullPolicy: Never
  ```

**Full Container Example:**
```yaml
containers:
- name: app
  image: my-app:latest
  imagePullPolicy: IfNotPresent
```

---

## **4. Pod Lifecycle Phases and Transitions**

Pods progress through several distinct phases:

### **a. Pending**
- **Description:** The pod has been accepted by the Kubernetes system, but one or more of the containers has not yet been created. This phase includes scheduling and pulling the container images.
- **Example Scenario:**  
  When you deploy a new pod, it might remain in Pending while Kubernetes downloads the necessary Docker image.

### **b. Running**
- **Description:** The pod has been bound to a node, and all containers are either running or in the process of starting.
- **Example Scenario:**  
  After the image is pulled and the container starts, the pod status shifts to Running.

### **c. Succeeded**
- **Description:** All containers in the pod have terminated normally, and they will not be restarted.
- **Example Scenario:**  
  Completed batch or job pods usually end up in the Succeeded state.

### **d. Failed**
- **Description:** All containers in the pod have terminated, and at least one container has terminated in failure. No further restarts will occur.
- **Example Scenario:**  
  A job that crashes due to an error will transition into Failed.

### **e. Unknown**
- **Description:** The state of the pod cannot be obtained, often due to a communication error with the node.
- **Example Scenario:**  
  If a node goes offline unexpectedly, the pod might enter an Unknown state.

**ASCII Flowchart Overview:**

```
       +---------+
       | Pending |
       +---------+
            |
            v
       +---------+      If all containers end normally
       | Running |  ------------------------->
       +---------+           Succeeded
            |
            v
       +---------+      If any container fails and no restart.
       |  Failed |
       +---------+
```

---

## **5. Pod Status Deep Dive**

Understanding pod status is crucial for debugging:

### **Common Pod Statuses:**

#### **a. CrashLoopBackOff**
- **What It Means:**  
  A pod’s container is repeatedly crashing and being restarted. Kubernetes pauses between restarts using an exponential backoff strategy.
- **Common Causes:**  
  - Errors in the application code
  - Misconfiguration (e.g., missing environment variables)
  - Uncaught exceptions
- **Diagnostic Tip:**  
  Inspect logs using:
  ```bash
  kubectl logs <pod-name>
  ```
  If you suspect the container restarts are hiding the error, try:
  ```bash
  kubectl logs <pod-name> --previous
  ```

#### **b. ImagePullBackOff**
- **What It Means:**  
  Kubernetes is unable to pull the specified image. This status indicates that Kubernetes has attempted to download the image, encountered an error, and is now waiting before retrying.
- **Common Causes:**  
  - Incorrect image name or tag
  - Authentication issues with the container registry
- **Diagnostic Tip:**  
  Use:
  ```bash
  kubectl describe pod <pod-name>
  ```
  This command will show events related to image pulling, such as "Failed to pull image" along with error details.

#### **c. ErrImagePull**
- **What It Means:**  
  Similar to ImagePullBackOff, this status usually precedes it and shows that the image failed to pull.
- **Diagnostic Tip:**  
  Check:
  ```bash
  kubectl describe pod <pod-name>
  ```
  Note the events section for clues like "ErrImagePull".

#### **d. Other Causes of Crash**
- **Common errors:**  
  - Misconfigured command or entrypoint in the Dockerfile.
  - Memory or CPU constraints causing the container to be OOMKilled.
- **Diagnostic Commands:**
  - `kubectl describe pod <pod-name>` – For events, conditions, and container statuses.
  - `kubectl logs <pod-name>` – For application-level error logs.

---

## **6. Debugging Common Pod Errors**

### **Step-by-Step Debugging Workflow:**

1. **Inspect the Pod Description:**
   - Run:
     ```bash
     kubectl describe pod <pod-name>
     ```
   - **What to Look For:**  
     - Check the events section at the bottom.
     - Look for clues like “Failed to pull image” or “Back-off restarting failed container.”

2. **Check Container Logs:**
   - Run:
     ```bash
     kubectl logs <pod-name>
     ```
   - If the container is crashing repeatedly (CrashLoopBackOff), get logs from the previous instance:
     ```bash
     kubectl logs <pod-name> --previous
     ```
   - **What to Look For:**  
     - Application error messages
     - Exception stack traces or resource limitations (OOM errors)

3. **Validate Configuration:**
   - Inspect your pod YAML for correct image names, resource limits, and environment variables.
   - Confirm the container’s command/entrypoint settings (CMD vs. ENTRYPOINT in Docker).

4. **Recreate Scenarios in a Test Pod:**
   - Sometimes isolating a problematic configuration in a minimal pod helps diagnose the issue. Use a simple test container to simulate environment variables or image pull policies.

---

## **Conclusion**

"In conclusion, today’s lesson covered the entire lifecycle of a Pod—from creation, through its various states, to deletion and graceful termination. We discussed termination signals (SIGTERM, SIGKILL) and how they influence shutdown behavior, examined restart and image pull policies to understand container recovery and deployment scenarios, and finally, took a deep dive into pod statuses like CrashLoopBackOff and ImagePullBackOff. By learning to leverage `kubectl describe` and `kubectl logs`, you now have a systematic approach to debugging and monitoring your applications in Kubernetes.

Remember: understanding these internals is key not only for passing certification exams but also for managing production-grade systems."