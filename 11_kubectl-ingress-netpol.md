# Troubleshooting Kubernetes for Application Developers: Part 11 - Mastering Ingress and NetworkPolicy

## Common Ingress Issues and Solutions

### 1. Endpoint Not Accessible

**Problem:** Your application endpoint returns a 404 or is not accessible even though the Ingress is configured.

**Common Cause 1: Path Matching Issues**
```yaml
# Incorrect Configuration
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /api   # This might not match /api/
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
```

**Solution:**
```yaml
# Correct Configuration
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /api/    # Include trailing slash
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
```

### 2. TLS Certificate Issues

**Problem:** HTTPS endpoints return certificate errors.

**Common Cause: Misconfigured TLS Secret**
```yaml
# Incorrect Configuration
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: secure-ingress
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: tls-secret  # Secret doesn't exist or wrong format
```

**Solution:**
```bash
# Create proper TLS secret
kubectl create secret tls tls-secret \
  --cert=path/to/tls.crt \
  --key=path/to/tls.key

# Verify secret
kubectl get secret tls-secret -o yaml
```

### 3. Cross-Origin Resource Sharing (CORS) Issues

**Problem:** Frontend applications can't make API calls due to CORS errors.

**Solution:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: cors-ingress
  annotations:
    nginx.ingress.kubernetes.io/enable-cors: "true"
    nginx.ingress.kubernetes.io/cors-allow-methods: "GET, POST, PUT, DELETE, OPTIONS"
    nginx.ingress.kubernetes.io/cors-allow-origin: "https://frontend.example.com"
spec:
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
```

### 4. Load Balancer Health Check Failures

**Problem:** Load balancer health checks fail, making the service unavailable.

**Solution:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: healthcheck-ingress
  annotations:
    nginx.ingress.kubernetes.io/healthcheck-path: "/health"
    nginx.ingress.kubernetes.io/healthcheck-interval-seconds: "10"
spec:
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
```

## Network Policy Troubleshooting

### 1. Pod Communication Blocked

**Problem:** Pods can't communicate with each other even within the same namespace.

**Solution:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-same-namespace
spec:
  podSelector: {}  # Applies to all pods
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector: {}  # Allow from all pods in same namespace
```

### 2. Database Access Issues

**Problem:** Application pods can't access the database pod.

**Solution:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-db-access
spec:
  podSelector:
    matchLabels:
      app: database
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: backend
    ports:
    - protocol: TCP
      port: 5432
```

## Troubleshooting Commands and Tools

### Ingress Troubleshooting
```bash
# Check Ingress status
kubectl describe ingress myapp-ingress

# Check Ingress controller logs
kubectl logs -n ingress-nginx deploy/ingress-nginx-controller

# Test endpoint resolution
kubectl run curl --image=curlimages/curl -it --rm -- \
  curl -v http://myapp.example.com

# Verify TLS certificate
openssl s_client -connect myapp.example.com:443 -servername myapp.example.com
```

### NetworkPolicy Troubleshooting
```bash
# Test pod connectivity
kubectl run test-pod --image=busybox -it --rm -- wget -qO- http://service-name

# Check NetworkPolicy rules
kubectl describe networkpolicy allow-db-access

# Verify pod labels
kubectl get pods --show-labels
```

## Best Practices

### Ingress Best Practices
1. Always use path type "Prefix" or "Exact" explicitly
2. Include proper annotations for your ingress controller
3. Implement proper health checks
4. Use TLS for production workloads
5. Test with curl from within the cluster

### NetworkPolicy Best Practices
1. Start with deny-all policy and add specific allow rules
2. Label pods appropriately for network policy targeting
3. Test connectivity after applying policies
4. Document policy intentions in annotations
5. Use namespaces for isolation

## Debug Checklist

### Ingress Issues
- [ ] Verify Ingress controller is running
- [ ] Check path matching configuration
- [ ] Validate TLS certificate and secret
- [ ] Confirm service endpoint is accessible
- [ ] Check CORS configuration if needed
- [ ] Verify DNS resolution
- [ ] Check load balancer health checks

### NetworkPolicy Issues
- [ ] Confirm pod labels match policy selectors
- [ ] Verify namespace matches
- [ ] Check port specifications
- [ ] Test with temporary allow-all policy
- [ ] Verify CNI plugin supports NetworkPolicy
- [ ] Check for conflicting policies

## Sample Debugging Session
```bash
# 1. Check Ingress status
kubectl get ingress
kubectl describe ingress myapp-ingress

# 2. Verify backend service
kubectl get svc backend-service
kubectl describe svc backend-service

# 3. Check endpoints
kubectl get endpoints backend-service

# 4. Test from within cluster
kubectl run test-pod --image=curlimages/curl -it --rm -- \
  curl -v http://backend-service

# 5. Check Ingress controller logs
kubectl logs -n ingress-nginx \
  -l app.kubernetes.io/component=controller
```
---
[Next Article: Deep Dive into kubectl RBAC & auth →](./12_kubectl-rback-auth.md)
