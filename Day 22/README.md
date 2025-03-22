# Day 22: Health probes | CKA Course 2025

## Video reference for Day 22 is the following:
[![Watch the video](https://img.youtube.com/vi/VEwP_wF67Tw/maxresdefault.jpg)](https://www.youtube.com/watch?v=VEwP_wF67Tw&ab_channel=CloudWithVarJosh)

![Alt text](/images/22.png)

### **Day 22: Health Probes in Kubernetes**

---

#### **Introduction**
In Kubernetes, health probes are used to check the status of applications running inside pods. These probes help Kubernetes ensure that only healthy pods are receiving traffic, and unhealthy pods are either given time to recover or restarted as necessary. The **kubelet** is responsible for performing these probes.

---

### **Types of Health Probes**

1. **Startup Probes**
   - **Purpose**: Used for **legacy or slow-starting applications** that take variable amounts of time to initialize.
   - **Problem Solved**: The `initialDelaySeconds` parameter cannot always capture the correct startup time for such applications. Setting it too high causes unnecessary delays, and too low leads to premature restarts.
   - **Behavior**:
     - When a startup probe is defined, **liveness and readiness probes do not start** until the startup probe succeeds.
   - **Example**:
     ```yaml
     startupProbe:
      httpGet:
        path: /healthz
        port: 8080
      failureThreshold: 30  # Kubernetes (via Kubelet) will attempt the probe up to 30 times before failing
      periodSeconds: 10     # Probe runs every 10 seconds

      # Explanation:
      # - This probe is responsible for determining when the application has successfully started.
      # - It sends an HTTP GET request to /healthz on port 8080.
      # - It will try every 10 seconds, up to 30 times (total grace period = 300 seconds).
      # - If all 30 attempts fail, Kubernetes will mark the pod as failed and stop trying.
      # - No restart happens because the app never started properly.

     ```
     This configuration ensures the app has **up to 5 minutes** (30 * 10 = 300 seconds) to initialize.

---

2. **Readiness Probes (RP)**
   - **Purpose**: Checks if a container is **ready to start accepting traffic**.
   - **Behavior**:
     - When the readiness probe fails, the pod is removed from the service's load balancer, but **the container is not restarted**.
   - **Key Advantage**: Prevents routing traffic to containers that are temporarily unable to serve requests due to initialization delays or resource constraints.
   - **Example**:
     ```yaml
      readinessProbe:
        httpGet:
          path: /readyz
          port: 8080
        initialDelaySeconds: 5  # Wait 5 seconds after container starts before probing
        periodSeconds: 10       # Probe every 10 seconds

        # Explanation:
        # - Checks if the application is ready to serve traffic.
        # - HTTP GET request is sent to /readyz on port 8080.
        # - Starts after an initial delay of 5 seconds.
        # - If probe fails, the pod is temporarily removed from Service endpoints (no restart).
        # - Once probe passes, pod starts receiving traffic.
     ```
   - **Use Case**:
     - A database connection might temporarily fail. During this time, the readiness probe ensures that traffic is not sent to the affected pod.

---

3. **Liveness Probes (LP)**
   - **Purpose**: Checks if a container is **alive and functioning correctly**.
   - **Behavior**:
     - When the liveness probe fails, **the pod is restarted**.
   - **Key Advantage**: Helps recover from **irrecoverable failures**, such as deadlocks or unresponsive applications.
   - **Example**:
     ```yaml
      livenessProbe:
        exec:
          command:
          - cat
          - /tmp/healthy
        initialDelaySeconds: 3  # Start checking 3 seconds after the container starts
        periodSeconds: 5        # Check every 5 seconds

        # Explanation:
        # - Verifies if the application is still alive.
        # - Executes 'cat /tmp/healthy' inside the container.
        # - If this command fails (e.g., file missing), the liveness probe fails.
        # - **Kubelet will restart the container automatically** to recover from the failure.

     ```
   - **Use Case**:
     - Detects and restarts applications stuck in an unrecoverable state (e.g., infinite loop or deadlock).

---

### **Why Configure Both Readiness and Liveness Probes?**

While it may seem that configuring only liveness probes is enough, **best practice dictates using both** for the following reasons:
- **Readiness Probes (RP)**:
  - Do not restart a failing pod; they just stop sending traffic to it.
  - This ensures that **transient issues** (e.g., high load or temporary database unavailability) do not unnecessarily trigger a restart.
  - Pods can continue running and recover without interruption.
- **Liveness Probes (LP)**:
  - Designed for **critical, irrecoverable issues** where a restart is the only solution.
  - Ensures that **dead or hung pods are recreated**, preserving application availability.

**Key Insight**:
- Without RP: Traffic may still be sent to pods experiencing temporary failures, degrading user experience.
- Without LP: Pods stuck in fatal states will remain idle and waste resources.

---

### **Kubernetes Health Probes: Comprehensive Comparison**

Here is the revised table with the important words and concepts **bolded** to highlight key details:

| **Aspect**                     | **Startup Probe**                                                                | **Readiness Probe**                                                               | **Liveness Probe**                                                                |
|--------------------------------|----------------------------------------------------------------------------------|----------------------------------------------------------------------------------|----------------------------------------------------------------------------------|
| **Purpose**                    | Ensure **slow-starting apps** are given enough time to start                     | Check if the pod is **ready to receive traffic**                                 | Check if the pod is **alive and functioning properly**                           |
| **When Does It Start?**        | **Immediately** after container starts                                           | Starts **when the pod enters the "Running" state** and **after Startup Probe succeeds (if configured)** | Starts **when the pod enters the "Running" state** and **after Startup Probe succeeds (if configured)** |
| **Failure Behavior**           | Pod is **killed & restarted** after failure threshold is exceeded during startup | Pod is **marked unready** and removed from **Service load balancers**; not restarted | Pod is **killed & restarted** after failure threshold exceeded                   |
| **Effect on Pod Traffic**      | **No direct effect**                                                             | **Stops sending traffic** to the pod                                             | **No direct effect** on traffic                                                  |
| **Common Use Case**            | **Legacy apps**, slow-booting apps with unpredictable startup times              | Apps needing time to **initialize external dependencies or configurations**       | Recover from **stuck apps**, **deadlocks**, or **memory leaks**                  |
| **Recovery Mechanism**         | Pod is **restarted** during initialization                                       | **No restart**; waits for probe to pass                                          | Pod is **restarted**                                                             |
| **Runs Until**                 | Probe **succeeds** → switches control to **RP & LP**                             | Runs **continuously** throughout the pod's lifecycle                             | Runs **continuously** throughout the pod's lifecycle                             |
| **Starts Again After Restart?**| **Yes**                                                                          | **Yes**                                                                          | **Yes**                                                                          |
| **Relation to Service Traffic**| **None**                                                                         | Directly **influences Service endpoints**                                        | **None**                                                                         |
| **Frequency (Default)**        | Configurable (e.g., `periodSeconds`, `failureThreshold`)                          | Configurable (e.g., `initialDelaySeconds`, `periodSeconds`)                      | Configurable (e.g., `initialDelaySeconds`, `periodSeconds`)                      |
| **Mechanisms Supported**       | **HTTP GET**, **TCP Socket**, **Exec Command**                                   | **HTTP GET**, **TCP Socket**, **Exec Command**                                   | **HTTP GET**, **TCP Socket**, **Exec Command**                                   |
| **Best Practices**             | Use for **apps with unpredictable boot times**                                   | **Always** use in production to avoid sending traffic to **unready pods**        | **Always** use to automatically recover from **unrecoverable failures**          |
| **Administrative Benefit**     | Avoid unnecessary **premature restarts** for **slow-start apps**                 | Prevent unnecessary **load and requests** hitting pods that **aren't ready**     | Ensures **automatic recovery** from **failures**                                 |
| **Impact on Resources**        | Uses **CPU/memory** during startup period                                        | Pod consumes resources but doesn't receive traffic if **unready**                | Pod consumes resources until **restarted**                                       |
| **Pod State While Failing**    | Pod **restarts** after failure                                                   | Pod **stays running** but removed from **Service traffic**                       | Pod **restarts** after failure                                                   |
| **Typical Example**            | **Java apps** taking >60s to start, **legacy monoliths**                         | App needs **DB connection** or **configuration loaded** before accepting requests | **Deadlock**, app frozen, **memory leak**, or stuck threads                      |

---

### **Probe Mechanisms**

Kubernetes provides **three mechanisms** for performing health probes:

1. **HTTP GET Requests**:
   - The kubelet sends an HTTP GET request to a specified endpoint.
   - **Success**: Status codes 200–399.
   - **Failure**: Any other status code.
   - **Example**:
     ```yaml
     livenessProbe:
       httpGet:
         path: /healthz
         port: 8080
       initialDelaySeconds: 3
       periodSeconds: 5
     ```
     In this case, the app responds with HTTP `200` to indicate health.

2. **TCP Socket**:
   - The kubelet checks if a TCP connection can be established to a specified port.
   - **Success**: Connection is successful.
   - **Failure**: Connection cannot be established.
   - **Example**:
     ```yaml
     readinessProbe:
       tcpSocket:
         port: 3306
       initialDelaySeconds: 10
       periodSeconds: 5
     ```
     This is useful for services like databases that listen on specific ports.

3. **Command Execution**:
   - The kubelet runs a specified command inside the container.
   - **Success**: Command exits with code `0`.
   - **Failure**: Command exits with a non-zero code.
   - **Example**:
     ```yaml
     livenessProbe:
       exec:
         command:
         - cat
         - /tmp/healthy
       initialDelaySeconds: 5
       periodSeconds: 10
     ```
     In this example, the app creates a `/tmp/healthy` file when healthy. If the file is missing, the probe fails.

---

### **Conclusion**
By combining **Startup**, **Readiness**, and **Liveness Probes**, you can:
- Protect slow-starting apps from premature terminations.
- Ensure only **ready** pods receive traffic, enhancing application stability.
- Recover from fatal application issues by restarting pods with **liveness probes**.

These probes allow Kubernetes to dynamically manage the health of your applications, ensuring availability and a better user experience.

---

### **Demo: Startup Probe**
**Objective**: Ensure slow-starting applications are given sufficient time to initialize without premature restarts.

#### YAML Configuration:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: startup-probe-demo
spec:
  containers:
  - name: slow-app
    image: busybox
    command: ["sh", "-c", "sleep 30; touch /tmp/healthy; sleep 300"]
    startupProbe:
      exec:
        command:
        - cat             # Executes 'cat' command to check if a specific file exists
        - /tmp/healthy    # Path to the file that indicates the app has started successfully
      failureThreshold: 5 # Kubelet will attempt the probe 5 times before considering it failed
      periodSeconds: 10   # Probe runs every 10 seconds
    # Explanation:
    # - This probe waits for the container to initialize by verifying the existence of /tmp/healthy.
    # - The container sleeps for 30 seconds before creating /tmp/healthy.
    # - If the file doesn't exist after 5 attempts, Kubelet considers the pod failed and doesn't restart it.

```

#### Test:
1. Apply the pod configuration:
   ```bash
   kubectl apply -f <file>
   ```
2. Observe the pod's status:
   ```bash
   kubectl describe pod startup-probe-demo
   ```
   Look for probe-related events under the `Events` section.
3. Simulate the probe succeeding:
   - Access the container and create the file `/tmp/healthy`:
     ```bash
     kubectl exec -it startup-probe-demo -- touch /tmp/healthy
     ```
   - Verify the pod is running and hasn’t restarted.
4. Simulate the probe failing:
   - Delete `/tmp/healthy` and observe the pod events:
     ```bash
     kubectl exec -it startup-probe-demo -- rm /tmp/healthy
     kubectl describe pod startup-probe-demo
     ```
   - The pod will eventually restart.

---

### **Demo: Readiness Probe**
**Objective**: Ensure the application is ready to handle traffic and prevent unready pods from receiving requests.

#### YAML Configuration:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readiness-probe-demo
spec:
  containers:
  - name: web-app
    image: nginx
    readinessProbe:
      httpGet:
        path: /healthz       # Kubernetes sends HTTP GET requests to this endpoint
        port: 80             # Port number where the readiness probe is executed
      initialDelaySeconds: 5 # Waits for 5 seconds before starting the readiness probe
      periodSeconds: 10      # Probe runs every 10 seconds
    # Explanation:
    # - The readiness probe ensures the pod is ready to serve traffic.
    # - Kubelet removes the pod from service endpoints if the probe fails.
    # - This prevents traffic from being routed to an unready pod while it is initializing or temporarily unhealthy.

```

#### Test:
1. Apply the pod configuration:
   ```bash
   kubectl apply -f <file>
   ```
2. Check the readiness state:
   ```bash
   kubectl get pods readiness-probe-demo
   ```
   Confirm the pod’s `READY` column reflects the probe state.
3. Expose the pod as a service:
   ```bash
   kubectl expose pod readiness-probe-demo --type=ClusterIP --port=80
   kubectl get endpoints
   ```
   Verify the pod is part of the service endpoints.
4. Simulate readiness probe failure:
   - Block `/healthz` in the container:
     ```bash
     kubectl exec readiness-probe-demo -- sh -c "echo 'fail' > /usr/share/nginx/html/healthz"
     ```
   - Confirm the pod is removed from service endpoints:
     ```bash
     kubectl get endpoints
     ```

---

### **Demo: Liveness Probe**
**Objective**: Restart the pod if it becomes unresponsive or stuck.

#### YAML Configuration:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-probe-demo
spec:
  containers:
  - name: faulty-app
    image: busybox
    command: ["sh", "-c", "touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 300"]
    livenessProbe:
      tcpSocket:
        port: 8080           # Kubernetes checks TCP connectivity on port 8080
      initialDelaySeconds: 10 # Kubelet waits 10 seconds before starting the probe
      periodSeconds: 15       # Probe runs every 15 seconds
    # Explanation:
    # - The liveness probe ensures the container is still functioning correctly.
    # - Initially, the app creates /tmp/healthy, but it deletes it after 30 seconds (simulating failure).
    # - If the probe fails, Kubelet restarts the container to recover from the failure.

```

#### Test:
1. Apply the pod configuration:
   ```bash
   kubectl apply -f <file>
   ```
2. Observe liveness events:
   ```bash
   kubectl describe pod liveness-probe-demo
   ```
   Look for probe failure events.
3. Simulate probe failure:
   - Block the `tcpSocket` probe by stopping the app’s process:
     ```bash
     kubectl exec liveness-probe-demo -- sh -c "rm -rf /tmp/healthy"
     ```
   - Confirm the pod is restarted by checking the `RESTARTS` column:
     ```bash
     kubectl get pods liveness-probe-demo
     ```

---

### **Demo: Combined Probes Demo**
**Objective**: Show all three probes (Startup, Readiness, and Liveness) working together.

#### YAML Configuration:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: combined-probes-demo
spec:
  containers:
  - name: comprehensive-app
    image: busybox
    command: ["sh", "-c", "sleep 20; touch /tmp/ready; sleep 300"]
    startupProbe:
      exec:
        command:
        - cat             # Executes 'cat' command to check if a specific file exists
        - /tmp/ready      # Path to the file that indicates the app has started successfully
      failureThreshold: 3  # Kubelet will attempt the probe 3 times before considering it failed
      periodSeconds: 5     # Probe runs every 5 seconds
    readinessProbe:
      httpGet:
        path: /healthz       # Kubernetes sends HTTP GET requests to this endpoint
        port: 8080           # Port number where the readiness probe is executed
      initialDelaySeconds: 25 # Waits for 25 seconds before starting the readiness probe
      periodSeconds: 10       # Probe runs every 10 seconds
    livenessProbe:
      exec:
        command:
        - cat             # Executes 'cat' command to check if a specific file exists
        - /tmp/ready      # Path to the file indicating the app is still functioning correctly
      initialDelaySeconds: 30 # Kubelet waits 30 seconds before starting the liveness probe
      periodSeconds: 15       # Probe runs every 15 seconds
    # Explanation:
    # - This pod combines all three probes:
    #   - Startup Probe ensures the app initializes correctly before readiness/liveness probes run.
    #   - Readiness Probe ensures traffic is routed only to ready pods.
    #   - Liveness Probe ensures pods that fail irrecoverably are restarted automatically.
```
#### Test:
1. Apply the combined configuration:
   ```bash
   kubectl apply -f <file>
   ```
2. Simulate success/failure for each probe:
   - Startup Probe: Delete or create `/tmp/ready` within 15 seconds of startup.
     ```bash
     kubectl exec combined-probes-demo -- rm /tmp/ready
     ```
   - Readiness Probe: Block `/healthz` on port `8080` and check service endpoints.
     ```bash
     kubectl exec combined-probes-demo -- sh -c "echo 'fail' > /usr/share/nginx/html/healthz"
     kubectl get endpoints
     ```
   - Liveness Probe: Remove `/tmp/ready` after startup.
     ```bash
     kubectl exec combined-probes-demo -- rm /tmp/ready
     kubectl get pods combined-probes-demo
     ```

---

