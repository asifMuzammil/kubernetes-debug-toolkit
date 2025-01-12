# Troubleshooting Kubernetes for Application Developers: Part 8/2 - Mastering Most Common Issues

## 1. Resource Quotas and Limits Deep Dive

### Understanding LimitRange
LimitRange provides constraints on resource allocations (to Pods or Containers) in a namespace.

**Common LimitRange Issues:**
1. Pod Creation Failures
2. Container OOMKilled
3. CPU Throttling

**Example LimitRange Configuration:**
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
  namespace: myapp
spec:
  limits:
  - type: Container
    default:            # Default limits if not specified
      memory: "512Mi"
      cpu: "500m"
    defaultRequest:     # Default requests if not specified
      memory: "256Mi"
      cpu: "250m"
    max:               # Maximum allowed
      memory: "1Gi"
      cpu: "1"
    min:               # Minimum allowed
      memory: "128Mi"
      cpu: "100m"
```

**Common Problems and Solutions:**

1. **OOMKilled Issues:**
```yaml
# Problem: Container being killed due to memory limits
apiVersion: apps/v1
kind: Deployment
metadata:
  name: memory-issue
spec:
  template:
    spec:
      containers:
      - name: app
        resources:
          limits:
            memory: "256Mi"  # Too low for application
          requests:
            memory: "128Mi"

# Solution: Adjust limits based on actual usage
        resources:
          limits:
            memory: "512Mi"  # Increased based on monitoring
          requests:
            memory: "256Mi"
```

2. **CPU Throttling:**
```yaml
# Check CPU throttling
kubectl top pod memory-issue
kubectl describe pod memory-issue | grep -A 5 "CPU Throttling"

# Solution: Adjust CPU limits
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cpu-optimal
spec:
  template:
    spec:
      containers:
      - name: app
        resources:
          limits:
            cpu: "1"    # Increased based on actual usage
          requests:
            cpu: "500m"
```

## 2. Field Immutability in Kubernetes

### What is Immutability?
Immutability in Kubernetes means certain fields cannot be modified after creation. This ensures system stability and predictable behavior.

### Why Use Immutability?
1. **Consistency:** Prevents unexpected changes
2. **Security:** Protects critical configurations
3. **Reliability:** Ensures stable cluster state
4. **Audit:** Makes changes trackable

### Common Immutable Fields and Solutions:

1. **Pod Specifications**
```yaml
# These fields are immutable in Pod spec:
spec:
  nodeName            # Cannot change after pod creation
  serviceAccountName  # Cannot change after pod creation
  containers:
    - name           # Container name cannot be changed
      image         # In pods (but can be updated in deployments)
```

2. **PersistentVolume Source**
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  # Cannot change these after creation
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data
```

3. **Service Fields**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  # These cannot be changed after creation:
  clusterIP: 10.0.0.10
  selector:
    app: myapp
```

### Handling Immutable Fields:

1. **For Deployments:**
```bash
# Create new deployment with different name
kubectl create deployment new-app --image=nginx

# Copy config from old to new
kubectl get deployment old-app -o yaml > new-app.yaml
# Edit new-app.yaml
kubectl apply -f new-app.yaml

# Switch traffic gradually
```

2. **For Services:**
```bash
# Backup old service
kubectl get svc my-service -o yaml > service-backup.yaml

# Delete and recreate
kubectl delete svc my-service
kubectl create -f new-service.yaml
```

## 3. Automated ConfigMap and Secret Updates

# Understanding the ConfigMap Checksum Pattern in Kubernetes

## What is the Checksum Pattern?

The checksum pattern is a technique to force Kubernetes pods to restart when their ConfigMaps or Secrets change. Since Kubernetes doesn't automatically restart pods when ConfigMaps change, we use this pattern to trigger a rolling update.

## How It Works

1. Calculate a SHA256 hash of the ConfigMap content
2. Add this hash as an annotation to the pod template
3. When ConfigMap changes, the hash changes
4. Changed hash triggers a rolling update of pods

## Implementation Examples

### 1. Basic Implementation

```bash
#!/bin/bash
# Script to update deployment when ConfigMap changes

# Variables
NAMESPACE="myapp"
CONFIGMAP_NAME="app-config"
DEPLOYMENT_NAME="myapp"

# Get ConfigMap data and calculate checksum
CHECKSUM=$(kubectl get configmap $CONFIGMAP_NAME -n $NAMESPACE -o yaml | sha256sum | awk '{print $1}')

