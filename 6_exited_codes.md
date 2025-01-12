# Troubleshooting Kubernetes for Application Developers: Part 6 - Understanding Exit Codes

When troubleshooting applications running within Kubernetes containers, exit codes play a crucial role in pinpointing the root cause of issues. These codes are integers returned by the container process upon termination, indicating the reason for its exit. By interpreting these exit codes, developers and administrators can gain valuable insights into application behavior and potential problems.

### Common Exit Codes and Their Meanings

Here's a breakdown of some frequently encountered exit codes in Kubernetes:

---

### **Common Exit Codes and Their Causes**

| Exit Code | Signal/Reason            | Cause/Description                                                                 |
|-----------|--------------------------|-----------------------------------------------------------------------------------|
| 0         | Successful               | The container completed successfully.                                            |
| 1         | General Error            | Application error or failure. Inspect logs for details.                          |
| 126       | Command Not Executable   | Command lacks executable permissions or is invalid.                              |
| 127       | Command Not Found        | Missing binary or incorrect path.                                                |
| 128       | Invalid Exit Argument    | Application exited with an invalid signal.                                       |
| 137       | SIGKILL (OOMKilled)      | Memory limits exceeded, or node memory pressure triggered termination.           |
| 139       | Segmentation Fault       | Application attempted to access restricted memory.                               |
| 143       | SIGTERM                  | Graceful termination; not handled within timeout.                                |
| 255       | Unknown Error            | Undefined error; consult logs and application documentation.                     |

---

**Detailed Explanation of OOMKilled (Exit Code 137):**

*   **Cause:** Container memory consumption surpasses the limits defined in the pod specification, particularly the `resources.limits.memory` setting. Alternatively, the host node where the pod resides might be experiencing overall memory pressure, leading to the OOM killer (a system process) terminating the container.
*   **Identification:**
    *   Use `kubectl describe pod <pod-name>` to inspect the pod's status. If the reason for termination is `OOMKilled`, the exit code will be displayed as `137`.
    *   Reviewing container logs using `kubectl logs <pod-name> --previous` can reveal clues about memory usage patterns before termination.
*   **Common Causes of OOMKilled:**
    *   **Exceeded Memory Limits:** The application's memory usage consistently exceeds the `resources.limits.memory` value allocated in the pod spec.
    *   **Memory Leaks:** The application code might contain memory leaks, gradually consuming more memory over time without proper release.
    *   **Host-Level Memory Pressure:** The node itself is running low on memory due to other workloads competing for resources.
*   **Resolving OOMKilled Issues:**
    *   **Increase Memory Limits:** Adjust the `resources.limits.memory` property in the pod or container definition to provide a higher memory allowance.
    *   **Optimize Application Memory Usage:** Identify and address memory leaks or areas of inefficient memory management within the application.
    *   **Enable Memory Monitoring:** Utilize tools like Prometheus and Grafana to track and monitor memory usage across pods and nodes.
    *   **Vertical Pod Autoscaler (VPA):** Implement a VPA to dynamically scale pod memory requests based on actual usage patterns. (See `kubectl apply` command in the previous section).
    *   **Adjust Host Node Resources:** If applicable, consider adding more nodes to the cluster or increasing the available memory on existing nodes.

### Additional Exit Codes

*   **139 (Segmentation Fault):** The application attempted to access memory that it's not allowed to, resulting in a segmentation fault. This typically indicates bugs or vulnerabilities in the application code that need to be addressed.
*   **143 (SIGTERM):** The container received a SIGTERM signal, which is typically used for graceful termination. However, if the application doesn't handle termination within a reasonable timeframe, it might be forcefully terminated with a different exit code.

**Exit Code 255 (Unknown Error):** This code signifies that the application exited with an undefined or unhandled error. Inspecting container logs and application-specific error messages is essential for further investigation.

---
### **Advanced Troubleshooting with Exit Codes**

1. **Debugging Permissions with Exit Code 126:**
   - Ensure the script or binary is executable:
     ```bash
     chmod +x <file>
     ```
   - Verify the correct working directory and permissions.

2. **Resolving Exit Code 127 (Command Not Found):**
   - Ensure the command is installed in the container:
     ```bash
     docker exec -it <container> which <command>
     ```
   - Verify the `PATH` variable includes the required directories.

3. **Segmentation Fault (Exit Code 139):**
   - Profile the application for invalid memory access using tools like **Valgrind** or **gdb**.
   - Check for incompatible libraries or binaries in the container image.

4. **SIGTERM (Exit Code 143):**
   - Ensure the application gracefully handles termination signals:
     ```python
     import signal

     def handle_termination(signum, frame):
         print("Received SIGTERM, cleaning up...")
         # Perform cleanup
         exit(0)

     signal.signal(signal.SIGTERM, handle_termination)
     ```

5. **Verbose Output with `--v=10`:**
   Adding `--v=10` to commands (e.g., `kubectl describe pod <pod-name> --v=10`) provides detailed debug logs from the Kubernetes API server, useful for deep troubleshooting.


**Key Takeaways:**

*   Exit codes provide valuable insights into container process termination.
*   Understanding common exit codes empowers developers and administrators to effectively troubleshoot application issues.
*   By analyzing exit codes in conjunction with container logs and resource usage metrics, you can pinpoint the root cause of problems and implement appropriate solutions.

**Further Reading:**

While there isn't one single definitive list of *all* possible exit codes specifically for Kubernetes (as many originate from the underlying operating system and the applications themselves), here are some helpful resources:

*   **Linux Standard Base Core Specification:** This specification defines some standard exit codes used in Linux environments, which are relevant to containers running on Linux nodes. You can find information on signals (which often translate to exit codes) here: [https://refspecs.linuxfoundation.org/LSB_5.0.0/LSB-Core-generic/LSB-Core-generic/baselib-exitcodes.html](https://refspecs.linuxfoundation.org/LSB_5.0.0/LSB-Core-generic/LSB-Core-generic/baselib-exitcodes.html)
*   **Kubernetes Documentation on Debugging:** The Kubernetes documentation provides valuable information on debugging pods and containers, including how to interpret exit codes and other diagnostic information: [https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/](https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/)
*   **Docker Documentation on Exit Codes:** Since Kubernetes uses Docker (or other container runtimes) under the hood, Docker's documentation on exit codes can also be useful: [https://docs.docker.com/engine/reference/run/#exit-status](https://docs.docker.com/engine/reference/run/#exit-status)

---

[Next Article: Deep Dive into kubectl exec command →](./7_exec.md)

---
