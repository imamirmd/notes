
**Note**: Linux Docker images can run on Windows using a Linux VM (WSL2 or Hyper-V), but Windows images require the Windows kernel and cannot run on Linux.

**Containers** run applications by sharing the host OS kernel, making them lightweight and fast, while **virtual machines** run full operating systems, making them heavier but more isolated.

---

Running **Docker on bare metal** uses the host’s hardware directly, giving better performance and lower overhead, while running **Docker inside a virtual machine** adds an extra layer of isolation but slightly reduces performance.

**Docker on bare metal**

-  Docker runs directly on the host OS, which talks straight to the hardware.
-  Containers share the **host kernel**, so there is **no extra virtualization layer**.
-  This gives **better performance**, **lower latency**, and **less resource overhead**.
-  Management is simpler, but containers are **closer to the host**, so isolation is weaker than a VM.

**Docker inside a virtual machine**

-  Docker runs inside a guest OS, which itself runs on a hypervisor.
-  There is **double isolation**: container → VM → host.
-  This improves **security and tenant isolation**, which is useful in cloud and multi-tenant setups.
-  Performance is slightly lower because hardware access goes through the VM layer, and resource usage is higher.

---
### Docker solves these main problems:

-  **Environment inconsistency**: *code runs the same on every system (“works on my machine” problem).*
-  **Dependency conflicts**: *each app has its own libraries and versions.*
-  **Slow deployment**: *apps can be started, stopped, and moved very quickly.*
-  **Scaling difficulty**: *containers are easy to replicate and scale.*
-  **Resource waste**: *containers are lightweight compared to virtual machines.*
-  **Complex setup**: *applications are packaged once and run anywhere.*

**In short:** Docker makes applications **portable, consistent, fast to deploy, and easy to scale**.

---

### Containers History

The idea of **containers** started in the early 2000s with Unix technologies like **chroot**, **FreeBSD Jails**, and later **Linux cgroups and namespaces**, which allowed process isolation on the same OS.

-  **1979 – chroot (Unix)**  

	Unix introduced `chroot`, which allowed a process to run with a limited view of the filesystem. 

	This was the first step toward isolation, but it only isolated files, not CPU, memory, or processes.

-  **2000 – FreeBSD Jails**  

	FreeBSD added **Jails**, which isolated filesystem, users, network, and processes.

	This was the first system that looked very similar to modern containers.

-  **2006–2008 – Linux cgroups and namespaces** 

	-  **Namespaces** isolate processes, users, networks, mounts
	-  **cgroups** limit and control CPU, memory, and I/O
	
	Together, these features made true OS-level containers possible.

-  **2008 – LXC (Linux Containers)**  

	LXC combined cgroups and namespaces into a usable container system.

	It worked, but was **complex and hard to manage**.

- **2013 – Docker**  (*Docker did **not invent containers**, but it*)

    -  Simplified container usage
    -  Added **images, layers, registries**
    -  Made containers **portable and developer-friendly**  

- **After Docker**

    -  **containerd**, **CRI-O** → container runtimes
	-  **Kubernetes** → container orchestration  

	**Containers became the standard for cloud and microservices.**

**Key insight:**  
Containers are **OS-level isolation**, not virtualization.  
Docker succeeded because it made existing kernel features **easy, repeatable, and practical**.

**In one line:**  
Containers evolved from simple filesystem isolation into a full process-isolation model, and Docker turned them into a global standard.

---

### LXC VS. Docker 

**LXC** and **Docker** both use Linux container technology, but they have **different goals**.

**LXC (Linux Containers)**

-  Focuses on running **full system containers** (like lightweight VMs).
-  You manage networking, storage, and lifecycle **manually**.
-  Feels closer to a traditional server or VM.
-  More control, but more complexity.

**Docker**

-  Focuses on running **single applications** in containers.
-  Strong tooling: images, layers, registries, CI/CD integration.
-  Easy to build, ship, and run apps.
-  Opinionated and developer-friendly.

**Note**: *Full system containers* mean containers that behave like a *complete operating system environment*, running multiple processes (init system, services, users) similar to a lightweight virtual machine.

**In short:**  
Use **LXC** when you want VM-like control.  
Use **Docker** when you want simple, portable app deployment.

---

### Docker Engine 

**Docker Engine** is the core software that builds, runs, and manages Docker containers on a system.

**Docker Engine** has three main components:

-  **Docker daemon (`dockerd`)** - runs in the background and manages containers, images, networks, and volumes

-  **Docker API** - lets tools and clients communicate with the daemon

-  **Docker CLI (`docker`)** - the command-line tool users interact with

Older Docker versions worked as a **single, tightly coupled system** where the **Docker daemon directly handled everything**.

-  `dockerd` directly managed **images, networking, storage, and containers**

-  It used **LXC at first**, and later its own runtime (`libcontainer`)

-  There was **no containerd separation** like today

-  All container lifecycle logic lived inside **dockerd**, making it large and less modular

**LXC (Linux Containers)** provides OS-level isolation by using Linux **namespaces** and **cgroups** to run processes in isolated environments that behave like lightweight virtual machines.

-  **Namespaces** isolate what a process can _see_ (like processes, network, filesystem).
-  **cgroups (control groups)** limit and control how much _resources_ (CPU, memory, I/O) a process can use.

**LXC provides APIs and tools** to manage containers, including a **C API**, a **command-line interface (`lxc` tools)**, and **language bindings** (like Python) to communicate with and control containers.

**libcontainer** is Docker’s low-level library that directly talks to the Linux kernel to create and manage containers using **namespaces and cgroups**, without relying on LXC.

