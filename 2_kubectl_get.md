# Troubleshooting Kubernetes for Application Developers: Part 2 - Mastering kubectl get

## Understanding kubectl get

The `kubectl get` command is your window into the Kubernetes cluster. Think of it as your primary reconnaissance tool - it's often the first command you'll run when investigating issues or checking the status of your applications.

## Basic Syntax

```bash
kubectl get [resource] [name] [flags]
```

## Core Features

### 1. Resource Types
You can retrieve various resource types:
```bash
# Get specific resource types
kubectl get pods                  # List all pods
kubectl get services             # List all services
kubectl get deployments         # List all deployments
kubectl get all                 # List all resources

# Get multiple resource types
kubectl get pods,services       # List pods and services
```

### 2. Namespace Operations
```bash
# Get resources from specific namespace
kubectl get pods -n my-namespace

# Get resources from all namespaces
kubectl get pods --all-namespaces
# or
kubectl get pods -A
```

### 3. Output Formatting
The real power of `kubectl get` lies in its output formatting options:

#### Plain Text (default)
```bash
kubectl get pods
```
```
NAME                    READY   STATUS    RESTARTS   AGE
web-7f6d8d85f9-abcd    1/1     Running   0          24h
```

#### Wide Output
```bash
kubectl get pods -o wide
```
```
NAME                    READY   STATUS    RESTARTS   AGE   IP           NODE
web-7f6d8d85f9-abcd    1/1     Running   0          24h   10.244.0.5   worker-1
```

#### YAML Format
```bash
kubectl get pod web-7f6d8d85f9-abcd -o yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-7f6d8d85f9-abcd
spec:
  containers:
  - name: nginx
    image: nginx:1.19
```

#### JSON Format
```bash
kubectl get pod web-7f6d8d85f9-abcd -o json
```

#### Custom Columns
```bash
kubectl get pods -o custom-columns=POD:metadata.name,STATUS:status.phase
```
```
POD                    STATUS
web-7f6d8d85f9-abcd    Running
```

### 4. Filtering and Sorting

#### Label Selectors
```bash
# Get pods with specific label
kubectl get pods -l app=nginx

# Get pods with multiple labels
kubectl get pods -l app=nginx,tier=frontend
```

#### Field Selectors
```bash
# Get pods on specific node
kubectl get pods --field-selector spec.nodeName=worker-1

# Combine multiple fields
kubectl get pods --field-selector status.phase=Running,spec.nodeName=worker-1
```

### 5. Watch Mode
Monitor resources in real-time:
```bash
kubectl get pods -w
```

## Common Troubleshooting Patterns

### 1. Check Pod Status
```bash
# Get pods with their status
kubectl get pods --sort-by=.status.phase

# Find pods not running
kubectl get pods --field-selector status.phase!=Running
```

### 2. Resource Usage
```bash
# Get node resource usage
kubectl get nodes --sort-by=.status.capacity.cpu

# Get pods sorted by restart count
kubectl get pods --sort-by=.status.containerStatuses[0].restartCount
```

### 3. Quick Health Check
```bash
# Check all workload resources
kubectl get all -n my-namespace

# Check pod conditions
kubectl get pods -o custom-columns=NAME:.metadata.name,READY:.status.conditions[?(@.type=='Ready')].status
```

## Pro Tips

### 1. Short Names
Save typing with resource short names:
```bash
kubectl get po  # instead of pods
kubectl get svc # instead of services
kubectl get dep # instead of deployments
```

### 2. Output Shortcuts
```bash
kubectl get pods -owide  # No space needed
kubectl get pods -ojson  # Shorthand for -o json
```

### 3. Useful Combinations
```bash
# Get all failed pods across all namespaces
kubectl get pods --all-namespaces --field-selector status.phase=Failed

# Get pods with their images
kubectl get pods -o custom-columns=NAME:.metadata.name,IMAGE:.spec.containers[*].image
```

## Common Issues and Solutions

### 1. No Resources Found
```bash
$ kubectl get pods
No resources found in default namespace.
```
**Solution**: Check if:
- You're in the correct namespace
- The resources exist
- You have proper permissions

### 2. Forbidden Error
```bash
Error from server (Forbidden): pods is forbidden
```
**Solution**: Verify your:
- RBAC permissions
- kubectl context
- Namespace access

## Practice Exercises

Try these commands to build familiarity:

1. List all pods sorted by age:
```bash
kubectl get pods --sort-by=.metadata.creationTimestamp
```

2. Show pods with their node assignments:
```bash
kubectl get pods -o custom-columns=NAME:.metadata.name,NODE:.spec.nodeName
```

3. Find pods with high restart counts:
```bash
kubectl get pods --sort-by=.status.containerStatuses[0].restartCount
```

## What's Next?

In Next Part, we'll dive deep into pod troubleshooting, where we'll build upon these `kubectl describe` commands and combine them with other tools for comprehensive pod debugging.

---

[Next Article: Deep Dive into kubectl describe Commands →](./3_kubectl_describe.md)

---
