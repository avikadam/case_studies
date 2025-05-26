# Kubernetes Architecture Explained 🚢

Kubernetes is a container orchestration platform that automates deployment, scaling, and management of containerized applications. It follows a **master-worker** (control plane - node) model.

---

## 📌 Kubernetes Architecture Overview
```

+---------------------+        +---------------------+
|                     |        |                     |
|  Control Plane      |<------>|   Nodes (Workers)   |
| (Master Components) |        |  (Where apps run)   |
|                     |        |                     |
+---------------------+        +---------------------+
```
![k8s_architecture.png](/k8s/k8s_architecture.png)

---

## 🧠 Core Components

### 🔹 Control Plane (Master)

| Component             | Description |
|----------------------|-------------|
| `kube-apiserver`     | Entry point for all API operations |
| `etcd`               | Distributed key-value store (cluster state) |
| `kube-scheduler`     | Assigns pods to nodes |
| `kube-controller-manager` | Reconciles cluster state |
| `cloud-controller-manager` | Manages cloud-specific logic (optional) |

### 🔹 Node Components (Workers)

| Component         | Description |
|------------------|-------------|
| `kubelet`        | Manages lifecycle of pods on a node |
| `kube-proxy`     | Manages service networking |
| Container Runtime| Runs containers (Docker, containerd, CRI-O) |

---

## 🏢 Real-World Analogy: **Airport Terminal**

| Kubernetes Component     | Real-World Equivalent                     |
|--------------------------|-------------------------------------------|
| `etcd`                   | **Airline reservation system** – stores all bookings (cluster state) |
| `kube-apiserver`         | **Airport front desk** – receives all service requests |
| `kube-scheduler`         | **Air traffic controller** – assigns planes (pods) to gates (nodes) |
| `controller-manager`     | **Airport operations manager** – ensures correct number of staff, gates, etc. |
| `kubelet`                | **Gate agent** – ensures plane is ready at the gate |
| `kube-proxy`             | **Runway network controller** – ensures traffic routing to correct runways |
| **Pods (apps)**          | **Airplanes** – actual payload or service |

---

## 🧪 Hands-On YAML Example: Deploying a Simple NGINX App

Create a file named `nginx-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
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
          image: nginx:1.25
          ports:
            - containerPort: 80
```
Deploy with:
```cmd
kubectl apply -f nginx-deployment.yaml
```

Then expose it using a service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30007
```

Apply it:

```bash
kubectl apply -f nginx-service.yaml
```
Access via ```http://<NodeIP>:30007```

## 📚 Useful Commands
```
kubectl get nodes
kubectl get pods -o wide
kubectl get services
kubectl describe pod <pod-name>
kubectl logs <pod-name>
```
---

## 📦 Kubernetes Resource Kinds

In Kubernetes, the `kind` field in a YAML manifest defines the **type of object** you're creating or managing. Each resource type has a specific structure and purpose.

---

### 🧱 Core Workload Resources

| Kind         | Description |
|--------------|-------------|
| `Pod`        | The smallest deployable unit in Kubernetes. Contains one or more containers. |
| `Deployment` | Manages stateless apps and handles rolling updates, scaling, and replicas. |
| `StatefulSet`| Manages stateful apps with persistent identities and storage. |
| `DaemonSet`  | Ensures a pod runs on every (or some) node, useful for log collectors or agents. |
| `Job`        | Runs one-off tasks to completion. |
| `CronJob`    | Schedules jobs periodically, like a Linux cron. |

---

### 🚪 Networking & Service Discovery

| Kind            | Description |
|-----------------|-------------|
| `Service`       | Exposes pods as a network service. Types: ClusterIP, NodePort, LoadBalancer. |
| `Ingress`       | Manages external access (HTTP/S) to services in the cluster. |
| `NetworkPolicy` | Controls the communication between pods. |

---

### 🔐 Configuration & Secrets

| Kind                | Description |
|---------------------|-------------|
| `ConfigMap`         | Stores configuration data in key-value pairs. |
| `Secret`            | Stores sensitive data like passwords and tokens (base64-encoded). |
| `PersistentVolume` (PV)   | Represents a piece of storage in the cluster. |
| `PersistentVolumeClaim` (PVC) | Requests storage resources. |
| `VolumeAttachment`  | Binds a persistent volume to a node. |

---

### 🛠️ Controllers & Scaling

| Kind                      | Description |
|---------------------------|-------------|
| `ReplicaSet`              | Ensures a specified number of pod replicas are running. Usually managed by Deployments. |
| `HorizontalPodAutoscaler` | Scales pods automatically based on CPU or custom metrics. |
| `PodDisruptionBudget`     | Ensures availability during voluntary disruptions. |

---

### 📦 Namespace & Policy

| Kind              | Description |
|-------------------|-------------|
| `Namespace`       | Virtual cluster within a Kubernetes cluster. Used for organizing resources. |
| `ResourceQuota`   | Limits the total amount of resources that can be used in a namespace. |
| `LimitRange`      | Sets default CPU/memory limits for pods/containers in a namespace. |
| `ServiceAccount`  | Provides an identity for processes running in a pod to interact with the cluster. |

---

### 🔐 RBAC (Role-Based Access Control)

| Kind                  | Description |
|-----------------------|-------------|
| `Role`                | Specifies permissions within a namespace. |
| `ClusterRole`         | Specifies cluster-wide permissions. |
| `RoleBinding`         | Grants a Role to a user/service account in a namespace. |
| `ClusterRoleBinding`  | Grants a ClusterRole across the cluster. |

---

### 🧪 Extensibility with Custom Resources

