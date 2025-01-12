# Troubleshooting Kubernetes for Application Developers: Part 8/1 - Mastering Most Common Issues

## 1. ImagePullBackOff Errors

**Symptoms:**

Pods are stuck in the `ImagePullBackOff` or `ErrImagePull` state, indicating that Kubernetes is unable to pull the specified container image.

### Wrong Image Name
- **Cause:** The image name in the pod's configuration file is incorrect or does not exist in the registry.
- **Fix:** Verify the image name in the `spec.containers[].image` field. Check the registry to ensure the image exists.

```bash
kubectl get pods
NAME        READY   STATUS             RESTARTS   AGE
myapp-pod   0/1     ImagePullBackOff   0          2m
```

**Common Causes:**
- Typos in image name
- Missing or incorrect registry prefix
- Case sensitivity issues

**Solution:**
```yaml
# Incorrect
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: myapp
    image: myapp:latest  # Wrong

# Correct
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: myapp
    image: docker.io/mycompany/myapp:latest  # Correct with full path
```

**Debugging Commands:**
```bash
# Check pod events
kubectl describe pod myapp-pod

# Check available tags for image
docker pull mycompany/myapp --all-tags
```
#### Unable to Resolve Registry Name**
- **Cause:** The registry URL is incorrect or there are DNS issues.
- **Fix:** Use `curl` or `nslookup` from within the cluster to test if the registry is resolvable. Update the registry name in the pod spec if needed.

### ImagePullSecrets Missing
**Symptom:** Authentication error when pulling private images
- **Cause:** Private registries require authentication, but the credentials are not provided or misconfigured.

**Solution:**
```yaml
# Create secret for private registry
kubectl create secret docker-registry regcred \
  --docker-server=<registry-server> \
  --docker-username=<username> \
  --docker-password=<password> \
  --docker-email=<email>

# Use secret in pod
apiVersion: v1
kind: Pod
metadata:
  name: private-app
spec:
  containers:
  - name: app
    image: private-registry.com/app:1.0
  imagePullSecrets: # Missing or Not Define
  - name: regcred
```

### Authorization Issues
**Symptom:** "insufficient_scope: authorization failed" error

**Debugging Steps:**
1. Verify registry credentials:
```bash
docker login <registry-server>
```

2. Check secret creation:
```bash
kubectl get secrets regcred -o yaml
```

3. Validate secret in pod:
```bash
kubectl describe pod myapp-pod | grep -A5 Events
```

## 2. CrashLoopBackOff Issues

### Understanding CrashLoopBackOff

**What Is `CrashLoopBackOff`?**
This state signifies that a container inside the pod crashes shortly after starting. Kubernetes attempts to restart the container repeatedly, causing a "loop."

**Common Causes:**
1. Application errors
2. Missing dependencies
3. Invalid configuration
4. Resource constraints

### Debugging Steps:

1. **Check Pod Logs:**
```bash
# Get logs from current crash
kubectl logs mypod

# Get logs from previous crash
kubectl logs mypod --previous

# Get logs with timestamps
kubectl logs mypod --timestamps=true
```

2. **Check Container Configuration:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: myapp
    image: myapp:1.0
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
    livenessProbe:      # Add proper health checks
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```

3. **Debug with Interactive Shell:**
```bash
# Run debug container
kubectl run debug --rm -it --image=busybox -- /bin/sh

# Check application files and permissions
ls -la /app
cat /app/config.yaml
```

4.**Incorrect Command or Arguments**
- **Cause:** The `command` or `args` specified in the container spec are invalid or misconfigured.
- **Fix:** Review the container spec for errors in the `command` and `args` fields. Example:
  ```yaml
  containers:
  - name: my-app
    image: my-image
    command: ["/app/start"]
    args: ["--config=/etc/config"]
```

5 **Missing Dependencies**
- **Cause:** The application depends on services, files, or configurations that are unavailable.
- **Fix:** Ensure required ConfigMaps, Secrets, or services are created and properly mounted in the container.

6 **Resource Constraints**
- **Cause:** The container runs out of memory or CPU, leading to termination.
- **Fix:**
  - Monitor resource usage with `kubectl top pod`.
  - Update resource requests and limits in the pod spec:
    ```yaml
    resources:
      requests:
        memory: "256Mi"
        cpu: "500m"
      limits:
        memory: "512Mi"
        cpu: "1"
    ```

## 3. Pending Pod Issues
Pods are stuck in the `Pending` state, unable to be scheduled on any node.

### 1. Insufficient Resources
**Symptom:** Pod stays in Pending state due to insufficient CPU or memory

**Debugging:**
```bash
# Check node resources
kubectl describe nodes | grep -A 5 "Allocated resources"

# Check pod resource requests
kubectl describe pod pending-pod | grep -A 3 "Requests"
```

**Solution:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: right-sized-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    resources:
      requests:
        memory: "64Mi"    # Adjust based on actual needs
        cpu: "250m"
```

### 2. Label Selector Mismatch
**Symptom:** Pod can't be scheduled due to node selector mismatch

**Debugging:**
```bash
# Check node labels
kubectl get nodes --show-labels

# Check pod node selector
kubectl describe pod pending-pod | grep -A 5 "Node-Selectors"
```

**Solution:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: labeled-pod
spec:
  nodeSelector:
    environment: production    # Make sure nodes have this label
  containers:
  - name: app
    image: myapp:1.0
