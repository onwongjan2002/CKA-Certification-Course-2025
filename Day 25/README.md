# Day 25: Kubernetes Core & Extensions Explained | CNI, CSI, CRI & Add-Ons | CKA Course 2025

## Video reference for Day 25 is the following:

---
## ⭐ Support the Project  
If this **repository** helps you, give it a ⭐ to show your support and help others discover it! 

---

![Alt text](/images/25.png)
## Table of Contents  

1. [Introduction](#introduction)  
2. [Kubernetes Core and Extended Architecture](#kubernetes-core-and-extended-architecture)  
   - [What is Kubernetes Core?](#what-is-kubernetes-core)  
   - [Control Plane Components](#1️⃣-control-plane-components)  
   - [Node Components](#2️⃣-node-components)  
3. [Kubernetes Extensions: Beyond the Core](#kubernetes-extensions-beyond-the-core)  
   - [Plugins](#1️⃣-plugins)  
   - [Add-Ons](#2️⃣-add-ons)  
   - [Third-Party Extensions](#3️⃣-third-party-extensions)  
4. [Deep Dive into Kubernetes Interfaces](#deep-dive-into-kubernetes-interfaces)  
   - [Overview of Kubernetes Interfaces](#overview-of-kubernetes-interfaces)  
5. [Why Kubernetes Uses a Plugin-Based Architecture?](#why-kubernetes-uses-a-plugin-based-architecture)  
6. [Key Takeaways](#key-takeaways)  
7. [Conclusion](#conclusion)  
8. [References](#references)  


---

# **Introduction**  
Kubernetes is built on a **modular and extensible architecture**, where the **core components** (Control Plane and Node Components) handle fundamental orchestration tasks, while **plugins, add-ons, and third-party extensions** enhance functionality without bloating the system. This lecture explores the **Kubernetes Core**, its **plugin-based architecture**, and key extension mechanisms like **CNI (networking), CSI (storage), and CRI (container runtimes)**. We’ll also discuss how this design ensures **scalability, flexibility, and vendor neutrality**, enabling seamless integration with diverse cloud-native tools.  

---

# **Kubernetes Core and Extended Architecture**  
Kubernetes is a powerful container orchestration platform built on a **modular design**. This modularity enables **flexibility, scalability, and extensibility**. The **core components** handle essential orchestration tasks, while **plugins, add-ons, and third-party extensions** extend functionality without bloating the core system.  

This section clarifies the distinction between **Kubernetes Core** and **External Extensions**, explaining how various components interact within the overall architecture.  

---

## **Kubernetes Core Architecture**  

### **What is Kubernetes Core?**  
The **Kubernetes Core** consists of essential built-in components responsible for cluster management and orchestration. These components are categorized into **Control Plane Components** and **Node Components**.  

### **1️⃣ Control Plane Components**  
The control plane manages cluster operations, making scheduling decisions and maintaining the desired state.  

- **API Server:** The central control plane component that validates and processes REST API requests, enforcing security policies and forwarding requests to relevant components.  
- **Scheduler:** Assigns Pods to nodes based on resource availability and scheduling policies.  
- **Controller Manager:** Runs control loops to maintain the desired state (e.g., replication, endpoint management).  
- **etcd:** A distributed, consistent key-value store that persistently stores all cluster state and configuration data.  
- **Cloud Controller Manager:** Manages cloud-provider-specific integrations such as external load balancers, persistent storage provisioning, and node lifecycle management.  

### **2️⃣ Node Components**  
Nodes are the worker machines that run containerized workloads.  

- **kubelet:** The primary node agent that ensures containers are running as expected.  
- **kube-proxy:** Maintains network rules on each node, enabling seamless service discovery and communication between Pods.  

---

## **Kubernetes Extensions: Beyond the Core**  
While the **Kubernetes Core** provides essential orchestration functionalities, additional features like networking, storage, monitoring, and security are implemented via **external components**.  

### **1️⃣ Plugins**  
Plugins extend Kubernetes by enabling external integration while adhering to standardized APIs.  

- **Container Network Interface (CNI):** Configures Pod networking and IP allocation.  
- **Container Storage Interface (CSI):** Manages external storage solutions.  
- **Container Runtime Interface (CRI):** Allows Kubernetes to interact with various container runtimes (e.g., containerd, CRI-O).  

### **2️⃣ Add-Ons**  
Add-ons enhance Kubernetes with additional functionalities:  

- **Ingress Controllers:** Handle external access, load balancing, and routing.  
- **Monitoring Tools:** Observability tools like Prometheus and Grafana.  
- **Service Meshes:** Solutions like Istio and Linkerd enhance inter-service communication.  

### **3️⃣ Third-Party Extensions**  
Third-party tools and solutions integrate seamlessly into Kubernetes:  

- **Helm:** Kubernetes package manager for simplified deployment.  
- **Prometheus:** Monitoring and alerting system for cloud-native applications.  
- **Istio:** Service mesh for security, traffic management, and observability.  

---

## **Deep Dive into Kubernetes Interfaces**  
Kubernetes uses standardized interfaces for seamless extensibility. The three primary interfaces are **CSI, CNI, and CRI**, each designed to manage storage, networking, and runtime integration.  

### **Overview of Kubernetes Interfaces**  

| **Interface** | **Purpose** | **Key Features** | **Common Implementations** |  
|--------------|------------|------------------|---------------------------|  
| **Container Storage Interface (CSI)** | Enables external storage integration without modifying Kubernetes core. | - Decouples storage drivers from Kubernetes. <br> - Supports snapshots, volume expansion, and lifecycle management. <br> - Uses gRPC-based APIs (`CreateVolume`, `DeleteVolume`, etc.). | - AWS EBS CSI Driver <br> - GCE Persistent Disk CSI Driver <br> - Ceph RBD CSI Driver |  
| **Container Network Interface (CNI)** | Standardizes how Kubernetes manages networking. | - Configures Pod networking and IP allocation. <br> - Supports network policies and encryption. <br> - Enables scalable and secure communication. | - Calico (Network policies) <br> - Cilium (eBPF-based performance) <br> - AWS VPC CNI (AWS-native networking) <br> - Weave Net (Encrypted multi-cluster networking) |  
| **Container Runtime Interface (CRI)** | Defines how Kubernetes interacts with different container runtimes. | - Manages container lifecycle (image pulling, creation, execution, logging). <br> - Ensures runtime consistency. <br> - Enables Kubernetes to work with multiple runtimes. | - containerd (Lightweight, Kubernetes-native) <br> - CRI-O (Minimal runtime for Kubernetes) <br> - Podman (Daemonless alternative) <br> - Docker (Previously supported via dockershim; now deprecated) |  

---

## **Why Kubernetes Uses a Plugin-Based Architecture?**  

Kubernetes’ **plugin-based architecture** is fundamental to its flexibility, scalability, and vendor neutrality. Here’s why:  

- **Interoperability:** Kubernetes interacts with various **networking, storage, and runtime solutions** without tight integration into its core.  
- **Flexibility & Vendor Neutrality:** Users can select the **best tools for their workloads** without being locked into a specific vendor.  
- **Encourages Innovation:** Plugin developers can **iterate and improve** their solutions independently, without requiring changes to Kubernetes itself.  
- **Scalability & Maintainability:** Individual components can be **scaled and upgraded independently**, improving reliability and maintainability.  

For example, **Cilium**, a CNI plugin using **eBPF**, enhances network security and observability without requiring modifications to Kubernetes' core networking model. This level of extensibility allows Kubernetes to adapt seamlessly across different infrastructure environments.  

---

## **Key Takeaways**  

- **Kubernetes Core** consists of essential control plane and node components (API Server, Scheduler, Controller Manager, etc.).  
- **Plugins** (CNI, CSI, CRI) and **Add-ons** (Ingress, Monitoring, Service Mesh) extend Kubernetes functionality beyond the core.  
- **CSI** allows for **scalable and flexible storage integrations** while adhering to strict API compliance.  
- **Modular architecture** ensures vendor neutrality, encourages innovation, and simplifies upgrades.  
- Kubernetes’ **plugin ecosystem** plays a crucial role in its extensibility, allowing it to adapt to diverse infrastructure needs.  

---
## **Conclusion**  

Kubernetes' **modular and extensible architecture** ensures that it remains flexible, scalable, and adaptable to various infrastructure needs. The **core components** (Control Plane and Node Components) handle essential orchestration tasks, while **plugins, add-ons, and third-party extensions** provide additional functionality without bloating the system.  

Key extension mechanisms like **CNI (networking), CSI (storage), and CRI (container runtime)** enable Kubernetes to work seamlessly across diverse cloud-native environments. This **plugin-based design** not only enhances **interoperability and vendor neutrality** but also fosters continuous innovation within the Kubernetes ecosystem.  

Understanding these architectural principles is crucial for **designing, deploying, and managing** Kubernetes clusters efficiently. As we progress, we’ll dive deeper into **Kubernetes Volumes**, focusing on **Persistent Volumes (PVs) and Persistent Volume Claims (PVCs)** to better understand storage management within Kubernetes.  

Now that we understand Kubernetes' **core and extended architecture**, our next lesson will cover **Kubernetes Volumes**, focusing on **Persistent Volumes (PVs)** and **Persistent Volume Claims (PVCs).**  

---

## **References**   
- [Container Network Interface (CNI) Specification](https://github.com/containernetworking/cni)
- [Container Storage Interface (CSI) Specification](https://github.com/container-storage-interface/spec)
- [Container Runtime Interface (CRI) Specification](https://kubernetes.io/docs/concepts/containers/runtime-class/) 
- [Kubernetes Add-Ons and Extensions](https://kubernetes.io/docs/concepts/extend-kubernetes/)

---