**In short:**  
Old Docker = one big daemon doing everything  
New Docker = split into smaller components (dockerd + containerd + runc)

---
### Docker Engine Architecture

#### 1. **Docker CLI**

-  The command-line tool (`docker run`, `docker build`, etc.)
-  Sends user commands to Docker Engine
-  Does **not** run containers itself

#### 2. **Docker API**

-  A REST API used by the CLI and other tools
-  Communication happens over:
    - **Unix socket** (`/var/run/docker.sock`) on Linux
    - TCP (optional)

#### 3. **Docker Daemon (`dockerd`)**

-  The main brain of Docker
-  Responsibilities:
    -  Image management (pull, build, store)
    -  Network management
    -  Volume management
    -  High-level container lifecycle decisions
-  Delegates low-level container work to `containerd`

#### 4. **containerd**

-  A container runtime manager
-  Focuses only on **container lifecycle**, not Docker-specific features
-  Responsibilities:
    -  Create, start, stop, delete containers
    -  Manage container metadata
    -  Pull and unpack images
-  Talks to `dockerd` via API
-  Spawns a **shim** per container

#### 5. **containerd-shim**

-  A small helper process created **per container**
-  Responsibilities:
    -  Becomes the **parent process** of the container
    -  Keeps the container alive if `containerd` restarts
    -  Handles STDIN / STDOUT / STDERR
    -  Reports exit status back to containerd
-  Critical for container reliability

#### 6. **runc**

-  Low-level container runtime (OCI compliant)
-  Does the actual work of:
    -  Creating **namespaces**
    -  Applying **cgroups**
    -  Setting up the container filesystem
-  Starts the container’s main process
-  Exits after container startup

#### 7. **Linux Kernel**

-  The real executor
-  Provides:
    -  Namespaces (PID, network, mount, user, etc.)
    -  cgroups (CPU, memory, I/O limits)
    -  Security features (seccomp, AppArmor, SELinux)

---
### Unix Socket

-   A **Unix socket** is a special file that allows processes on the same system to communicate with each other efficiently without using network ports.

-  A **Unix socket in Docker** is a local communication file (like `/var/run/docker.sock`) that allows Docker clients and tools to talk to the Docker daemon securely on the same machine.

### Manage Docker as a non-root user

-  To manage Docker as a **non-root user**, add your user to the **docker group**, which allows access to the Docker Unix socket without using `sudo`.

```shell
sudo usermod -aG docker [<username> | $USER]
```

---

#### create and start a new container

```shell
# sudo docker run -it <repository>[:<tag>] [<command>]
sudo docker run -it centos:latest /bin/bash
```

- **`-i`** → keep input open (interactive)
- **`-t`** → allocate a terminal (TTY)

In a container, **PID 1 belongs to the container’s main process**.
-  In **Docker**, it’s usually the **command you run** (e.g. `/bin/bash`, `nginx`)
-  In **LXC/system containers**, it’s often an **init system** like `systemd`

```shell
sudo docker run -itd --name centos centos:latest /bin/bash
```

-  **`-d`** → runs a container in **detached mode** (or CTRL + PQ).
-  **`--name`** → sets a custom, easy-to-use name for a Docker container.

**Note**: CTRL + D causes to exit, and the container stops.

A **container ID** is a unique identifier that Docker assigns to each container.

```shell
sudo docker ps --no-trunc # Don't truncate output
```

---

### Live Restore

**Live Restore** is a Docker feature that keeps containers running even if the Docker daemon (`dockerd`) is restarted or upgraded.

```shell
sudo nano /etc/docker/daemon.json # dockerd config
```

```json
{
	"live-restore": true
}
```

```shell
systemctl reload docker
```

```shell
sudo docker info | grep -i "live restore" # must be true
```

**Live Restore works by decoupling container processes from the Docker daemon.**

-  Containers are actually run and supervised by **containerd + containerd-shim**, not directly by `dockerd`
-  When `dockerd` restarts, **shim keeps the container processes alive**
-  After restart, `dockerd` **reconnects to containerd** and regains control

Containers don’t die because their **parent process is the shim**, not `dockerd`.

**Note**: If **`containerd` dies**, **running containers keep running**, but **management is lost temporarily**. 

-  You **cannot start/stop/inspect containers**    
-  When `containerd` restarts, it **reconnects to existing shims**
-  Control is restored without killing containers

**Note**: Live Restore only controls what happens when **`dockerd` restarts** (`systemctl restart docker`)

**Note**: **Live Restore** only prevents Docker from **intentionally stopping containers during a graceful `dockerd` restart**.

**Note**: Daemon crashes (`dockerd` / `containerd`) don’t kill containers unless the **shim or host** dies.

If **Live Restore is OFF** and you **kill `dockerd` while a container is running**:

-  The **container keeps running** (it’s owned by the shim)
-  When `dockerd` starts again, it **does NOT reattach** to that running container
-  Docker treats the container as **stopped / unknown**
-  You usually must **stop and start it manually** to manage it again

A container’s processes are **normal Linux processes on the host**, just running inside **namespaces**.  So on the Docker host you can see them with tools like `ps` or `top`, but they appear **isolated inside the container** because namespaces hide other processes.

---

A **storage driver** is the Docker component that manages how image layers and container filesystems are stored, shared, and written on disk.

```shell
sudo nano /etc/docker/daemon.json # dockerd config
```

```json
{
	"storage-driver": "<driver>"
}
```

```shell
sudo docker info | grep -i "storage driver"
```

