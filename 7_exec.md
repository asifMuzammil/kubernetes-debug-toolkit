# Troubleshooting Kubernetes for Application Developers: Part 7 - Mastering kubectl exec

## Understanding kubectl exec

The `kubectl exec` command is your direct line into running containers. Think of it as SSH for containers - it allows you to execute commands inside containers, inspect their state, and perform interactive debugging.

## Basic Syntax
```bash
kubectl exec [-it] POD [-c CONTAINER] -- COMMAND [args...]
```

## Core Features and Use Cases

### 1. Interactive Shell Sessions
```bash
# Start bash shell
kubectl exec -it my-pod -- /bin/bash

# Start sh shell (for minimal containers)
kubectl exec -it my-pod -- /bin/sh

# Specify container in multi-container pod
kubectl exec -it my-pod -c nginx -- /bin/bash
```

### 2. Direct Command Execution
```bash
# List files
kubectl exec my-pod -- ls -la

# Check process list
kubectl exec my-pod -- ps aux

# View file contents
kubectl exec my-pod -- cat /etc/config/app.conf
```

## Advanced Debugging Techniques

### 1. Network Diagnostics
```bash
# Test internal DNS
kubectl exec my-pod -- nslookup kubernetes.default

# Check network connectivity
kubectl exec my-pod -- curl -v service-name:8080

# Monitor network traffic
kubectl exec my-pod -- netstat -tuln
```

### 2. File System Operations
```bash
# Check disk usage
kubectl exec my-pod -- df -h

# Find large files
kubectl exec my-pod -- find /app -type f -size +100M

# Check file permissions
kubectl exec my-pod -- ls -la /app/config
```

### 3. Process Management
```bash
# View running processes
kubectl exec my-pod -- top

# Check memory usage
kubectl exec my-pod -- free -m

# List open files
kubectl exec my-pod -- lsof
```

## Common Debugging Scenarios

### 1. Application Configuration
```bash
# View environment variables
kubectl exec my-pod -- printenv

# Check configuration files
kubectl exec my-pod -- cat /app/config/settings.json

# Verify mounted volumes
kubectl exec my-pod -- mount | grep volumeName
```

### 2. Database Debugging
```bash
# Connect to database
kubectl exec -it my-pod -- mysql -u user -p

# Check Redis connection
kubectl exec my-pod -- redis-cli ping

# Monitor PostgreSQL
kubectl exec -it my-pod -- psql -U postgres
```

### 3. Web Application Testing
```bash
# Test HTTP endpoints
kubectl exec my-pod -- curl -v localhost:8080/health

# Check web server config
kubectl exec my-pod -- nginx -T

# View access logs
kubectl exec my-pod -- tail -f /var/log/nginx/access.log
```

## Advanced Use Cases

### 1. Multi-container Pod Debugging
```bash
# List containers in pod
kubectl get pod my-pod -o jsonpath='{.spec.containers[*].name}'

# Execute in specific container
kubectl exec my-pod -c sidecar -- ps aux

# Copy files between containers
kubectl exec my-pod -c container1 -- cp /tmp/file /shared/
kubectl exec my-pod -c container2 -- cat /shared/file
```

### 2. Resource Monitoring
```bash
# CPU usage
kubectl exec my-pod -- top -b -n 1

# Memory statistics
kubectl exec my-pod -- cat /proc/meminfo

# Disk I/O
kubectl exec my-pod -- iostat
```

### 3. Log Investigation
```bash
# Real-time log tailing
kubectl exec my-pod -- tail -f /var/log/app.log

# Search logs for errors
kubectl exec my-pod -- grep -r ERROR /var/log/

# Compress logs for analysis
kubectl exec my-pod -- tar czf /tmp/logs.tar.gz /var/log/
```

## Pro Tips

### 1. Shell Aliases
```bash
# Create useful aliases
alias kex='kubectl exec -it'
alias kexsh='kubectl exec -it -- /bin/sh'
alias kexbash='kubectl exec -it -- /bin/bash'
```

### 2. Debugging Scripts
```bash
# Create debug.sh
cat <<EOF > debug.sh
#!/bin/bash
echo "=== Environment Variables ==="
printenv
echo "=== Disk Usage ==="
df -h
echo "=== Network Connections ==="
netstat -tuln
EOF

# Execute script in pod
kubectl cp debug.sh my-pod:/tmp/
kubectl exec my-pod -- bash /tmp/debug.sh
```

### 3. One-liners
```bash
# Quick HTTP check
kubectl exec my-pod -- wget -qO- http://localhost:8080/health

# Check certificate expiry
kubectl exec my-pod -- openssl x509 -in /etc/ssl/cert.pem -noout -dates

# Monitor resource usage
kubectl exec my-pod -- sh -c 'while true; do ps aux; sleep 1; done'
```

## Best Practices

### 1. Security Considerations
- Avoid running as root when possible
- Use minimal base images
- Don't store sensitive data in container file system

### 2. Resource Management
- Be careful with resource-intensive commands
- Clean up temporary files after debugging
- Use timeouts for long-running commands

### 3. Troubleshooting Flow
1. Check basic connectivity
2. Verify configuration
3. Inspect logs
4. Test application functionality
5. Monitor resource usage

## Next Steps

Next Part, we'll explore into 2 Parts `Mastering Most Common Issues` Like imagePullerror, CrashLoopBackoff
---
[Next Article: Deep Dive into k8s Most Common Issues Part-1→](./8_1_kubectl-common.md)
---
