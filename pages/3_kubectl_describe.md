# Troubleshooting Kubernetes for Application Developers: Part 3 - Mastering kubectl describe

## Understanding kubectl describe

While `kubectl get` gives you a quick overview, `kubectl describe` is your magnifying glass - it provides detailed information about Kubernetes resources, including events, conditions, and related resources. This command is invaluable when you need to understand why a resource isn't behaving as expected.

## Basic Syntax

```bash
kubectl describe [resource_type] [resource_name] [flags]
```

## Key Features and Use Cases

### 1. Pod Investigation
```bash
kubectl describe pod <pod-name>
```

This command reveals crucial information like:
- Pod status and conditions
- Container images and ports
- Volume mounts
- Environment variables
- Recent events
- Resource limits and requests

Example output sections:
```
Name:         web-app-7b9fd7c5d5-x2v4j
Namespace:    default
Priority:     0
Node:         worker-1/10.0.0.5
Start Time:   Mon, 10 Jan 2025 10:00:00 +0000

Status:       Running
IP:           172.17.0.5
Containers:
  web-app:
    Container ID:   docker://abc123...
    Image:          nginx:1.19
    State:          Running
      Started:      Mon, 10 Jan 2025 10:00:05 +0000
    Ready:          True
    Restart Count:  0

Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  5m    default-scheduler  Successfully assigned default/web-app to worker-1
  Normal  Pulled     4m    kubelet            Container image "nginx:1.19" already present on machine
```

### 2. Deployment Analysis
```bash
kubectl describe deployment <deployment-name>
```

Reveals information about:
- Deployment strategy
- Replica status
- Pod template
- Rolling update configuration
- Deployment conditions

### 3. Service Investigation
```bash
kubectl describe service <service-name>
```

Shows details about:
- Service type and IP
- Port mappings
- Endpoints (pod IPs)
- Selector information

### 4. Node Examination
```bash
kubectl describe node <node-name>
```

Provides information about:
- Node capacity and allocatable resources
- System info
- Node conditions
- Resource usage
- Pods running on the node

## Common Troubleshooting Patterns

### 1. Pod Startup Issues
```bash
# Check pod events for startup problems
kubectl describe pod <pod-name> | grep -A 10 Events
```

Common event messages and their meanings:
```
ImagePullBackOff: Container image pull failed
CrashLoopBackOff: Container is crashing repeatedly
Pending: Pod cannot be scheduled
```

### 2. Resource Constraints
```bash
# Check if pods are being throttled
kubectl describe pod <pod-name> | grep -A 5 Limits
```

Example output:
```
Limits:
  cpu:     500m
  memory:  256Mi
Requests:
  cpu:     250m
  memory:  128Mi
```

### 3. Network Issues
```bash
# Investigate service endpoint issues
kubectl describe service <service-name> | grep -A 5 Endpoints
```

### 4. Volume Problems
```bash
# Check volume mount details
kubectl describe pod <pod-name> | grep -A 10 Volumes
```

## Advanced Usage

### 1. Multiple Resources
```bash
# Describe all pods matching a label
kubectl describe pods -l app=nginx

# Describe multiple resource types
kubectl describe pods,services -l app=web
```

### 2. Cross-Namespace Investigation
```bash
# Describe resources in specific namespace
kubectl describe pod <pod-name> -n <namespace>

# Describe resources across all namespaces
kubectl describe pods --all-namespaces
```

## Common Issues and Their Events

### 1. Pod Scheduling Issues
```
Events:
  Type     Reason            Message
  ----     ------            -------
  Warning  FailedScheduling  0/3 nodes are available: 3 Insufficient memory
```

### 2. Container Image Issues
```
Events:
  Type     Reason          Message
  ----     ------          -------
  Warning  Failed          Failed to pull image "nginx:latest": rpc error: code = Unknown desc = Error response from daemon: pull access denied
```

### 3. Resource Limit Issues
```
Events:
  Type     Reason       Message
  ----     ------       -------
  Warning  FailedCreate  Error creating: pods "web-" is forbidden: exceeded quota: compute-resources
```

## Pro Tips

### 1. Event Filtering
```bash
# Show only Warning events
kubectl describe pod <pod-name> | grep -A 5 "Warning"

# Show events from last hour
kubectl describe pod <pod-name> | grep -A 5 "$(date -d '1 hour ago' +'%H:%M')"
```

### 2. Quick Debugging Scripts
```bash
# Create an alias for common describe patterns
alias kdp='kubectl describe pod'
alias kdd='kubectl describe deployment'
```

### 3. Output Processing
```bash
# Extract specific sections
kubectl describe pod <pod-name> | sed -n '/Events:/,$p'
```

## Practice Exercises

1. Investigate a failing pod:
```bash
kubectl describe pod <pod-name> | grep -A 10 Events
```

2. Check resource usage across nodes:
```bash
kubectl describe nodes | grep -A 5 "Allocated resources"
```

3. Analyze service connectivity:
```bash
kubectl describe service <service-name> | grep -A 5 Endpoints
```

## Next Steps

In Part 4, we'll explore `kubectl events` and how to effectively debug application issues through log analysis.

---
[Next Article: Deep Dive into kubectl events Commands →](./4_kubectl_events.md)
