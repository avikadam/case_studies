**Service discovery** is a key concept in microservices and distributed systems, referring to the process by which services automatically detect and communicate with each other without needing hardcoded addresses.

---

# 🔍 Why is Service Discovery Needed?
In dynamic environments like Kubernetes or cloud-based microservice systems:
- Services are scaled up/down frequently.
- IP addresses of instances change.
- New instances come and go based on load.
Without service discovery, you'd have to hardcode service addresses, which is error-prone and not scalable.

---

# 📚 Types of Service Discovery
1. Client-Side Discovery
- The client is responsible for querying the service registry to get the address of available service instances.
- Examples: Netflix Eureka + Ribbon (in Spring Cloud), Consul
- Flow:\
  ```Client → Service Registry → Get instance info → Request to service```
2. Server-Side Discovery
- The client makes a generic request, and a load balancer or gateway queries the registry and routes it.
- Examples: Kubernetes, AWS Elastic Load Balancer
- Flow:\
  ```Client → Load Balancer → Service Registry → Route to instance```
---

# 🧩 Popular Tools for Service Discovery
| Tool               | Type        | Notes                                               |
| ------------------ | ----------- | --------------------------------------------------- |
| **Eureka**         | Client-side | Part of Netflix OSS; well-supported in Spring Cloud |
| **Consul**         | Both        | Key-value store + service discovery + health checks |
| **Zookeeper**      | Server-side | Used by Apache projects like Kafka                  |
| **Kubernetes DNS** | Server-side | Services are discovered via DNS names automatically |
| **Nacos**          | Both        | Alibaba’s open-source registry, supports config too |

# 🛠️ With Kubernetes Example
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
```
Then you can access it from other pods via:
```http://my-app.default.svc.cluster.local```
Kubernetes handles the DNS resolution and service routing.

---

# ✅ Best Practices
- Use health checks to keep service registry entries accurate.
- Combine with load balancing and circuit breakers (e.g., Resilience4j).
- Monitor and log discovery performance for resilience.

---

# 🚀 1. Kubernetes Service Discovery
Kubernetes provides basic service discovery mechanisms built into the platform.

## ✅ How it works:
- When a Service is created (e.g., ClusterIP, NodePort, LoadBalancer), Kubernetes assigns a DNS name (e.g., my-service.my-namespace.svc.cluster.local) and a virtual IP.
- Pods within the cluster can resolve the service via DNS and connect to it.
- Under the hood, kube-proxy handles the routing by rewriting IP tables or using IPVS.

## 🧩 Features:
- DNS-based service discovery.
- Load balancing across pods behind a service.
- Health checks (via Kubernetes Probes).
- Automatic updates when Pods change.

## 🚫 Limitations:
- No built-in support for:
  - Fine-grained traffic control (e.g., canary, A/B).
  - Retries, timeouts, circuit breaking.
  - Secure service-to-service communication (mTLS).
  - Detailed observability/tracing.

## How to overcome each limitations [without service mesh]
### 🚫 Retries, timeouts, circuit breaking 
#### At the Application Layer (Spring Boot + Resilience4j)
- Spring Cloud + Resilience4j supports:
  - Retries
  - Circuit Breaker
  - Timeouts
  - Rate Limiting
##### ⚙️ Example (application.yml):
```yaml
resilience4j:
  retry:
    instances:
      myService:
        max-attempts: 3
        wait-duration: 2s
  circuitbreaker:
    instances:
      myService:
        slidingWindowSize: 10
        failureRateThreshold: 50
        waitDurationInOpenState: 30s
  timelimiter:
    instances:
      myService:
        timeoutDuration: 3s
```
##### ⚙️ Code:
```yaml
@Retry(name = "myService")
@CircuitBreaker(name = "myService")
@TimeLimiter(name = "myService")
public CompletableFuture<String> callRemoteService() {
    return CompletableFuture.supplyAsync(() -> restTemplate.getForObject(url, String.class));
}
```
#### ✅ At API Gateway Level 
You can define retry and timeout policies at the API gateway layer for ingress traffic.

##### 🛡️ AWS API Gateway Resilience Features

