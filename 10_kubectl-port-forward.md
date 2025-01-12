# Troubleshooting Kubernetes for Application Developers: Part 10 - Mastering kubectl port-forward, Service Types & ServiceAccounts

## Understanding kubectl port-forward
The `kubectl port-forward` command creates a secure tunnel between your local machine and a pod or service in your Kubernetes cluster. This is invaluable for debugging and testing applications without exposing them publicly.

## Understanding Service Accounts in Kubernetes
Service accounts are a vital component in Kubernetes, enabling pods to interact securely with the API server. By default, Kubernetes assigns a default service account to every pod, granting limited permissions. For tasks requiring specific access levels, creating custom service accounts and binding them to roles via Role-Based Access Control (RBAC) ensures secure and precise access control.

When using commands like `kubectl port-forward`, ensure the service account associated with the pod has the necessary permissions to access the required resources securely. Misconfigured service accounts can inadvertently expose sensitive cluster information, so adhere to the principle of least privilege when assigning roles.

## Service Types in Kubernetes

Kubernetes offers several service types to manage and expose applications efficiently. Each type serves a distinct purpose based on how the service is accessed. Below is a comprehensive list of the available service types:

---

#### 1. **ClusterIP (Default)**
   - Exposes the service on an internal IP address within the cluster.
   - Accessible **only** from within the cluster.
   - Ideal for internal communication between pods or services.
   - Example Use Case: Backend microservices communicating with each other.

---

#### 2. **NodePort**
   - Exposes the service on a specific port of each node's IP address.
   - Accessible from outside the cluster using `<NodeIP>:<NodePort>`.
   - Suitable for exposing services during development or testing.
   - Example Use Case: Debugging a pod by exposing it directly to external traffic.

---

#### 3. **LoadBalancer**
   - Provisions an external load balancer (if supported by the cloud provider).
   - Routes external traffic to the service and distributes it across pods.
   - Best suited for production applications requiring high availability.
   - Example Use Case: A public-facing web application hosted on AWS or GCP.

---

#### 5. **Headless Service (Special Case)**
   - A variation of the `ClusterIP` service where no cluster IP is assigned (`ClusterIP: None`).
   - Directly exposes the DNS of the pods for better control over traffic routing.
   - Example Use Case: StatefulSets or databases like Cassandra or MongoDB.

---

### Summary
Each service type is designed for specific use cases, whether you're exposing services internally, to specific external endpoints, or globally.

### Basic Port-Forward Syntax
```bash
kubectl port-forward [TYPE/NAME] [LOCAL_PORT:]REMOTE_PORT
```

### Common Port-Forward Use Cases

1. **Forward to a Pod**
```bash
# Forward local port 8080 to pod port 80
kubectl port-forward pod/my-pod 8080:80

# Forward multiple ports simultaneously
kubectl port-forward pod/my-pod 8080:80 8443:443
```

2. **Forward to a Service**
```bash
# Forward local port 8000 to service port 80
kubectl port-forward -n myapp svc/frontend 8000:80
```

3. **Forward to a Deployment**
```bash
# Forward to the first pod of a deployment
kubectl port-forward deployment/myapp 8080:8080
```

## Understanding Kubernetes Service Types

### 1. ClusterIP Service
ClusterIP is the default service type, providing internal-only access within the cluster.

**Example ClusterIP Service:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 8080
```

**Testing ClusterIP Services:**
```bash
# Create a temporary pod to test internal access
kubectl run test-pod --rm -it --image=curlimages/curl -- sh
curl backend-service.default.svc.cluster.local
```

### 2. NodePort Service
NodePort exposes the service on each node's IP at a static port.

**Example NodePort Service:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  type: NodePort
  selector:
    app: frontend
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30080  # Port range 30000-32767
```

**Accessing NodePort Services:**
```bash
# Get node IP
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[0].address}')

# Access service
curl $NODE_IP:30080
```

### 3. LoadBalancer Service
LoadBalancer provisions an external load balancer in cloud providers.

**Example LoadBalancer Service:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 8080
```

**Checking LoadBalancer Status:**
```bash
# Get external IP
kubectl get svc web-service -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
```

### 4. ExternalName Service
ExternalName maps a service to a DNS name, useful for external service integration.

**Example ExternalName Service:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: database.example.com
```

## Understanding ServiceAccounts and Common Issues

### What is a ServiceAccount?
A ServiceAccount provides an identity for processes running in a Pod, allowing them to interact with the Kubernetes API and other services.

### Default ServiceAccount Behavior
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  serviceAccountName: default  # Automatically mounted if not specified
  containers:
  - name: myapp
    image: myapp:1.0
```

### Common ServiceAccount Issues and Solutions

1. **Permission Denied Errors**
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-service-account
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
subjects:
- kind: ServiceAccount
  name: app-service-account
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

2. **Token Mount Issues**
```bash
# Check if token is mounted
kubectl exec myapp-pod -- ls /var/run/secrets/kubernetes.io/serviceaccount

# Verify token content
kubectl exec myapp-pod -- cat /var/run/secrets/kubernetes.io/serviceaccount/token
```

3. **Debugging ServiceAccount Configuration**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: debug-pod
spec:
  serviceAccountName: app-service-account
  containers:
  - name: debug
    image: curlimages/curl
    command: ['sleep', '3600']
    volumeMounts:
    - name: token
      mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      readOnly: true
```

### ServiceAccount Troubleshooting Commands
```bash
# Check ServiceAccount details
kubectl describe serviceaccount app-service-account

# List ServiceAccount secrets
kubectl get serviceaccount app-service-account -o yaml

# Test API access with ServiceAccount token
TOKEN=$(kubectl exec myapp-pod -- cat /var/run/secrets/kubernetes.io/serviceaccount/token)
curl -k -H "Authorization: Bearer $TOKEN" https://kubernetes.default.svc/api/v1/namespaces/default/pods
```

### Best Practices for ServiceAccounts

1. **Explicit ServiceAccount Assignment**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  serviceAccountName: custom-sa  # Always specify explicitly
```

2. **Minimal Permissions**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: minimal-access
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get"]  # Only grant necessary permissions
```

3. **Token Automounting Control**
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: no-auto-mount
automountServiceAccountToken: false
```

## Conclusion
Understanding service types and ServiceAccount configuration is crucial for secure and effective Kubernetes deployments. Common issues often stem from:
- Incorrect RBAC permissions
- Missing or incorrectly mounted service account tokens
- Mismatched ServiceAccount names
- Insufficient access rights

Always validate ServiceAccount configurations and permissions when troubleshooting application connectivity issues within your cluster.

---
[Next Article: Deep Dive into kubectl ingress & networkpolicy →](./11_kubectl-ingress-netpol.md)
---

