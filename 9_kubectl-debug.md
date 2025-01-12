# Troubleshooting Kubernetes for Application Developers: Part 9 - Mastering `kubectl debug`

Debugging Kubernetes applications can be challenging, especially in complex, distributed environments. The `kubectl debug` command is a game-changer for application developers, offering an interactive and versatile approach to troubleshooting. Whether you're attaching a debugging container to a running pod, modifying an existing workload, or creating a replica of a pod for isolated testing, `kubectl debug` provides the tools you need to diagnose and resolve issues effectively.

This guide delves deep into the capabilities of `kubectl debug`, complete with practical examples and expert tips, to empower developers with the knowledge to troubleshoot Kubernetes like a pro.

---
#### **Examples of `kubectl debug` Usage**

1. **Create an interactive debugging session in an existing pod**
   ```bash
   kubectl debug mypod -it --image=busybox
   ```
   This command creates a debugging container using the `busybox` image and attaches you to it interactively.

---

2. **Create an interactive debugging session from a pod specification file**
   ```bash
   kubectl debug -f pod.yaml -it --image=busybox
   ```
   - Requires the **EphemeralContainers** feature to be enabled.
   - Useful for debugging pods defined in YAML files.

---

3. **Create a debug container with a custom image**
   ```bash
   kubectl debug --image=myproj/debug-tools -c debugger mypod
   ```
   - Adds a new container named `debugger` to the existing pod `mypod` using a custom debugging image.

---

4. **Create a copy of a pod and add a debug container**
   ```bash
   kubectl debug mypod -it --image=busybox --copy-to=my-debugger
   ```
   - Creates a duplicate of `mypod` named `my-debugger` and attaches a debug container to it.

---

5. **Modify a pod's container command**
   ```bash
   kubectl debug mypod -it --copy-to=my-debugger --container=mycontainer -- sh
   ```
   - Changes the command of the container `mycontainer` in the copied pod to `/bin/sh`.

---

6. **Change all container images in a copied pod**
   ```bash
   kubectl debug mypod --copy-to=my-debugger --set-image=*=busybox
   ```
   - Updates all container images in the copied pod to `busybox`.

---

7. **Combine debugging containers and image updates**
   ```bash
   kubectl debug mypod -it --copy-to=my-debugger --image=debian --set-image=app=app:debug,sidecar=sidecar:debug
   ```
   - Creates a debugging container with the `debian` image and updates the images of the `app` and `sidecar` containers in the copied pod.

---

8. **Debug a Kubernetes node**
   ```bash
   kubectl debug node/mynode -it --image=busybox
   ```
   - Creates a container on the node `mynode` with access to the host's namespaces and filesystem, mounted at `/host`.

---

#### **Detailed Options for `kubectl debug`**
Below are the key options for the `kubectl debug` command, with explanations:

| **Option**                   | **Description**                                                                                                         |
|------------------------------|-------------------------------------------------------------------------------------------------------------------------|
| `--arguments-only`           | Passes everything after `--` as arguments to the new container.                                                         |
| `--attach`                   | Waits for the container to start running and then attaches to it. Defaults to `true` if `-i/--stdin` is set.            |
| `-c, --container`            | Specifies the name of the container for debugging.                                                                      |
| `--copy-to`                  | Creates a copy of the target pod with the specified name.                                                               |
| `--custom`                   | Path to a JSON or YAML file for customizing the debug container spec.                                                   |
| `--env`                      | Sets environment variables in the debug container. Example: `--env=KEY=VALUE`.                                          |
| `-f, --filename`             | Specifies the resource file (YAML/JSON) to debug.                                                                       |
| `--image`                    | Specifies the image for the debug container. Example: `--image=busybox`.                                                |
| `--image-pull-policy`        | Sets the pull policy for the debug container image (e.g., `Always`, `IfNotPresent`).                                     |
| `--keep-annotations`         | Retains the original pod's annotations when copying it.                                                                 |
| `--keep-init-containers`     | Keeps init containers in the copied pod. Defaults to `true`.                                                            |
| `--keep-labels`              | Retains the original pod's labels when copying it.                                                                      |
| `--keep-liveness`            | Keeps liveness probes in the copied pod.                                                                                |
| `--keep-readiness`           | Keeps readiness probes in the copied pod.                                                                               |
| `--keep-startup`             | Keeps startup probes in the copied pod.                                                                                 |
| `--profile`                  | Sets the debug profile. Options: `legacy`, `general`, `baseline`, `netadmin`, `restricted`, `sysadmin`. Default: `legacy`.|
| `--replace`                  | Deletes the original pod when creating a copy.                                                                          |
| `--set-image`                | Updates container images in the copied pod. Example: `--set-image=container1=busybox`.                                  |
| `--share-processes`          | Enables process namespace sharing in the copied pod. Default: `true`.                                                   |
| `-i, --stdin`                | Keeps stdin open in the debug container.                                                                                |
| `-t, --tty`                  | Allocates a TTY for the debug container.                                                                                 |