```

### 3. Taints and Tolerations
**Symptom:** Pod can't be scheduled due to node taints
- **Cause:** Taints on nodes prevent pod scheduling, and the pod does not have matching tolerations.
- **Fix:**
  - Check node taints:
    ```bash
    kubectl describe node <node-name>
    ```
  - Add appropriate tolerations in the pod spec:
    ```yaml
    tolerations:
    - key: "key"
      operator: "Equal"
      value: "value"
      effect: "NoSchedule"
    ```

**Debugging:**
```bash
# Check node taints
kubectl describe nodes | grep Taints

# Verify pod tolerations
kubectl describe pod pending-pod | grep -A 5 Tolerations
```

**Solution:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: toleration-pod
spec:
  tolerations:
  - key: "special"
    operator: "Equal"
    value: "true"
    effect: "NoSchedule"
  containers:
  - name: app
    image: myapp:1.0
```

### 4. Storage-Related Pending Issues

#### A. Pending Due to PVC Binding Issues
**Symptom:** Pod stays in Pending state with events showing "waiting for volume to be created"

**Debugging:**
```bash
# Check PVC status
kubectl get pvc
kubectl describe pvc my-claim

# Check PV status
kubectl get pv
kubectl describe pv my-volume

# Check storage class
kubectl get storageclass
kubectl describe storageclass standard
```

**Common Issue 1: StorageClass Not Found**
```yaml
# Incorrect PVC Configuration
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: non-existent-class  # StorageClass doesn't exist

# Solution: Use correct StorageClass
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: standard  # Use existing StorageClass
```

**Common Issue 2: Access Mode Mismatch**
```yaml
# Incorrect Configuration
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteMany  # Mismatch with PV
  resources:
    requests:
      storage: 1Gi

# Solution: Match access modes
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce  # Match PV access mode
  resources:
    requests:
      storage: 1Gi
```

#### B. Storage Class Provisioner Issues
**Symptom:** PVC stays in Pending state, no PV is created

**Debugging Steps:**
```bash
# Check StorageClass provisioner
kubectl describe storageclass standard

# Check provisioner pod logs (if using external provisioner)
kubectl logs -n kube-system -l app=provisioner

# Check events
kubectl get events --field-selector involvedObject.kind=PersistentVolumeClaim
```

**Example: Storage Class Configuration**
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs  # Make sure provisioner is running
parameters:
  type: gp2
  fsType: ext4
```

#### C. Volume Mount Permission Issues
**Symptom:** Pod stuck in ContainerCreating state with mount permission errors

**Solution:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    volumeMounts:
    - name: data-volume
      mountPath: /data
    securityContext:      # Add security context
      fsGroup: 2000
      runAsUser: 1000
  volumes:
  - name: data-volume
    persistentVolumeClaim:
      claimName: my-pvc
```

#### D. Troubleshooting Commands for Storage Issues

```bash
# Check PVC/PV status
kubectl get pvc,pv --all-namespaces
kubectl describe pvc my-claim
kubectl describe pv my-volume

# Check StorageClass
kubectl get sc
kubectl describe sc standard

# Check pod events
kubectl describe pod my-pod | grep -A 10 Events

# Check mount points in pod
kubectl exec -it my-pod -- df -h

# Check volume permissions
kubectl exec -it my-pod -- ls -la /mount/path

# Debug with utility pod
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: volume-debug
spec:
  containers:
  - name: debug
    image: busybox
    command: ['sleep', '3600']
    volumeMounts:
    - name: problematic-volume
      mountPath: /data
  volumes:
  - name: problematic-volume
    persistentVolumeClaim:
      claimName: my-pvc
EOF
```

## Best Practices

1. **Image Pull Issues:**
   - Always use specific image tags, not 'latest'
   - Verify registry credentials before deployment
   - Use secrets for private registries

2. **CrashLoopBackOff:**
   - Implement proper health checks
   - Set appropriate resource limits
   - Review logs immediately after deployment

3. **Pending Pods:**
   - Monitor cluster resource usage
   - Document node labels and taints
   - Use pod resource requests appropriately


#### E. Best Practices for Storage Configuration

1. **PVC/PV Management:**
   - Always specify storage class when creating PVC
   - Use appropriate access modes
   - Set reasonable resource requests
   - Consider using storage class with volume expansion enabled

2. **Security:**
   - Set appropriate fsGroup and runAsUser
   - Use appropriate SELinux contexts if required
   - Consider encryption requirements

3. **Monitoring:**
   - Monitor PV usage and capacity
   - Set up alerts for PVC binding failures
   - Monitor storage provisioner health

4. **Common Checks for Storage Issues:**
```bash
# Check if StorageClass exists
kubectl get storageclass

# Verify PVC is bound
kubectl get pvc -o wide

# Check if PV is available
kubectl get pv -o wide

# Monitor provisioner logs
kubectl logs -n kube-system -l app=storage-provisioner

# Check node storage capacity
kubectl describe nodes | grep "Allocated resources"

```

## Quick Reference Commands

```bash
# General Debugging Commands
kubectl get events --sort-by='.lastTimestamp'
kubectl describe pod <pod-name>
kubectl logs <pod-name> --previous
kubectl get pods -o wide
kubectl diff -f <manifest_file> ***This will check difference between Running resource and manifest_file***

# Resource Checking
kubectl top nodes
kubectl top pods

# Node Information
kubectl describe nodes | grep -A 5 "Allocated resources"
kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints

# Configuration Validation
kubectl explain pod.spec.containers.resources
kubectl explain pod.spec.tolerations
```
[Next Article: Deep Dive into k8s Most Common Issues Part-2→](./8_2_kubectl-common.md)