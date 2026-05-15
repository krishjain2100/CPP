A **Container** is a concept—a feature of the Linux Kernel. **Docker** is a product, a company and a suite of software tools that made that concept actually usable.

### Before Docker

Containers are just isolated processes created using Linux namespaces and cgroups. These kernel features have existed since around 2008 (known back then as LXC, or Linux Containers).

However, before Docker came along in 2013, using them was a nightmare.

If you wanted to build a container from scratch using C++, you would have to manually invoke the `clone()` system call, passing in complex flags like `CLONE_NEWPID` and `CLONE_NEWNET`. Then, you would have to manually mount the virtual file systems, write configuration files to the `/sys/fs/cgroup` directory to limit memory, and manually configure IP tables so the container could connect to the internet.

It was tedious, highly prone to error, and required deep Linux system administration knowledge. Only massive tech companies like Google were doing it at scale.

### What Docker Actually Did

Docker wrapped all of those terrifying, low-level Linux kernel APIs into a single, beautiful, developer-friendly software package. They introduced five major things:

### 1. **Union File System (UnionFS / OverlayFS)**

**Core Concept:** A filesystem service that allows multiple directories (layers) to be physically separate but appear as a single, unified directory tree. In Docker, this is used to share base image data across multiple containers while allowing each container to have its own unique changes.

#### **a. The Layered Architecture (OverlayFS)**

- **LowerDir (Read-Only):** These are the **Image Layers**. They are shared across all containers running that image. They never change.
    
- **UpperDir (Writable):** This is the **Container Layer**. It is unique to a specific container instance. All new files or modifications live here.
    
- **MergedDir (The View):** The unified "mount point." This is what the application actually sees and interacts with.

#### **b. Key Mechanisms**

- When looking for a file, the kernel looks from the top (Upper) down to the bottom (Lower). If a file exists in both, the version in the **UpperDir** hides the one in the **LowerDir**.

-  If you modify a file from a read-only layer, the kernel **copies** it up to the Writable Layer (UpperDir) first, then applies the changes. Subsequent reads will always hit the modified copy in the UpperDir.
  
- **Whiteouts (Deletion):** Since you can't delete files from read-only layers, deleting a file creates a "whiteout" file in the Writable Layer. This acts as a mask, hiding the file in the Merged View.

#### **c. Efficiency

- **Storage:** 50 containers based on the same 1GB image only consume **1GB + (small unique changes)** on disk, not 50GB.
    
- **Speed:** Creating a new container is a simple directory creation and mount operation—taking **milliseconds** regardless of image size.
    
- **Isolation:** Changes in one container's `UpperDir` are invisible to all other containers sharing the same `LowerDir`.

### 2. The Docker Engine 

This is a background process (a daemon) that runs on your machine. When you type a simple command like `docker run nginx`, the Docker Engine translates that friendly command into the hundreds of complex `clone()` syscalls, namespace configurations, and cgroup limits required by the Linux kernel. It acts as your automated system administrator.

### 2. The Dockerfile 

Before Docker, if you wanted to move an application to another server, you had to write a 10-page Word document explaining which packages to install. Docker introduced the **Dockerfile**. It’s a simple text file that acts as an executable blueprint.

```Dockerfile
FROM python:3.9          # Use Python as the base layer
RUN pip install numpy    # Install the required library
COPY bot.py /app/        # Copy your local code into the container
CMD ["python", "/app/bot.py"] # Tell the container what to run
```

Docker reads this file and automatically builds a perfect, repeatable container environment.

### 3. The Docker Image

Docker created a standardised way to compress a container's filesystem and settings into a single, shareable artefact called an **Image**. Because Docker made this format an industry standard, an image built on a laptop in Delhi will run exactly the same way on an AWS server in Virginia.

### 4. Docker Hub

Docker launched a massive cloud registry where people could upload and download these Images. Instead of spending two days compiling and configuring a PostgreSQL database from source code, you can now just type `docker pull postgres` and download a perfectly configured, containerised database in 5 seconds.

---
### The Analogy

- **The Container** is the audio file (the actual music you want to listen to).
- **The Linux Kernel** is the speaker (the hardware that actually plays the sound).
- **Docker** is Spotify (the beautiful interface that lets you search for, organize, play, and share the music without needing to understand audio codecs and frequencies).

Today, Docker is just one of many tools that can build and run containers. Alternatives like **Podman** or **containerd** do the exact same thing (interacting with namespaces and cgroups), but Docker remains the most famous because it was the one that made the technology mainstream.


## How Docker works on Mac and Windows 

When you run Docker Desktop on your Mac (which uses the XNU kernel) or Windows (which uses the NT kernel), you aren't actually running containers directly on those kernels.

Instead, Docker Desktop secretly starts a **very lightweight Linux Virtual Machine** in the background.

- **On Mac:** It uses the Apple Hypervisor Framework to run a tiny Linux distro.
- **On Windows:** It uses **WSL2** (Windows Subsystem for Linux), which is essentially a highly optimised Linux kernel running alongside Windows.

When you type `docker run`, your Mac/Windows terminal sends the command to that hidden Linux VM. The Linux kernel inside that VM provides the namespaces and cgroups, and the container runs there.