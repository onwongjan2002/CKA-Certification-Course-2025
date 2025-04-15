# Day 28: Kubernetes ConfigMaps and Secrets | CKA Course 2025

## Video reference for Day 28 is the following:


---
## ⭐ Support the Project  
If this **repository** helps you, give it a ⭐ to show your support and help others discover it! 

---


### **Introduction**

In modern DevOps practices, it’s important to separate configuration data from application logic. Hardcoding environment-specific settings (like service URLs, logging levels, or deployment profiles) into container images makes applications less portable and harder to maintain.

This is where **ConfigMaps** come in.

---

## ConfigMaps
### **Why Do We Need ConfigMaps?**

In real-world Kubernetes deployments:

- Maintaining a large number of hardcoded environment variables in Pod specs can become unmanageable.
- You may want to update configuration without rebuilding your container image.
- You might need to inject configuration data via environment variables, CLI arguments, or mounted files.

**ConfigMaps** allow you to **decouple configuration from your container images**, making your applications more portable and easier to manage.

---

### **What is a ConfigMap?**

A **ConfigMap** is a Kubernetes API object used to store **non-confidential key-value configuration data**. 

Pods can consume ConfigMaps in three primary ways:

1. As **environment variables**
2. As **command-line arguments** (Less Common)
3. As **configuration files via mounted volumes**

> **Important:** ConfigMaps do not provide encryption or secrecy. For sensitive information, use a **Secret** instead.



**What are Environment Variables?**

Environment variables are key-value pairs used by the operating system and applications to store configuration settings. They help control how processes behave without modifying the underlying code.

**Example on a Linux system:**

```bash
export APP_NAME=frontend
export ENVIRONMENT=production
```

You can access these values using the `echo` command:

```bash
echo $APP_NAME     # Output: frontend
echo $ENVIRONMENT  # Output: production
```

These variables are commonly used in shell scripts or passed to applications during execution to configure them dynamically.

---

### **Without ConfigMap: Hardcoded Environment Variables**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend-container
          image: nginx
          env:
            - name: APP
              value: frontend
            - name: ENVIRONMENT
              value: production
```

While this works, hardcoding environment variables is not scalable or reusable across environments.

---

## **Demo 1: Using ConfigMap as Environment Variables**

For this demo, we’ll build on the `frontend-deploy` configuration introduced in **Day 12** of this course. If you'd like a deeper understanding of **Kubernetes Services**, feel free to explore the following resources:

- [Day 12 GitHub Notes](https://github.com/CloudWithVarJosh/CKA-Certification-Course-2025/tree/main/Day%2012)

- [Day 12 YouTube Video](https://www.youtube.com/watch?v=92NB8oQBtnc)

### Step 1: Create ConfigMap

```yaml
apiVersion: v1  # Defines the API version for the Kubernetes object.
kind: ConfigMap  # Declares this resource as a ConfigMap.
metadata:
  name: frontend-cm  # The name used to reference this ConfigMap in pods or other resources.
data:
  APP: "frontend"  # Key-value pair representing configuration data (e.g., application name).
  ENVIRONMENT: "production"  # Key-value pair used to define the deployment environment.

```

Apply:

```bash
kubectl apply -f frontend-cm.yaml
```

---

### Step 2: Reference ConfigMap in Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deploy  # Name of the Deployment resource
spec:
  replicas: 3  # Number of pod replicas to maintain
  selector:
    matchLabels:
      app: frontend  # Selector to match pods managed by this Deployment
  template:
    metadata:
      labels:
        app: frontend  # Label assigned to pods created by this Deployment
    spec:
      containers:
      - name: frontend-container  # Name of the container within each pod
        image: nginx  # Using the official Nginx image
        env:
        - name: APP  # Environment variable name inside the container
          valueFrom:
            configMapKeyRef:
              name: frontend-cm  # Name of the ConfigMap from which the value is pulled
              key: APP  # Key in the ConfigMap whose value is used for this environment variable
        - name: ENVIRONMENT  # Another environment variable inside the container
          valueFrom:
            configMapKeyRef:
              name: frontend-cm
              key: ENVIRONMENT

        # NOTE:
        # The value of key "APP" will be picked up from the key "APP" in the ConfigMap named "frontend-cm".
        # The 'name' field here (APP) is the environment variable name that will be visible inside the container.
        # It does NOT need to match the 'key' from the ConfigMap.
        # For example:
        # - You could write: name: SERVICE and key: APP
        # - Inside the container, the environment variable would be called SERVICE and hold the value of APP from the ConfigMap.
```