# Update deployment with new checksum
kubectl patch deployment $DEPLOYMENT_NAME -n $NAMESPACE -p \
  "{\"spec\":{\"template\":{\"metadata\":{\"annotations\":{\"checksum/config\":\"$CHECKSUM\"}}}}}"
```

### 2. Automated Deployment Configuration

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    metadata:
      annotations:
        # This will be automatically updated when ConfigMap changes
        checksum/config: "abc123def456"  # Initial value
    spec:
      containers:
      - name: myapp
        image: myapp:1.0
        volumeMounts:
        - name: config-volume
          mountPath: /config
      volumes:
      - name: config-volume
        configMap:
          name: app-config
```

### 3. Complete Example with ConfigMap and Update Script

```yaml
# ConfigMap definition
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  config.properties: |
    app.name=MyApp
    app.port=8080
    app.environment=production
---
# Deployment using ConfigMap
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    metadata:
      annotations:
        checksum/config: "${SHA256_OF_CONFIGMAP}"
    spec:
      containers:
      - name: myapp
        image: myapp:1.0
        volumeMounts:
        - name: config-volume
          mountPath: /config
      volumes:
      - name: config-volume
        configMap:
          name: app-config
```

```bash
#!/bin/bash
# comprehensive-update.sh

# Set variables
NAMESPACE="myapp"
CONFIGMAP_NAME="app-config"
DEPLOYMENT_NAME="myapp"

# Function to calculate ConfigMap checksum
calculate_checksum() {
    kubectl get configmap $CONFIGMAP_NAME -n $NAMESPACE -o yaml | sha256sum | awk '{print $1}'
}

# Function to get current checksum from deployment
get_current_checksum() {
    kubectl get deployment $DEPLOYMENT_NAME -n $NAMESPACE -o jsonpath='{.spec.template.metadata.annotations.checksum/config}'
}

# Calculate new checksum
NEW_CHECKSUM=$(calculate_checksum)
CURRENT_CHECKSUM=$(get_current_checksum)

# Compare checksums
if [ "$NEW_CHECKSUM" != "$CURRENT_CHECKSUM" ]; then
    echo "ConfigMap changed, updating deployment..."

    # Update deployment with new checksum
    kubectl patch deployment $DEPLOYMENT_NAME -n $NAMESPACE -p \
        "{\"spec\":{\"template\":{\"metadata\":{\"annotations\":{\"checksum/config\":\"$NEW_CHECKSUM\"}}}}}"

    # Wait for rollout to complete
    kubectl rollout status deployment/$DEPLOYMENT_NAME -n $NAMESPACE

    echo "Deployment updated successfully!"
else
    echo "No ConfigMap changes detected."
fi
```

## Use Cases and Benefits

1. **Automatic Updates:**
   - Pods automatically restart when config changes
   - No manual intervention required
   - Rolling updates maintain availability

2. **Change Tracking:**
   - Easy to verify if config changes are applied
   - Track config versions through checksums
   - Audit trail of changes

3. **Consistent State:**
   - All pods use updated config
   - No mixed states in the cluster
   - Predictable behavior

## Common Pitfalls and Solutions

1. **Multiple ConfigMaps:**
```yaml
metadata:
  annotations:
    checksum/config1: "${SHA256_OF_CONFIGMAP1}"
    checksum/config2: "${SHA256_OF_CONFIGMAP2}"
```

2. **Handling Update Failures:**
```bash
# Add error handling to update script
if ! kubectl patch deployment $DEPLOYMENT_NAME -n $NAMESPACE -p \
    "{\"spec\":{\"template\":{\"metadata\":{\"annotations\":{\"checksum/config\":\"$NEW_CHECKSUM\"}}}}}"; then
    echo "Failed to update deployment"
    exit 1
fi
```

3. **Preventing Unnecessary Updates:**
```bash
# Add timestamp to reduce frequent updates
ANNOTATION_VALUE="${NEW_CHECKSUM}-$(date +%s)"
```

## Best Practices

1. **Version Control:**
   - Keep ConfigMaps in version control
   - Document changes properly
   - Use meaningful commit messages

2. **Monitoring:**
   - Track config changes in monitoring system
   - Alert on update failures
   - Monitor pod restart counts

3. **Backup:**
```bash
# Backup before updating
kubectl get configmap $CONFIGMAP_NAME -n $NAMESPACE -o yaml > config_backup.yaml
```

4. **Gradual Rollout:**
```yaml
spec:
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
```