| Kind                        | Description |
|-----------------------------|-------------|
| `CustomResourceDefinition` (CRD) | Enables you to define your own API resource types. |
| Custom Kinds                | Once a CRD is installed, you can create resources like `KafkaCluster`, `ElasticSearchCluster`, etc. |

---

### 📘 Example: `Deployment` Manifest

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app-container
          image: my-app-image:v1
```
---

## 🚪 Kubernetes Service Types Explained

In Kubernetes, a **Service** is an abstraction that defines **a logical set of Pods** and a policy to access them. Since Pods are ephemeral, Services provide a **stable endpoint** to communicate with your app.

Kubernetes supports **4 main types of Services**:

---

### 🔹 1. ClusterIP (Default)

**Use Case:** Internal-only communication within the cluster.

- Pods can be reached only from **within** the cluster.
- Default service type if none is specified.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-clusterip-service
spec:
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP
```

#### 💡 Analogy:
<i>Like an internal office extension – only reachable from within the same office network.</i>

---
### 🔹 2. NodePort
**Use Case:** Expose service on a static port on each Node’s IP.

- Accessible from outside the cluster via ``<NodeIP>:<NodePort>.``
- NodePort range is usually ``30000–32767.``
  
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nodeport-service
spec:
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080
  type: NodePort
```

#### 💡 Analogy:
<i>Like an office doorbell wired to ring on a specific port – anyone with the address and port can reach it.</i>

---

### 🔹 3. LoadBalancer
**Use Case:** Expose services to the internet using a cloud provider’s load balancer.

- Creates an external load balancer in supported environments (AWS, Azure, GCP, etc.).
- Automatically assigns a public IP address.
  
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-loadbalancer-service
spec:
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
  type: LoadBalancer
```
#### 💡 Analogy:
<i>Like placing your business on Google Maps – open to the public via a real IP.

---

### 🔹 4. ExternalName
**Use Case:** Redirect service traffic to an external DNS name outside the cluster.

- Maps the Service name to a DNS name (e.g., external.database.example.com).
- Does not create typical service endpoints.
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-external-service
spec:
  type: ExternalName
  externalName: api.external-service.com
```
### 💡 Analogy:
<i> Like a phone contact that forwards to an external number.</i>

---

### 📊 Comparison Table
| Type           | Internal Access | External Access | Public IP | Port Control  | Use Case                             |
| -------------- | --------------- | --------------- | --------- | ------------- | ------------------------------------ |
| `ClusterIP`    | ✅               | ❌               | ❌         | Internal only | Default for in-cluster communication |
| `NodePort`     | ✅               | ✅               | Node IP   | Manual        | Dev/test or simple ingress           |
| `LoadBalancer` | ✅               | ✅               | ✅         | Auto          | Production apps in cloud             |
| `ExternalName` | ❌ (redirects)   | ✅ (via DNS)     | N/A       | N/A           | Redirect to external service         |

---

## 📶 Ingress – HTTP Routing Layer
**Ingress** allows HTTP and HTTPS traffic to reach Services from outside the cluster using a single external **IP and domain-based routing**.

### 🔸 Why Use Ingress?
- Consolidate multiple services behind one entry point.
- Enables TLS termination (HTTPS).
- Path-based or subdomain-based routing.

### 📄 Sample Ingress YAML
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 80
          - path: /web
            pathType: Prefix
            backend:
              service:
                name: web-service
                port:
                  number: 80
```

### 💡 Analogy:
_Ingress is like a reception desk that decides which internal team gets the visitor based on the URL or domain._

---

## 🔁 Service Mesh – Advanced Service-to-Service Communication
A **Service Mesh** is a dedicated infrastructure layer to manage **service-to-service communication**.

🔥 Features:
- Traffic control (retries, timeouts, mirroring, canary releases)
- Security (mTLS encryption between services)
- Observability (metrics, tracing, logging)
- Policy enforcement

### 🚀 Popular Service Meshes
| Mesh    | Features                                 | Controllers         |
| ------- | ---------------------------------------- | ------------------- |
| Istio   | mTLS, traffic routing, policy, telemetry | Pilot, Mixer, Envoy |
| Linkerd | Lightweight, mTLS, retries, metrics      | Proxy (Rust)        |
| Consul  | Service discovery, mesh, key-value store | Consul agent        |

### 📄 Example: Istio VirtualService (simplified)
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: my-mesh-route
spec:
  hosts:
    - my-service
  http:
    - match:
        - uri:
            prefix: /v1
      route:
        - destination:
            host: my-service
            subset: v1
    - match:
        - uri:
            prefix: /v2
      route:
        - destination:
            host: my-service
            subset: v2
```
> [!NOTE]
> Requires DestinationRule, Envoy sidecars, and Istio control plane.

### 💡 Analogy:
_A Service Mesh is like an internal air traffic control system for your services — ensuring safe, efficient, and secure routing._

### 🔚 Summary

| Component      | Purpose                           | Use Case                          |
| -------------- | --------------------------------- | --------------------------------- |
| `Service`      | Stable networking to Pods         | Internal/external access          |
| `Ingress`      | HTTP gateway for external traffic | Domain/path routing               |
| `Service Mesh` | Advanced control between services | Security, observability, policies |

> [!TIP]
> Combine Ingress + Service + Service Mesh for robust production-grade Kubernetes networking!

---

## ✅ Summary
- Kubernetes is designed to automate container orchestration.
- It splits responsibilities into control plane and worker nodes.
- Uses declarative YAML configuration for defining applications.
- The real-world analogy (like an airport) helps map components to familiar operations.