Apply:

```bash
kubectl apply -f frontend-deploy.yaml
```

---

### Verification

Pick a Pod from the Deployment:

```bash
kubectl get pods -l app=frontend
```

Then exec into one of them:

```bash
kubectl exec -it <pod-name> -- printenv | grep -E 'APP|ENVIRONMENT'
```

Expected Output:

```
APP=frontend
ENVIRONMENT=production
```

---

## **Demo 2: Using ConfigMap as a Configuration File (Volume Mount)**

For this demo, we’ll build on the `frontend-deploy` configuration introduced in **Day 12** of this course. If you'd like a deeper understanding of **Kubernetes Services**, feel free to explore the following resources:

- [Day 12 GitHub Notes](https://github.com/CloudWithVarJosh/CKA-Certification-Course-2025/tree/main/Day%2012)

- [Day 12 YouTube Video](https://www.youtube.com/watch?v=92NB8oQBtnc)

### Step 1: Modify ConfigMap to Include HTML File

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: frontend-cm  # Name of the ConfigMap, used for reference in Pods or Deployments.
data:
  APP: "frontend"  # Key-value pair that can be used as an environment variable in a Pod.
  ENVIRONMENT: "production"  # Another environment variable for defining the environment.
  index.html: |  # When mounted as a volume, this key creates a file named 'index.html' with the following contents.
    <!DOCTYPE html>
    <html>
    <head><title>Welcome</title></head>
    <body>
      <h1>This page is served by nginx. Welcome to Cloud With VarJosh!!</h1>
    </body>
    </html>
```

Apply:

```bash
kubectl apply -f frontend-cm.yaml
```

---

### Step 2: Mount ConfigMap Volume in Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deploy  # Name of the Deployment
spec:
  replicas: 3  # Number of pod replicas
  selector:
    matchLabels:
      app: frontend  # Label selector to identify Pods managed by this Deployment
  template:
    metadata:
      labels:
        app: frontend  # Label to match with the selector above
    spec:
      containers:
      - name: frontend-container  # Name of the container
        image: nginx  # NGINX image to serve web content
        env:
        - name: APP
          valueFrom:
            configMapKeyRef:
              name: frontend-cm  # Name of the ConfigMap to pull the key from
              key: APP  # Key in the ConfigMap whose value will become the value of the env variable APP
        - name: ENVIRONMENT
          valueFrom:
            configMapKeyRef:
              name: frontend-cm
              key: ENVIRONMENT  # Key in the ConfigMap whose value becomes the value of ENVIRONMENT variable
        volumeMounts:
        - name: html-volume
          mountPath: /usr/share/nginx/html  # Mount point inside the container (default HTML dir for NGINX)
          readOnly: true  # Make the volume read-only
      volumes:
      - name: html-volume
        configMap:
          name: frontend-cm  # The ConfigMap being mounted as a volume
          items:
          - key: index.html  # The key in the ConfigMap to be used as a file
            path: index.html  # The filename that will be created inside the mount path
```

Understand that **ConfigMaps are mounted as directories** in Kubernetes. If you try to mount a ConfigMap at a path that is already a file (like `/usr/share/nginx/html/index.html`), the container will **fail to start**, because Kubernetes **cannot mount a directory over an existing file**.

That’s why we only use `/usr/share/nginx/html` as the `mountPath`, which is a **directory**, not a file.

If you **omit the `items:` section**, like this:

```yaml
volumes:
  - name: html-volume
    configMap:
      name: frontend-cm
```

Then **all the keys** in the ConfigMap (`APP`, `ENVIRONMENT`, `index.html`) will be mounted as **individual files** inside `/usr/share/nginx/html`.

So inside the container, you’d get:
```
/usr/share/nginx/html/APP
/usr/share/nginx/html/ENVIRONMENT
/usr/share/nginx/html/index.html
```

If you **only want `index.html`** to be mounted (and not `APP`, `ENVIRONMENT`), you should use the `items:` section like this:

```yaml
volumes:
  - name: html-volume
    configMap:
      name: frontend-cm
      items:
        - key: index.html
          path: index.html
```

---

### What if you *do* want to mount the ConfigMap **as a single file**?

You cannot mount a directory on top of an existing file — but you **can mount a specific key** as a file using `subPath`. Here’s how:

```yaml
volumeMounts:
  - name: html-volume                    # References the volume named 'html-volume' defined in the 'volumes' section
    mountPath: /usr/share/nginx/html/index.html  # The exact file path inside the container where the file should be mounted
    subPath: index.html                  # Mount only the 'index.html' key from the ConfigMap as a file (not the whole directory)

```

This tells Kubernetes:  
> “Mount only the file named `index.html` from the ConfigMap into the container at `/usr/share/nginx/html/index.html`.”

This works even if `index.html` already exists in the NGINX image because you're **not replacing a file with a directory**, but a file with a file.

---

**Advantages of Using `subPath`**

- Allows **fine-grained control** over what file is mounted and where.
- Helps you **inject configuration** (like `index.html`) **without overriding entire directories**.
- Enables **mixing static content** from image and dynamic content from ConfigMap.


**Disadvantages of `subPath`**

- The mounted file is **not updated automatically** if the ConfigMap changes (unless the pod is restarted).
- Slightly more complex syntax.
- If the mount path already exists and is a file, mounting may fail without proper use of `subPath`.

---

So, in summary:

- Use `mountPath: /usr/share/nginx/html` to mount a **directory**.
- Use `subPath` to mount a **single file** precisely over a specific path.
- Use `items:` in the volume to **control which keys** from the ConfigMap are mounted.



Apply:

```bash
kubectl apply -f frontend-deploy.yaml
```

---

## **Step 3: Expose the Deployment using a NodePort Service**

We’ll now expose our NGINX-based frontend deployment using a **NodePort** service. This allows access to the application from outside the cluster by targeting any worker node’s IP and the assigned NodePort.

### NodePort Service Manifest

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc  # Name of the service
spec:
  type: NodePort       # Exposes the service on a static port on each node
  selector:
    app: frontend      # Targets pods with this label
  ports:
    - protocol: TCP
      port: 80         # Port exposed inside the cluster (ClusterIP)
      targetPort: 80   # Port the container is listening on
      nodePort: 31000  # External port accessible on the node (must be between 30000–32767)
```

Apply the service:

```bash
kubectl apply -f frontend-svc.yaml
```
---

## **Verification**

### 1. Get the Pod name

```bash
kubectl get pods -l app=frontend
```

### 2. Verify the `index.html` content inside the container

```bash
kubectl exec -it <pod-name> -- cat /usr/share/nginx/html/index.html
```

**Expected Output:**

```html
<!DOCTYPE html>
<html>
<head><title>Welcome</title></head>
<body>
  <h1>This page is served by nginx. Welcome to Cloud With VarJosh!!</h1>
</body>
</html>
```

---

### 3. Access the Application via NodePort

You can access the application using:

```bash
http://<Node-IP>:31000
```

Or with `curl`:

```bash
curl http://<Node-IP>:31000
```

> Replace `<Node-IP>` with the IP address of any node in your cluster (typically a worker node).

---

### 4. **If You're Using KIND (Kubernetes in Docker)**

If you've been following this course using a **KIND cluster**, the worker nodes are running inside a Docker container. In this case, KIND maps NodePorts to your host (Mac/Windows/Linux) via localhost.

So you can directly access the app using:

```bash
http://localhost:31000
```

Or:

```bash
curl http://localhost:31000
```

This will display the content served by NGINX from the ConfigMap-mounted `index.html`.

--- 


## **Best Practices for ConfigMap**

- Avoid storing sensitive data in ConfigMaps.
- Mount only the necessary keys using `items` to limit volume content.
- Use version control tools (e.g., Kustomize, Helm) to manage changes.
- Combine ConfigMaps with `subPath` if you want to mount specific files.

---

## Kubernetes Secrets
### **Why Do We Need Secrets in Kubernetes?**

In production environments, applications often require access to sensitive data such as:

- Database credentials
- API tokens
- SSH private keys
- TLS certificates

Storing these directly in your container image or Kubernetes manifests as plain text is insecure. **Kubernetes Secrets** provide a way to manage this data securely.

---

### **What is a Kubernetes Secret?**

A **Secret** is a Kubernetes object used to store and manage sensitive information. Secrets are base64-encoded and can be made more secure by:

- Enabling encryption at rest
- Limiting RBAC access to Secrets
- Using external secret managers like HashiCorp Vault, AWS Secrets Manager, or Sealed Secrets

Accessible to Pods via:
  1. **Environment variables**
  2. **Mounted volumes (as files)**
  3. **Command-line arguments** (less common)

> Note: By default, Kubernetes stores secrets unencrypted in `etcd`. It is recommended to enable encryption at rest for better security.

---

### **Demo 1: Injecting Secrets into a Pod**

We’ll enhance the existing `backend-deploy` and `backend-svc` resources by securely injecting database credentials using a Kubernetes Secret.


For this demo, we’ll build on the `backend-deploy` configuration introduced in **Day 12** of this course. If you'd like a deeper understanding of **Kubernetes Services**, feel free to explore the following resources:

- [Day 12 GitHub Notes](https://github.com/CloudWithVarJosh/CKA-Certification-Course-2025/tree/main/Day%2012)

- [Day 12 YouTube Video](https://www.youtube.com/watch?v=92NB8oQBtnc)

These resources provide clear examples and explanations of how Kubernetes Services work in real-world deployments.



### **Step 1: Create the Secret**

Create a Secret manifest `backend-secret.yaml`:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: backend-secret
type: Opaque
data:
  DB_USER: YmFja2VuZHVzZXI=          # base64 for 'backenduser'
  DB_PASSWORD: c3VwZXJzZWNyZXQ=      # base64 for 'supersecret'
```

To generate the base64-encoded values:

```bash
echo -n 'backenduser' | base64      # YmFja2VuZHVzZXI=
echo -n 'supersecret' | base64      # c3VwZXJzZWNyZXQ=
```

Apply the Secret:

```bash
kubectl apply -f backend-secret.yaml
```

---

### **Step 2: Update the Deployment to Use the Secret**

Here’s the updated deployment manifest (`backend-deploy.yaml`) using the `backend-secret` Secret as environment variables:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deploy
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: backend-container
          image: hashicorp/http-echo
          args:
            - "-text=Hello from Backend"
          env:
            - name: DB_USER
              valueFrom:
                secretKeyRef:
                  name: backend-secret
                  key: DB_USER
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: backend-secret
                  key: DB_PASSWORD
```

Apply the deployment:

```bash
kubectl apply -f backend-deploy.yaml
```

---

### **Step 3: Verify Secret Injection**

1. List the running Pods:

```bash
kubectl get pods -l app=backend
```

2. Exec into one of the Pods:

```bash
kubectl exec -it <pod-name> -- /bin/sh
```

3. Print the environment variables:

```bash
echo $DB_USER       # Expected output: backenduser
echo $DB_PASSWORD   # Expected output: supersecret
```

---

### **Alternative: Mount Secret as Files**

If you prefer to mount secrets as files (instead of environment variables), here’s how:

#### Update container spec:

```yaml
        volumeMounts:
          - name: secret-volume
            mountPath: /etc/secrets
            readOnly: true
```

#### Add volume to spec:

```yaml
      volumes:
        - name: secret-volume
          secret:
            secretName: backend-secret
```

This will create files at `/etc/secrets/DB_USER` and `/etc/secrets/DB_PASSWORD`.

---

### **Service (Unchanged)**

The existing service manifest remains valid and doesn’t require modification:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-svc
spec:
  type: ClusterIP
  ports:
    - protocol: TCP
      port: 9090
      targetPort: 5678
  selector:
    app: backend
```

Apply the service:

```bash
kubectl apply -f backend-svc.yaml
```

---

## **Best Practices for Using Kubernetes Secrets**

- Avoid storing secrets in Git repositories.
- Use `kubectl create secret` or Helm to avoid manually encoding data.
- Enable encryption at rest in `etcd`.
- Use Role-Based Access Control (RBAC) to restrict access to secrets.
- Use external secret managers for enhanced security and auditability.
- Consider using `subPath` when mounting specific keys from a Secret as individual files.

---

### **Understanding Dynamic Updates with ConfigMaps and Secrets**

- Environment variables defined via `env.valueFrom.configMapKeyRef` or `secretKeyRef` are **evaluated only once** when the pod starts.
- Updating the underlying ConfigMap or Secret **does not affect** the values already injected as environment variables in a running container.
- ConfigMaps or Secrets mounted as **volumes** (without `subPath`) **do reflect updates dynamically** inside the container.
- Kubernetes handles dynamic updates using **symlinks** to new file versions, but the application must **re-read the files** to detect changes.
- When mounting individual keys using `subPath`, the file is **copied**, not symlinked, so updates to the ConfigMap or Secret **will not propagate**.
- To enable live updates without restarting pods, prefer **volume mounts without `subPath`** and ensure the application supports **hot reloading** or use a **config-reloader**.