---

#### **Practical Examples**

1. **Debugging environment issues**
   - Use the `--env` flag to add variables during debugging:
     ```bash
     kubectl debug mypod -it --image=busybox --env=DEBUG=true
     ```

2. **Customizing the debug container**
   - Use `--custom` to specify a detailed container spec:
     ```bash
     kubectl debug mypod --custom=custom-debug-config.yaml
     ```

3. **Debugging a workload without affecting production**
   - Create a duplicate pod and experiment with changes:
     ```bash
     kubectl debug mypod --copy-to=debug-pod --set-image=*=debug-tools --replace
     ```

4. **Accessing a node’s environment**
   - Debug network or configuration issues directly on the node:
     ```bash
     kubectl debug node/mynode -it --image=debian
     ```
---
### **Advanced Debugging Concepts**

#### 1. **Understanding Ephemeral Containers**
   - **What are Ephemeral Containers?**
     Ephemeral containers are temporary containers that are added to a running pod for debugging purposes. They do not modify the original pod definition and are designed for non-intrusive troubleshooting.
   - **Use Cases:**
     - Debugging crashes without restarting the pod.
     - Inspecting live network traffic within the pod.
   - **Pre-Requisites:**
     Ensure that the `EphemeralContainers` feature gate is enabled in your Kubernetes cluster. This can be checked by running:
     ```bash
     kubectl get nodes -o yaml | grep ephemeralcontainers
     ```
     If not enabled, update the Kubernetes API server configuration to include `--feature-gates=EphemeralContainers=true`.

---

#### 2. **Debugging Stateful Workloads**
   Stateful applications often require additional context, such as persistent volumes or config files. Here’s how to troubleshoot:

   - **Access Stateful Data:**
     ```bash
     kubectl debug mypod -it --image=busybox -- sh -c "cat /mnt/data/logs"
     ```
     In this example, `/mnt/data` is the mount point of the Persistent Volume (PV) attached to the pod.

   - **Inspecting Configurations:**
     Use `kubectl exec` to inspect mounted configuration files, then transition to `kubectl debug` for in-depth troubleshooting.

---

#### 3. **Debugging Nodes**
   - **Interactive Node Access:**
     ```bash
     kubectl debug node/mynode -it --image=busybox
     ```
     This grants access to the node's filesystem at `/host`, allowing you to inspect system logs (`/var/log`) or network configurations (`/etc/netplan`).
   - **Host Network Debugging:**
     Use the `--share-processes` option to enable process namespace sharing:
     ```bash
     kubectl debug node/mynode -it --share-processes --image=busybox
     ```

---

#### 4. **Comprehensive Debugging Workflows**

1. **Scenario: Application Crash on Startup**
   - **Symptoms:** Pods fail with `CrashLoopBackOff`.
   - **Debugging Steps:**
     ```bash
     kubectl debug mypod -it --image=busybox --copy-to=debug-pod
     kubectl logs debug-pod
     ```
     Analyze logs for startup issues, inspect environment variables, or test alternate configurations.

