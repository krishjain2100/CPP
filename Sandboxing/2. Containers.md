A container is not a computer. It is just a standard software program (like a Python script or a database) running on your computer but the OS has wrapped an invisible, restrictive box around that program so it cannot see or touch any other files, programs, or networks on the machine.

When you run a container, there is **no Hypervisor**. The system relies on three purely software-based pillars built directly into the Linux Kernel to isolate these processes.

### 1. Namespaces

Namespaces are a Linux kernel feature that limits what a process can _see_. When you start a container, the kernel creates a custom set of namespaces just for that process.

There are several types of namespaces, but here are the three most critical:

- **PID Namespace (Process ID):** In Linux, the very first thing that boots is PID 1, and every other program branches off it. If you look at the host, your containerized app might be process ID `4598`. But inside the container's PID namespace, the app looks at itself and sees PID `1`. It thinks it is the absolute root of the computer.
    
- **Mount Namespace (File System):** This restricts the file system. The container cannot see the host's `C:\` drive or `/home/user` directory. It is completely locked into a tiny, fake root directory (`/`).
    
- **Network Namespace:** The kernel gives the container its own completely isolated virtual network interface, its own routing table, and its own IP address.
    
### 2. Cgroups

If Namespaces limit what a container can _see_, **Control Groups (cgroups)** limit what a container can _use_.

Because all containers share the same host kernel, a badly written C++ program with a memory leak in Container A could consume 100% of the physical RAM, crashing Container B, Container C, and the host itself.

Cgroups allow you to tell the kernel: _"Do not let Container A use more than 2 CPU cores and 512MB of RAM."_ If the process tries to allocate 513MB, the kernel's Out-Of-Memory (OOM) killer immediately steps in and forcefully terminates the container process to protect the rest of the system.


---

To sum up the architecture:

- **Virtual Machines** fake the **Hardware**. They use a Hypervisor to manage CPU privilege rings so you can run entirely different, heavy Operating Systems.
    
- **Containers** fake the **Operating System**. They use Linux Kernel tricks (Namespaces and cgroups) to put a tiny box around a standard process, sharing the same underlying kernel.


the **Security Risks** of containers sharing a kernel compared to the iron-clad hardware isolation of a VM?

