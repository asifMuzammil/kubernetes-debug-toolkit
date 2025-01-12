# Troubleshooting Kubernetes for Application Developers: Part 5 - Mastering kubectl logs

## Understanding kubectl logs

Container logs are your window into application behavior. The `kubectl logs` command is crucial for debugging applications running in Kubernetes, helping you understand what's happening inside your containers without having to directly access them.

## Basic Syntax
```bash
kubectl logs [-f] [-p] (POD | TYPE/NAME) [-c CONTAINER] [flags]
```

## Core Features

### 1. Basic Log Retrieval
```bash
# Get logs from a pod
kubectl logs my-pod

# Get logs from specific container in a pod
kubectl logs my-pod -c my-container

# Get logs with timestamps
kubectl logs my-pod --timestamps
```

### 2. Real-time Log Streaming
```bash
# Stream logs in real-time
kubectl logs -f my-pod

# Stream logs with timestamps
kubectl logs -f my-pod --timestamps
```

### 3. Historical Log Analysis
```bash
# Get logs from previous container instance
kubectl logs my-pod --previous

# Get logs from the last hour
kubectl logs my-pod --since=1h

# Get logs since specific timestamp
kubectl logs my-pod --since-time=2024-01-10T10:00:00Z
```

## Advanced Usage Patterns

### 1. Multi-Container Scenarios
```bash
# Get logs from all containers
kubectl logs my-pod --all-containers=true

# Get logs from specific container
kubectl logs my-pod -c nginx
```

### 2. Label-based Log Retrieval
```bash
# Get logs from pods with label
kubectl logs -l app=frontend

# Get logs from all containers matching label
kubectl logs -l app=frontend --all-containers=true
```

### 3. Log Output Control
```bash
# Get last N lines
kubectl logs my-pod --tail=100

# Limit log size
kubectl logs my-pod --limit-bytes=1000000

# Output to file
kubectl logs my-pod > pod_logs.txt
```

## Common Troubleshooting Patterns

### 1. Application Startup Issues
```bash
# Check initialization logs
kubectl logs my-pod --since=5m --timestamps

# Monitor startup in real-time
kubectl logs -f my-pod --timestamps
```

### 2. Crash Investigation
```bash
# Check logs before crash
kubectl logs my-pod --previous

# Get logs with context
kubectl logs my-pod --previous --timestamps --tail=500
```

### 3. Performance Analysis
```bash
# Monitor real-time performance logs
kubectl logs -f my-pod | grep "performance"

# Analyze error patterns
kubectl logs my-pod | grep -i error
```

## Log Analysis Best Practices

### 1. Structured Logging
```bash
# Parse JSON logs
kubectl logs my-pod | jq '.'

# Filter specific JSON fields
kubectl logs my-pod | jq 'select(.level == "error")'
```

### 2. Time-based Analysis
```bash
# Morning logs
kubectl logs my-pod --since-time="2024-01-10T06:00:00Z" --until-time="2024-01-10T12:00:00Z"

# Last hour's errors
kubectl logs my-pod --since=1h | grep ERROR
```

### 3. Cross-Pod Analysis
```bash
# Get logs from multiple pods
for pod in $(kubectl get pods -l app=myapp -o name); do
  echo "=== Logs from $pod ==="
  kubectl logs $pod --tail=50
done
```

## Advanced Debugging Techniques

### 1. Log Correlation
```bash
# Track request across pods
kubectl logs -l app=myapp | grep "request-id: abc123"
```

### 2. Error Pattern Analysis
```bash
# Count error occurrences
kubectl logs my-pod | grep -i error | sort | uniq -c

# Find peak error times
kubectl logs my-pod --timestamps | grep -i error
```

### 3. Performance Monitoring
```bash
# Monitor response times
kubectl logs my-pod | grep "response_time" | awk '{print $NF}' | sort -n

# Track resource usage patterns
kubectl logs my-pod | grep "memory_usage"
```

## Pro Tips

### 1. Custom Log Formats
```bash
# Create alias for common log patterns
alias klogs='kubectl logs -f --timestamps'
alias kerrors='kubectl logs --since=1h | grep -i error'
```

### 2. Log Filtering Scripts
```bash
#!/bin/bash
# get-pod-errors.sh
pod=$1
kubectl logs $pod --since=1h | grep -i error > "${pod}_errors.log"
```

### 3. Real-time Monitoring
```bash
# Monitor multiple pods simultaneously
kubectl logs -f -l app=myapp --all-containers=true --prefix=true
```

## Common Issues and Solutions

### 1. Log Size Limitations
```bash
# Handle large logs
kubectl logs my-pod --tail=1000 --limit-bytes=1048576
```

### 2. Container Restarts
```bash
# Check logs across restarts
kubectl logs my-pod --previous
kubectl logs my-pod # current instance
```

### 3. Missing Logs
```bash
# Verify log configuration
kubectl describe pod my-pod | grep -A 10 "Events:"
```

## Practice Exercises

1. Monitor application startup:
```bash
kubectl logs -f my-pod --since=0s
```

2. Analyze error patterns:
```bash
kubectl logs my-pod | grep -i error | sort | uniq -c | sort -nr
```

3. Track specific request flow:
```bash
kubectl logs -l app=myapp | grep "transaction-id: xyz789"
```

## Next Steps

In Part 6, we'll explore `kubectl exec` and how to interact directly with containers for advanced debugging.

---
[Next Article: Deep Dive into exited codes →](./6_exited_codes.md)