2. **Scenario: Network Connectivity Issues**
   - **Symptoms:** Pod unable to reach external services.
   - **Debugging Steps:**
     ```bash
     kubectl debug mypod -it --image=busybox
     ping 8.8.8.8  # Test basic connectivity
     nslookup google.com  # Test DNS resolution
     ```
     Use network utilities like `ping`, `curl`, or `dig` in the debug container.

3. **Scenario: Pod Scheduling Issues**
   - **Symptoms:** Pod stuck in `Pending` state.
   - **Debugging Steps:**
     ```bash
     kubectl describe pod mypod
     kubectl debug node/mynode -it --image=busybox
     ```
     Inspect node logs or verify resource constraints.

---

### **In-Depth Debugging Examples**

#### 1. **Debugging with Custom Permissions**
   For secure environments, use the `--profile` option to limit privileges:
   ```bash
   kubectl debug mypod --copy-to=restricted-debugger --profile=restricted
   ```
   Profiles like `restricted` or `baseline` align with Pod Security Standards (PSS).

#### 2. **Injecting Custom Tools**
   If your application container lacks debugging tools:
   ```bash
   kubectl debug mypod --image=myregistry/debug-tools:latest -c debug-container
   ```
   Replace `myregistry/debug-tools:latest` with a container image pre-loaded with utilities like `tcpdump`, `netcat`, or `strace`.

#### 3. **Dynamic Pod Inspection**
   Debugging dynamic environments where pods scale up and down:
   ```bash
   kubectl debug deploy/mydeployment --image=busybox -- sh -c "env"
   ```
   This command attaches to one of the pods in the deployment for real-time debugging.

---

### **Advanced Tips and Practices**

1. **Logging and Metrics Correlation:**
   - Pair `kubectl debug` with tools like Prometheus and Grafana to correlate debugging actions with metrics trends.

2. **Ephemeral Debugging Containers vs. Pod Duplication:**
   - **Ephemeral Containers:** Lightweight and direct, ideal for transient issues.
   - **Pod Duplication:** Safer for deeper investigation in production, as it doesn’t affect the original pod.

3. **Use Debug Profiles Strategically:**
   - `netadmin`: For debugging complex network policies or connectivity issues.
   - `sysadmin`: For diagnosing kernel-level or node-specific issues.
   - Example:
     ```bash
     kubectl debug mypod --profile=netadmin
     ```

4. **Automating Debugging Workflows:**
   - Integrate `kubectl debug` with CI/CD pipelines to automate fault isolation:
     ```bash
     kubectl debug mypod --copy-to=debug-pod --image=myregistry/debug-tools --replace
     ```

5. **Security Considerations:**
   - Always ensure debugging actions adhere to your organization’s security policies.
   - Use Role-Based Access Control (RBAC) to restrict `kubectl debug` usage to authorized personnel.

---

### **Tools to Complement `kubectl debug`**

- **Ksniff**: Packet capturing tool for debugging network issues.
  ```bash
  kubectl sniff mypod
  **Ensure you have the kubectl sniff plugin installed.**
  ```

- **K9s**: A terminal-based UI for Kubernetes troubleshooting.
  ```bash
  k9s
  ```

- **stern**: Real-time log streaming with filtering capabilities.
  ```bash
  stern mypod
  ```

---

#### **Tips and Best Practices**
- Always use the `--copy-to` option when debugging critical production pods to avoid disruptions.
- Use the `--profile` option to align debugging permissions with security policies.
- When debugging nodes, ensure you have sufficient privileges and understand the host environment to avoid accidental changes.

This detailed guide should serve as a comprehensive reference for utilizing the `kubectl debug` command effectively.
---
[Next Article: Deep Dive into kubectl port-foward & serviceaccount commands →](./10_kubectl-port-forward.md)
---
