## Namespaces

In Linux, namespaces are a core feature of the kernel that provide process isolation and resource separation. They allow multiple processes to have their own isolated view of system resources, preventing interference between different processes or groups of processes. Each namespace provides an independent instance of a particular resource, making it appear as if each process or group of processes has its own isolated environment.

Namespaces are used to implement containerization technologies like Docker and other forms of process isolation. Here are some of the key namespaces in Linux:

1. **PID Namespace (pid)**: This isolates process IDs. Each process within a namespace has its own unique set of process IDs, starting from 1. This allows processes in different namespaces to have the same process ID without conflicting with each other.

2. **Network Namespace (net)**: This isolates network-related resources, including network interfaces, IP addresses, routing tables, and firewall rules. Each network namespace has its own isolated network stack, allowing processes to have separate network configurations.

3. **Mount Namespace (mnt)**: This isolates filesystem mount points. Each mount namespace has its own filesystem hierarchy, and mounts made within a namespace do not affect other namespaces.

4. **UTS Namespace (uts)**: This isolates the hostname and domain name of a system. Processes in different UTS namespaces can have distinct hostnames, making them appear as separate system instances.

5. **IPC Namespace (ipc)**: This isolates inter-process communication resources, such as message queues, semaphores, and shared memory segments. Processes in different IPC namespaces cannot directly communicate or interfere with each other's IPC mechanisms.

6. **User Namespace (user)**: This isolates user and group IDs. It allows mapping of user and group IDs between the namespace and the host system, which is useful for enhanced security and privilege separation.

7. **Cgroup Namespace (cgroup)**: This isolates control groups, which are used to manage resource usage and limits for groups of processes. This namespace is not as commonly used as the others.



## CGroups


Control Groups, or CGroups, are a Linux kernel feature that allows you to manage and limit the resource usage of processes and groups of processes. Docker leverages CGroups to provide resource isolation and control for containers. With CGroups, Docker can allocate and manage resources such as CPU, memory, disk I/O, and network bandwidth for individual containers, ensuring fair resource sharing and preventing a single container from monopolizing the resources of the host system.

Here's how Docker uses CGroups:

1. **Resource Allocation**: When you start a Docker container, Docker creates a new CGroup for that container. This CGroup acts as a container for all the processes within the container. Docker can then allocate a specific portion of system resources (CPU, memory, etc.) to this CGroup, which in turn limits the resources available to the container.

2. **Resource Limiting**: CGroups allow Docker to set resource limits for containers. For example, you can specify the maximum amount of CPU time a container can use, the maximum memory it can consume, and other resource constraints. This ensures that containers share resources in a controlled manner and prevents any single container from using more than its fair share.

3. **Resource Monitoring**: Docker uses CGroups to monitor the resource usage of containers in real-time. This information is crucial for understanding how containers are utilizing resources and whether they are operating within their resource limits.

4. **Fair Resource Sharing**: CGroups help Docker ensure that containers get a fair share of resources based on their limits and requirements. This is important for preventing one resource-hungry container from negatively impacting the performance of other containers or the host system.

5. **Isolation**: CGroups provide a level of isolation between containers. By isolating resources using CGroups, Docker ensures that resource contention is minimized and that containers do not interfere with each other's performance.

6. **Container Termination**: When a container is stopped or removed, Docker cleans up the associated CGroup, freeing up the resources that were allocated to the container.