| Feature       | Supports? | Where Configured                     |
| ------------- | --------- | ------------------------------------ |
| Retries       | ✅         | Integration-level (for AWS services) |
| Timeout       | ✅         | Integration-level                    |
| Rate Limiting | ✅         | Usage plans + API keys               |

##### 🔁 1. Retries in API Gateway
API Gateway automatically retries integration requests to AWS services (e.g., Lambda, DynamoDB, etc.) up to 2 times by default on 5xx errors or network timeouts.

>❗ You cannot configure retries directly for HTTP integrations or custom services—only for AWS service integrations.

##### ⏱️ 2. Timeouts
- Maximum timeout for API Gateway (HTTP or Lambda): 29 seconds
- You can configure it in the integration settings.

##### 🔧 How to Configure (Timeout):
In the API Gateway console:
  1. Go to your API → Resource → Method (e.g., ``GET`` or ``POST``)
  2. Click on Integration Request
  3. Under HTTP Timeout (ms), set a value (e.g., ``10000`` for ``10`` seconds)

> Applies to HTTP/HTTP_PROXY and Lambda proxy integrations.

##### 🚦 3. Rate Limiting
Use usage plans + API keys to enforce throttling:

##### 🔧 How to Configure:
1. Go to API Gateway → Usage Plans
2. Create a plan:
  - Rate limit (requests per second, e.g., 100)
  - Burst limit (max requests in a burst, e.g., 200)
3. Associate the plan with an API stage
4. Create or attach an API key to consumers

**Example:**
```
Rate limit: 50 RPS
Burst limit: 100 requests
```

So:
- Normal flow is 50 requests/sec
- Spikes up to 100 requests allowed briefly

> 🔒 Only applies if API key is required and passed in the request.

##### ✅ Optional: Use AWS WAF (Web Application Firewall)
Attach AWS WAF to API Gateway for:
- Advanced rate limiting
- Geo blocking
- IP blacklisting
- Bot detection

**Example WAF Rule (Rate-based):**
  ```
  Limit: 1000 requests per 5 minutes per IP
  ```

### 🚫 Fine-grained traffic control (e.g., canary, A/B)
#### 🛠️ Tools and Strategies 
| Feature                | Tool/Strategy                                            | Notes               |
| ---------------------- | -------------------------------------------------------- | ------------------- |
| **Canary Deployments** | Argo Rollouts, Kubernetes-native (manual), NGINX Ingress | Gradual rollout     |
| **A/B Testing**        | NGINX Ingress, Traefik, custom headers + routing         | Header/cookie based |
| **Blue/Green**         | Kubernetes + Ingress Controller or manual label switch   | Full version switch |
| **Traffic Splitting**  | Ingress-based weighted routing (e.g., NGINX, Gloo Edge)  | Based on weights    |

### 🚫 Secure service-to-service communication (mTLS)
#### ✅ Ways to Enable mTLS in EKS

| Method                                                  | Description                                                       |
| ------------------------------------------------------- | ----------------------------------------------------------------- |
| 1. 🔧 **Custom Sidecar Proxies** (e.g., Envoy)          | Manually deploy Envoy as sidecars to handle mTLS                  |
| 2. 🔐 **Application-Level TLS**                         | Modify your services to implement TLS/mTLS directly               |
| 3. 🔁 **Use Kubernetes Ingress + NGINX + Cert-Manager** | Secure ingress traffic and possibly internal traffic              |
| 4. 🔐 **Use SPIRE (SPIFFE Runtime Environment)**        | Secure identity + mTLS without a full mesh                        |
| 5. 🔑 **AWS Private CA + Mutual TLS**                   | Use AWS ACM Private CA to issue certs and manually configure mTLS |

### 🚫 Detailed observability/tracing
#### ✅ Key Components for Observability in EKS

| Layer               | Tools to Use                            | Notes                |
| ------------------- | --------------------------------------- | -------------------- |
| **Metrics**         | Prometheus + Grafana, AWS CloudWatch    | System & app metrics |
| **Tracing**         | AWS X-Ray, OpenTelemetry, Jaeger, Tempo | Distributed traces   |
| **Logging**         | Fluent Bit, CloudWatch Logs, OpenSearch | App + system logs    |
| **Instrumentation** | OpenTelemetry SDKs, X-Ray SDKs          | Add to app code      |

