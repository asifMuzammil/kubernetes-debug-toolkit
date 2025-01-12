# Troubleshooting Kubernetes for Application Developers: Part 4 - Understanding Kubernetes Events

## Introduction to Kubernetes Events

Events in Kubernetes are like your cluster's diary - they tell you what's happening, what went wrong, and what succeeded. For application developers, events are crucial for understanding deployment issues, resource constraints, and other problems that might affect your applications.

## Basic Event Commands

### 1. View Recent Events
```bash
# Get events in current namespace
kubectl get events

# Get events across all namespaces
kubectl get events --all-namespaces
# or
kubectl get events -A
```

### 2. Watch Events in Real-time
```bash
# Stream events as they happen
kubectl get events -w

# Stream events with timestamp
kubectl get events -w --sort-by=.metadata.creationTimestamp
```

## Event Types and Reasons

### Event Types
1. **Normal**: Routine operations and successes
2. **Warning**: Issues that require attention

### Common Event Reasons
```bash
# Filter events by reason
kubectl get events --field-selector reason=Failed
```

Common reasons include:
- `Created`: Resource creation
- `Started`: Container/Pod startup
- `Pulled`: Image pull completion
- `Failed`: Operation failures
- `Killing`: Container termination
- `Scheduled`: Pod scheduling
- `FailedScheduling`: Scheduling failures
- `BackOff`: Container restart delays

## Advanced Event Filtering

### 1. By Resource Type
```bash
# Get pod-related events
kubectl get events --field-selector involvedObject.kind=Pod

# Get deployment events
kubectl get events --field-selector involvedObject.kind=Deployment
```

### 2. By Specific Resource
```bash
# Events for specific pod
kubectl get events --field-selector involvedObject.name=my-pod

# Events for specific deployment
kubectl get events --field-selector involvedObject.name=my-deployment
```

### 3. By Event Type
```bash
# Show only warnings
kubectl get events --field-selector type=Warning

# Show normal events
kubectl get events --field-selector type=Normal
```

## Common Event Patterns and Their Meanings

### 1. Pod Lifecycle Events
```
LAST SEEN   TYPE      REASON      OBJECT          MESSAGE
2m          Normal    Scheduled   pod/web-app     Successfully assigned default/web-app to worker-1
1m          Normal    Pulling     pod/web-app     Pulling image "nginx:1.19"
1m          Normal    Pulled      pod/web-app     Successfully pulled image "nginx:1.19"
1m          Normal    Created     pod/web-app     Created container web-app
1m          Normal    Started     pod/web-app     Started container web-app
```

### 2. Resource Constraint Events
```
LAST SEEN   TYPE      REASON            OBJECT          MESSAGE
30s         Warning   FailedScheduling  pod/web-app     0/3 nodes are available: insufficient memory
```

### 3. Image Pull Issues
```
LAST SEEN   TYPE      REASON          OBJECT          MESSAGE
1m          Warning   Failed          pod/web-app     Failed to pull image "nginx:latest": rpc error: code = Unknown
45s         Normal    BackOff         pod/web-app     Back-off pulling image "nginx:latest"
```

## Event Investigation Patterns

### 1. Chronological Analysis
```bash
# Get events sorted by timestamp
kubectl get events --sort-by='.lastTimestamp'
```

### 2. Resource-Specific Investigation
```bash
# Get events related to a specific pod and its containers
kubectl get events --field-selector involvedObject.name=my-pod,involvedObject.kind=Pod
```

### 3. Namespace-Wide Analysis
```bash
# Get all warning events in a namespace
kubectl get events -n my-namespace --field-selector type=Warning
```

## Pro Tips

### 1. Custom Output Formats
```bash
# Show specific fields
kubectl get events -o custom-columns=TIMESTAMP:.lastTimestamp,TYPE:.type,REASON:.reason,MESSAGE:.message

# Output in JSON format for processing
kubectl get events -o json
```

### 2. Event Aggregation
```bash
# Count similar events
kubectl get events --sort-by='.metadata.creationTimestamp' | uniq -c
```

### 3. Time Window Analysis
```bash
# Get events from last hour
kubectl get events --field-selector 'lastTimestamp.time>=2025-01-10T10:00:00Z'
```

## Common Troubleshooting Scenarios

### 1. Deployment Rollout Issues
```bash
# Track deployment events
kubectl get events --field-selector involvedObject.kind=Deployment,reason=FailedCreate
```

### 2. Node Problems
```bash
# Monitor node-related events
kubectl get events --field-selector involvedObject.kind=Node
```

### 3. Storage Issues
```bash
# Check PersistentVolume events
kubectl get events --field-selector involvedObject.kind=PersistentVolume
```

## Practice Exercises

1. Monitor deployment rollout:
```bash
kubectl create deployment test-deploy --image=nginx
kubectl get events -w --field-selector involvedObject.kind=Deployment
```

2. Track resource constraints:
```bash
kubectl get events --field-selector type=Warning,reason=FailedScheduling
```

3. Investigate image issues:
```bash
kubectl get events --field-selector reason=Failed,reason=BackOff
```

## Next Steps

In Next Part, we'll explore `kubectl logs` and how to effectively analyze application logs for debugging.

---

[Next Article: Deep Dive into kubectl logs Commands →](./5_kubectl_logs.md)