Remember: The checksum pattern ensures your application configuration stays synchronized with your ConfigMaps, making configuration management more reliable and automated.

## 4. Missing Pods Investigation

Pods cannot be scheduled due to namespace quotas or insufficient node resources.
Pods relying on a `ServiceAccount` that isn't created or properly bound will fail to run.
ConfigMaps, Secrets, or PersistentVolumes referenced by pods may not exist or are misconfigured.

### Common Causes for Missing Pods

1. **Resource Constraints:**
```bash
# Check resource usage
kubectl top nodes
kubectl describe node <node-name> | grep -A 5 "Allocated resources"
```

2. **Failed Rollouts:**
```bash
# Check rollout status
kubectl rollout status deployment/myapp -n test

# Investigate rollout history
kubectl rollout history deployment/myapp -n test

# Understanding rollout restart
kubectl rollout restart deployment/myapp -n test
# This command creates new pods with updated configuration while terminating old ones
# Useful when:
# - ConfigMap/Secret changes need to be applied
# - Pod template annotations need updating
# - Container image needs to be refreshed
```

3. **Dependencies Missing:**
```yaml
# Check for missing ServiceAccount
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myapp-sa
  namespace: test
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      serviceAccountName: myapp-sa  # Must exist before pod creation
```

## 5. CreateContainerError Resolution

### Common Configuration Errors

1. **Environment Variable Issues:**
```yaml
# Incorrect Configuration
spec:
  containers:
  - name: myapp
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: wrong-secret  # Secret doesn't exist
          key: password

# Solution: Verify Secret/ConfigMap existence
kubectl get secret wrong-secret -n mynamespace
kubectl get configmap myconfig -n mynamespace
```

2. **Secret/ConfigMap Reference Debug:**
```bash
# Check if secret exists
kubectl get secret -n mynamespace

# Verify secret keys
kubectl get secret mysecret -o yaml

# Check ConfigMap data
kubectl get configmap myconfig -o yaml
```

## 6. Handling Endlessly Terminating Pods

### Force Pod Deletion
```bash
# Standard deletion with grace period
kubectl delete pod mypod -n mynamespace

# Force deletion
kubectl delete pod mypod -n mynamespace --grace-period=0 --force

# If still stuck, check finalizers
kubectl get pod mypod -n mynamespace -o yaml | grep finalizers -A 5

# Remove finalizers if necessary
kubectl patch pod mypod -n mynamespace -p '{"metadata":{"finalizers":null}}'
```

## 7. EnableServiceLinks Issues

### Managing Service Link Environment Variables

**Problem:**
```yaml
# Too many environment variables due to service links
spec:
  containers:
  - name: myapp
    env:
    - name: SERVICE_PORT
      value: "8080"
  # Default enableServiceLinks: true causes conflicts
```

**Solution:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  enableServiceLinks: false  # Disable automatic service link injection
  containers:
  - name: myapp
    env:
    - name: SERVICE_PORT
      value: "8080"
```

### Troubleshooting Commands Reference
```bash
# Resource Quota
kubectl describe resourcequota -n mynamespace
kubectl get pods -n mynamespace --show-labels

# Missing Pods
kubectl get events -n mynamespace --sort-by='.lastTimestamp'
kubectl describe deployment myapp -n mynamespace

# ConfigMap/Secret
kubectl get configmap --all-namespaces
kubectl get secret --all-namespaces

# Pod Termination
kubectl get pod mypod -o yaml
kubectl logs mypod --previous

# Service Links
kubectl describe pod myapp
kubectl get pod myapp -o yaml | grep enableServiceLinks
```

### Best Practices

1. **Resource Management:**
   - Regularly monitor resource usage
   - Set appropriate requests and limits
   - Use namespace quotas wisely

2. **Configuration Management:**
   - Use version control for ConfigMaps/Secrets
   - Implement automated deployment updates
   - Document immutable field requirements

3. **Service Links:**
   - Disable when not needed
   - Document required environment variables
   - Use explicit environment variable definitions

4. **Pod Lifecycle:**
   - Implement proper readiness/liveness probes
   - Handle termination gracefully
   - Document resource dependencies

---
## Next Steps

In Part 9, we'll explore the `kubectl debug command` this will give you more access and insight into your Kubernetes resources, allowing you to troubleshoot issues more effectively and understand the state of your applications in greater detail.

---
[Next Article: Deep Dive into kubectl debug Commands →](./9_kubectl-debug.md)