#### 🔭 1. Distributed Tracing (OpenTelemetry)
##### 🔧 OpenTelemetry (Recommended Standard)
- Supports Jaeger, AWS X-Ray, Tempo, and Zipkin backends.
- You deploy the OpenTelemetry Collector as a sidecar or daemonset.
- App sends spans → collector → trace backend.

**Steps:**
- Instrument your app using OpenTelemetry SDK.
- Deploy OpenTelemetry Collector to forward traces.
- Visualize in Grafana Tempo, Jaeger, or AWS X-Ray.

**Example:**
```java
OpenTelemetry openTelemetry = OpenTelemetrySdk.builder()
    .setTracerProvider(
        SdkTracerProvider.builder()
            .addSpanProcessor(BatchSpanProcessor.builder(OtlpGrpcSpanExporter.builder().build()).build())
            .build())
    .build();
```

**Collector Config Example (for OTLP to X-Ray):**

```yaml
Copy
Edit
receivers:
  otlp:
    protocols:
      grpc:

exporters:
  awsxray:
    region: us-west-2

service:
  pipelines:
    traces:
      receivers: [otlp]
      exporters: [awsxray]
```
#### 📊 Metrics: Prometheus + Grafana or CloudWatch
- **Prometheus**: Most common for scraping metrics in K8s.
- **CloudWatch Container Insights**: AWS-native metrics without much setup.

##### Option A: Prometheus + Grafana
- Install Prometheus (Helm or Operator)
- Use Prometheus node exporters and Kubernetes scrape configs
- Visualize with Grafana

##### Option B: AWS CloudWatch Agent
```bash
eksctl enable logging --region <region> --cluster <cluster_name> --components metrics
```
#### 🧠 4. Correlating Logs, Metrics, and Traces
If using OpenTelemetry:
- Add trace IDs to log lines
- Use context propagation (e.g., W3C Trace Context) in your application
- Link traces to logs via trace_id field

---
# 🌐 2. Service Mesh Service Discovery
A Service Mesh (like Istio, Linkerd, Consul Connect) enhances or replaces Kubernetes service discovery with a more powerful and flexible mechanism.

## ✅ How it works:
- Each Pod runs a sidecar proxy (e.g., Envoy for Istio).
- The mesh maintains a centralized control plane that knows about all services and endpoints.
- Communication goes through the sidecar proxies, which apply policies, security, and routing rules.

## 🧩 Features:
- Enhanced service discovery: based on metadata, labels, and policies.
- Traffic control: weighted routing, fault injection, mirroring.
- mTLS encryption for all service-to-service traffic.
- Observability: distributed tracing, metrics, and logging.
- Resilience: retries, timeouts, circuit breakers.

## 🔧 Discovery in Service Mesh:
- Often combines Kubernetes DNS + service registry with mesh’s internal control plane.
- Uses dynamic service registry (not just DNS) — more efficient for large clusters.

# 🆚 Summary Comparison

| Feature                     | Kubernetes Service Discovery | Service Mesh Discovery            |
| --------------------------- | ---------------------------- | --------------------------------- |
| Mechanism                   | DNS + kube-proxy             | Sidecar proxies + Control Plane   |
| Dynamic Endpoint Updates    | Yes                          | Yes                               |
| DNS-based Resolution        | Yes                          | Optional (can use direct IPs)     |
| Traffic Management          | No                           | Yes (routing rules, etc.)         |
| Resilience (retry, timeout) | No                           | Yes                               |
| Observability               | Basic (via Prometheus, etc.) | Advanced (tracing, logs, metrics) |
| Security (mTLS)             | No                           | Yes                               |
| External Service Support    | Limited                      | Strong (via egress gateways etc.) |

# 🧠 When to Use What?
- **Kubernetes service discovery** is sufficient for basic microservices in a trusted, internal environment with minimal requirements for observability or control.
- **Service mesh** is better suited for complex, production-grade systems that require security, observability, resilience, and advanced traffic control.
