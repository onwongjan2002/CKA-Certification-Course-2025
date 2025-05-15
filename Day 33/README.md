# Day 33: TLS in Kubernetes MASTERCLASS | PART 3 | Private CAs, TLS Auth Flow, mTLS in Kubernetes Cluster | CKA 2025

## Video reference for Day 33 is the following:

---
## ⭐ Support the Project  
If this **repository** helps you, give it a ⭐ to show your support and help others discover it! 

Here’s the revised version of your pre-requisites section without the book emoji:

---

### Pre-Requisites for Day 33

Before you dive into Day 33, make sure you have gone through the following days to get a better understanding:


1. **Day 7**: Kubernetes Architecture
   Understanding Kubernetes components and their roles will help you grasp when each component acts as a client or server.

   * **GitHub**: [Day 7 Repo](https://github.com/CloudWithVarJosh/CKA-Certification-Course-2025/tree/main/Day%2007)
   * **YouTube**: [Day 7 Video](https://www.youtube.com/watch?v=-9Cslu8PTjU&ab_channel=CloudWithVarJosh)

2. **Day 30**: How HTTPS & SSH Work, Encryption, and Its Types
   The concepts of encryption, as well as HTTPS and SSH mechanisms, will be essential in understanding security within Kubernetes.

   * **GitHub**: [Day 30 Repo](https://github.com/CloudWithVarJosh/CKA-Certification-Course-2025/tree/main/Day%2030)
   * **YouTube**: [Day 30 Video](https://www.youtube.com/watch?v=MkGPyJqCkB4&ab_channel=CloudWithVarJosh)

3. **Day 31**: TLS in Kubernetes MASTERCLASS | PART 1 | Mutual Auth (SSH), TLS 1.3, Types of CAs
   Start your journey into TLS by understanding client-server trust models, SSH mutual auth, 1-way TLS with TLS 1.3, and different types of Certificate Authorities.

   * **GitHub**: [Day 31 Repo](https://github.com/CloudWithVarJosh/CKA-Certification-Course-2025/tree/main/Day%2031)
   * **YouTube**: [Day 31 Video](https://www.youtube.com/watch?v=afQIvp5a0wM&ab_channel=CloudWithVarJosh)

4. **Day 32**: TLS in Kubernetes MASTERCLASS | PART 2 | mTLS, kubeconfig & Kubernetes Context
   Deep dive into mutual TLS (mTLS), the kubeconfig file structure, and how Kubernetes context enables managing multiple clusters and namespaces efficiently.

   * **GitHub**: [Day 32 Repo](https://github.com/CloudWithVarJosh/CKA-Certification-Course-2025/tree/main/Day%2032)
   * **YouTube**: [Day 32 Video](https://www.youtube.com/watch?v=VBlI0IG4ReI&ab_channel=CloudWithVarJosh)

---

## Table of Contents

1. [Introduction](#introduction)   
    6.1. [Why mTLS in Kubernetes?](#why-mtls-in-kubernetes)  
    6.2. [Private CAs in Kubernetes Clusters](#private-cas-in-kubernetes-clusters)  
7. [Kubernetes Components as Clients and Servers](#kubernetes-components-as-clients-and-servers)  
8. [Private Keys and Certificates in Kubernetes](#what-are-private-keys-and-certificates)  
    8.1. [Key-Pairs for Kubernetes Components](#key-pairs-for-kubernetes-components)  
    8.2. [API Server Key-Pairs](#understanding-api-server-key-pairs-in-kubernetes)  
9. [Granting Cluster Access to a New User (Seema)](#granting-cluster-access-to-a-new-user-seema-using-certificates-and-rbac)  
10. [References](#references)


---

## Introduction

In this session, we explore how **TLS (Transport Layer Security)** is implemented within a Kubernetes cluster. While TLS is commonly associated with secure communication over the internet, Kubernetes uses it extensively for **securing internal communication** between its various components.

By the end of this session, you will have a clear understanding of how TLS and **mutual TLS (mTLS)** work across Kubernetes components such as `kubectl`, `kube-apiserver`, `etcd`, `controller-manager`, `scheduler`, and others.

---

### **Private CAs in Kubernetes Clusters**
In Kubernetes, **TLS certificates secure communication** between core components. These certificates are signed by a **private Certificate Authority (CA)**, ensuring authentication and encryption. Depending on security requirements, a cluster can be configured to use:

* A **single CA** for the entire cluster (simpler, easier to manage), or
* **Multiple CAs** (e.g., a separate CA for etcd) for added security and isolation.

**Why Use Multiple CAs?**

Using **multiple private CAs** strengthens security by **isolating trust boundaries**. This approach is particularly useful for sensitive components like **etcd**, which stores the **entire cluster state**.

Security Considerations:
- If **etcd is compromised**, an attacker could gain full control over the cluster.
- To **limit the blast radius** in case of a CA or key compromise, it's common to **assign a separate CA exclusively for etcd**.
- Other control plane components (e.g., API server, scheduler, controller-manager) can share a **different CA**, ensuring **compartmentalized trust**.

By segmenting certificate authorities, Kubernetes operators can **reduce risk** and **enhance security posture**.

You can configure Kubernetes components to **trust a private CA** by distributing the CA’s public certificate to each component’s trust store. For example:

* `kubectl` trusts the API server because it has the **private CA** certificate that signed the API server's certificate.
* Similarly, components like the `controller-manager`, `scheduler`, and `kubelet` trust the API server and each other using certificates signed by this **shared private CA**.

Most Kubernetes clusters—whether provisioned via tools like `kubeadm`, `k3s`, or through managed services like **EKS**, **GKE**, or **AKS**—automatically generate and manage a **private CA** during cluster initialization. This CA is used to issue certificates for key components, enabling **TLS encryption and mutual authentication** out of the box.

When using **managed Kubernetes services**, this private CA is maintained by the cloud provider. It remains **hidden from users**, but all internal components are configured to trust it, ensuring secure communication without manual intervention.

However, when setting up a cluster **“the hard way”** (e.g., via [Kelsey Hightower’s guide](https://github.com/kelseyhightower/kubernetes-the-hard-way)), **you are responsible for creating and managing the entire certificate chain**. This means:

* Generating a root CA certificate and key,
* Signing individual component certificates,
* And distributing them appropriately.

While this approach offers **maximum transparency and control**, it also demands a solid understanding of **PKI, TLS, and Kubernetes internals**.


**Do Enterprises Use Public CAs for Kubernetes?**

Enterprises do **not** use public Certificate Authorities (CAs) for **core Kubernetes internals**. Instead, they rely on **private CAs**—either auto-generated (using tools like `kubeadm`) or centrally managed—to sign certificates used by Kubernetes components like the API server, kubelet, controller-manager, and scheduler. These certificates facilitate secure **TLS encryption and mutual authentication** within the cluster.

For **public-facing services**, however, it's common to use **public CAs**. Components such as:

* Ingress controllers
* Load balancers
* Gateway API implementations

...require certificates trusted by browsers. In these cases, enterprises use public CAs (e.g., Let’s Encrypt, DigiCert, GlobalSign) to issue TLS certificates, ensuring a secure **HTTPS** experience for users and avoiding browser trust warnings.

In short:

* **Private CAs** → Used for internal Kubernetes communication
* **Public CAs** → Used for securing external-facing applications
* ⚠️ Public CAs are **not** used for control plane or internal Kubernetes components
---

### Kubernetes Components as Clients and Servers

In the diagram below, arrows represent the direction of client-server communication:

* The **arrow tail** indicates the **client**, and the **arrowhead** points to the **server**.
* Some arrows have **only arrowheads** (e.g., between **kubelet** and **API server**) to indicate that **the server initiates the connection** in specific cases:

  * When a user runs commands like `kubectl logs` or `kubectl exec`, the **API server acts as the client**, reaching out to the **kubelet**.
  * Conversely, when the **kubelet pushes node or pod health data**, it becomes the **client**, and the **API server** is the **server**.
* The **etcd arrow is colored yellow** to indicate that **etcd always acts as a server**, receiving requests from the API server.

---

### **Client (Initiates a Request)**

1. **kubectl**: **Interacts with the API server**, used by admins and DevOps engineers for cluster management, deployment, viewing logs.
2. **Scheduler**: **Requests the API server** for pod scheduling, checks for unscheduled pods, manages resource placement.
3. **API Server**: **Communicates with etcd**, **interacts with kubelet** for logs, exec commands, etc.
4. **Controller Manager**: **Requests the API server** to verify desired vs. current state, manages controllers, ensures cluster state matches desired configuration.
5. **Kube-Proxy**: **Communicates with the API server** for service discovery and endpoints, acts as a network router for traffic.
6. **Kubelet**: **Reports to the API server** about node health and pod status, fetches ConfigMaps & Secrets, ensures desired containers are running.


### **Server (Responds to the Request)**

1. **API Server**: **Responds to kubectl**, admins, DevOps, and third-party clients, manages resources and cluster state.
2. **etcd**: **Responds to API server**, stores cluster configuration, state, and secrets (only the API server interacts with etcd).
3. **Kubelet**: **Responds to API server** for pod status, logs, and exec commands.

> The roles mentioned above for clients and servers are **indicative**, not exhaustive. Kubernetes components may interact in multiple ways, and their responsibilities can evolve as the system grows and new features are added.

**Client or Server? It Depends on the Context**

In Kubernetes, whether a component acts as a **client** or **server** depends entirely on the direction of the request.

A single component can play both roles depending on the scenario. For example, when a user or a tool like `kubectl` accesses the **API server**, the API server acts as the **server**. However, when the **API server** communicates with another component like the **kubelet**—for fetching logs (`kubectl logs`), executing (`kubectl exec`) into containers, or retrieving node and pod status—the API server becomes the **client**, and the kubelet acts as the **server**.

Components such as the **scheduler**, **controller manager**, and **kube-proxy** are always **clients** because they initiate communication with the **API server** to get the desired cluster state, pod placements, or service endpoints.

On the other hand, **etcd** is **always a server** in the Kubernetes architecture. It **only** communicates with the **API server**, which acts as its **client**—no other component talks to etcd directly. This design keeps etcd isolated and secure, as it holds the cluster’s source of truth.

---

### Key-Pairs for Kubernetes Components

| **Component**             | **Private Key**                                         | **Certificate**                                  | **Default Location**                      | **Purpose / Usage**                                                                 |
| ------------------------- | ------------------------------------------------------- | ------------------------------------------------ | ----------------------------------------- | ----------------------------------------------------------------------------------- |
| **API Server**            | `apiserver.key`                                         | `apiserver.crt`                                  | `/etc/kubernetes/pki/`                    | Serves HTTPS; proves server identity to clients.                                    |
| **API Server → etcd**     | `apiserver-etcd-client.key`                             | `apiserver-etcd-client.crt`                      | `/etc/kubernetes/pki/`                    | Authenticates API server to etcd via mTLS.                                          |
| **API Server → Kubelet**  | `apiserver-kubelet-client.key`                          | `apiserver-kubelet-client.crt`                   | `/etc/kubernetes/pki/`                    | Used by API server to authenticate to kubelets.                                     |
| **Controller Manager**    | Referenced via `controller-manager.conf` → `client-key` | `controller-manager.conf` → `client-certificate` | `/etc/kubernetes/controller-manager.conf` | Authenticates to the API server via mTLS using a kubeconfig with embedded cert/key. |
| **Scheduler**             | Referenced via `scheduler.conf` → `client-key`          | `scheduler.conf` → `client-certificate`          | `/etc/kubernetes/scheduler.conf`          | Authenticates to the API server via mTLS using a kubeconfig with embedded cert/key. |
| **Kubelet (per node)**    | `kubelet.key`                                           | `kubelet.crt`                                    | `/var/lib/kubelet/pki/`                   | Authenticates kubelet to API server.                                                |
| **Kube Proxy (per node)** | `kube-proxy.key`                                        | `kube-proxy.crt`                                 | `/var/lib/kube-proxy/`                    | Authenticates kube-proxy to API server.                                             |
 | **etcd Server**      | `etcd/etcd.key`        | `etcd/etcd.crt`        | `/etc/kubernetes/pki/etcd/` | Presented by the etcd server to authenticate to clients (e.g., kube-apiserver).           |
 | **etcd Peer**        | `etcd/etcd-peer.key`   | `etcd/etcd-peer.crt`   | `/etc/kubernetes/pki/etcd/` | Used for mTLS between etcd nodes in a multi-node etcd (HA) setup.                         |
 | **etcd Client**      | `etcd/etcd-client.key` | `etcd/etcd-client.crt` | `/etc/kubernetes/pki/etcd/` | 	Only needed if etcd itself acts as a client, e.g., to another etcd node in very specific setups.              |
 | **Main CA**          | `ca.key`               | `ca.crt`               | `/etc/kubernetes/pki/`      | Root CA for signing control plane component certs (API server, controller-manager, etc.). |
| **Service Account Token** | `sa.key`                                                | `sa.pub`                                         | `/etc/kubernetes/pki/`                    | Used to sign service account tokens (for pod-to-API authentication).                |

---


## Example: mTLS Between Scheduler (Client) and API Server (Server)

Let’s walk through an end-to-end explanation of how **mutual TLS (mTLS)** works in Kubernetes — specifically, the communication between the **kube-scheduler** (client) and the **kube-apiserver** (server).

This example will show you **how to figure out certificate paths and their roles** by inspecting configuration files — **not by memorizing paths**.

---

### 🔑 Key Principle: All Clients Use a Kubeconfig File to Connect to the API Server

> **IMPORTANT:** Anything that connects to the API server — whether it's `kubectl`, a control plane component like the scheduler or controller-manager, or an automation tool — uses a **kubeconfig file** to do so.

We already saw this in action in a previous lecture when **Seema**, using `kubectl`, connected to the API server using her personal `~/.kube/config` file. The same logic applies here: the **scheduler is also a client**, and its communication with the API server is facilitated by a kubeconfig file — in this case, `/etc/kubernetes/scheduler.conf`.

> 🔍 For `kube-proxy`, since it runs as a **DaemonSet**, its kubeconfig file is mounted into each pod. You can `kubectl exec` into a `kube-proxy` pod and find the kubeconfig typically at:
> `/var/lib/kube-proxy/kubeconfig`

This principle not only helps you understand **how TLS authentication is wired** in Kubernetes, but also gives you a **consistent mental model** for troubleshooting and inspecting certificates.

---

## 📌 Important File Locations (on the Control Plane Node)

| Purpose                          | Path                        |
| -------------------------------- | --------------------------- |
| Kubeconfig files                 | `/etc/kubernetes/*.conf`    |
| Static Pod manifests             | `/etc/kubernetes/manifests` |
| API server & control plane certs | `/etc/kubernetes/pki`       |
| etcd certificates                | `/etc/kubernetes/pki/etcd`  |
| Kubelet certs                    | `/var/lib/kubelet/pki`      |

---

## 🔐 SERVER-SIDE: How the API Server Presents Its Certificate

**Goal:** Identify what certificate the API server presents and how the scheduler verifies it.

### Step 1: Know Who’s the Server

* In this scenario: **Scheduler is the client**, **API server is the server**.

### Step 2: How Does the Server Present a Certificate?

To identify what certificate the API server presents, we need to look at its configuration. Since we know from [Day 15](https://github.com/CloudWithVarJosh/CKA-Certification-Course-2025/tree/main/Day%2015) that control plane components run as **static pods**, we check:

```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

Search for this flag:

```yaml
--tls-cert-file=/etc/kubernetes/pki/apiserver.crt
```

This tells us the API server presents the certificate at:

```bash
/etc/kubernetes/pki/apiserver.crt
```

You can inspect it:

```bash
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
```

From this, you’ll see:

* **Subject**: CN=kube-apiserver
* **Issuer**: CN=kubernetes (a self-signed root CA)

This proves that:

* The API server identifies itself as `kube-apiserver`.
* The certificate is signed by the Kubernetes **cluster CA**.

### Step 3: How Does the Client (Scheduler) Trust the Server?

To verify the server’s certificate, the **scheduler must trust the CA** that signed it.

So how does the scheduler get that CA cert?

Let’s move to the client side.

---

## 🔑 CLIENT-SIDE: How the Scheduler Authenticates to the API Server

### Step 1: Find the Kubeconfig File

As per our **Key Principle**, the scheduler uses a kubeconfig file to connect to the API server.

Check the static pod manifest:

```bash
cat /etc/kubernetes/manifests/kube-scheduler.yaml
```

Look for the `--kubeconfig` flag:

```yaml
--kubeconfig=/etc/kubernetes/scheduler.conf
```

This is the **kubeconfig file** used by the scheduler.

---

### Step 2: How Does the Scheduler Verify the API Server?

Open the kubeconfig file:

```bash
cat /etc/kubernetes/scheduler.conf
```

Look for this field:

```yaml
certificate-authority-data: <base64 encoded certificate>
```

This field contains the **CA certificate** (base64 encoded) that the scheduler uses to validate the API server’s certificate.

To decode and inspect it:

```bash
echo -n "<base64 data>" | base64 --decode > ca.crt
openssl x509 -in ca.crt -text -noout
```

You’ll see the **Issuer** and **Subject** are both `CN=kubernetes` — meaning this is a **self-signed root CA**.

This tells us:

* The scheduler **trusts the Kubernetes CA**.
* That’s how it **authenticates the API server's identity**.

---

  **Why Do Popular Certificate Authorities Also Have a Root CA?**
  Many well-known Certificate Authorities (CAs), like **Sectigo**, **Let's Encrypt**, or **DigiCert**, operate their own **Root Certificate Authority (Root CA)** in addition to one or more **Intermediate CAs**.

  **What Is a Root CA and Why Does It Exist?**
  * A **Root CA** sits at the **top of the trust hierarchy** — it is the anchor of the entire Public Key Infrastructure (PKI).
  * A **root certificate is always self-signed** because there is no higher authority to sign it. That’s why you might sometimes see warnings like “this certificate is self-signed.”
  * However, **self-signed is acceptable** when the certificate belongs to a **trusted root**, which is explicitly trusted by operating systems, browsers, and other clients.

  **Why Have a Separate Root and Intermediate CA?**
  * **Security:** The root CA’s private key is kept **offline or in secure environments**. Intermediate CAs handle day-to-day certificate issuance, minimizing exposure.
  * **Resilience:** If an **intermediate CA is compromised**, it can be revoked without affecting the root. This limits the blast radius.
  * **Best Practice:** This layered structure follows industry standards for security and trust management.

---


### Step 3: How Does the Scheduler Authenticate Itself to the API Server?

Now flip the flow.

The scheduler, being the client, also needs to **present its own certificate** to the API server. This is mutual TLS.

Back in the same kubeconfig file:

```yaml
users:
- name: system:kube-scheduler
  user:
    client-certificate-data: <base64 encoded cert>
    client-key-data: <base64 encoded key>
```

These two fields contain:

* The scheduler’s **public certificate**
* Its **private key**

Decode the certificate:

```bash
echo -n "<client-certificate-data>" | base64 --decode > scheduler.crt
openssl x509 -in scheduler.crt -text -noout
```

You’ll see:

* **Subject:** `CN=system:kube-scheduler`
* **Issuer:** `CN=kubernetes`

This confirms:

* The scheduler is identifying itself as `system:kube-scheduler`.
* Its identity is trusted by the API server since the certificate is signed by the **same CA** that the API server trusts.

How do we know the API server trusts it?

Check the API server manifest again:

```bash
--client-ca-file=/etc/kubernetes/pki/ca.crt
```

So the API server will **only accept client certificates** signed by this CA. The scheduler's certificate qualifies.

---

## 🧠 Conclusion: Focus on Config Files, Not Hardcoded Paths

Here’s what we’ve learned:

* You **don’t need to memorize** cert locations.
* All control plane components expose their configuration either via:

  * **`--tls-cert-file`** (for server-side certs)
  * **`--kubeconfig`** (for client-side certs)

Instead of trying to remember file paths, **just remember where to look**: the static pod YAMLs under `/etc/kubernetes/manifests/`.

---

### Summary Table

| Component  | What It Does                          | Where to Look                                   | What to Look For                                                           |
| ---------- | ------------------------------------- | ----------------------------------------------- | -------------------------------------------------------------------------- |
| API Server | Presents TLS cert to clients          | `/etc/kubernetes/manifests/kube-apiserver.yaml` | `--tls-cert-file`, `--client-ca-file`                                      |
| Scheduler  | Verifies server, presents client cert | `/etc/kubernetes/scheduler.conf`                | `certificate-authority-data`, `client-certificate-data`, `client-key-data` |

---


