
There are **three main ways** to communicate with **`dockerd`**:

- **Unix socket** (`/var/run/docker.sock`) - the default and most common method on Linux

- **TCP socket** - remote access over the network (optionally secured with TLS)

- **Docker API (REST)** - used by the Docker CLI and other tools, over Unix or TCP sockets

---
#### Create a new container

```shell
sudo docker create <repository[:tag]>
```

---
#### Memory & Swap

```shell
docker run -m 400M --memory-swap 1G nginx
```

- **`-m 400M` / `--memory 400M`**  
	Limits the container to **400 MB of RAM**.

- **`--memory-swap 1G`**  
    Sets the **total memory + swap** limit to **1 GB**.  

	That means:    
		- RAM: **400 MB**
	    - Swap: up to **600 MB**


```shell
sudo docker run -m 400M nginx # Swap equals RAM (400M + 400M)
```

	Swap is **enabled automatically** and equals memory limit.

```shell
sudo docker --memory-swap 1G nginx # error
```

	You cannot set swap without setting RAM.

```shell
docker run -m 400M --memory-swap -1 nginx # Swap unlimited
```

	Container can swap as much as the host allows.

```shell
docker run -m 400M --memory-swap 400M nginx
```

	If RAM is exhausted → **OOM kill**

---

**OOM (Out Of Memory)** is a Linux kernel condition where the system runs out of available memory.

-  The kernel activates the **OOM Killer**
-  It forcefully **kills a process** (often a container’s main process) to free memory

**In short:**  
OOM = not enough memory → kernel kills processes to survive

---
#### CPU Management

```shell
lscpu | grep -i "^CPU(s):" # total number of CPU cores
```

```shell
docker run --cpuset-cpus="0,2" nginx
```

-  `--cpuset-cpus`
	restricts a container to run **only on specific CPU cores

---
#### Update Container Configuration

```shell
sudo docker update <options> <container-name | container-id>
```

```shell
sudo docker update --memory 1G nginx
```

```shell
sudo docker update --cpuset-cpus="1" nginx
```

---
#### Storage Management

```shell
docker run --storage-opt size=10G nginx
```

-  `--storage-opt`
	lets you **set storage-specific options** for a container, such as limiting how much disk space it can use (depending on the storage driver).

-  Limits the container’s writable layer to **10 GB**
-  Works with drivers like **overlay2** (on supported filesystems)

**`pquota`** is a **filesystem mount option** (mainly for **XFS**) that enables **project quotas**, which Docker uses to **enforce disk size limits** on containers.

-  Docker’s `--storage-opt size=...` needs **XFS with `pquota` enabled**
- `pquota` lets the filesystem **track and limit disk usage per container**
-  Without `pquota`, Docker **cannot enforce writable-layer size limits**

**Note**: if your system is using **ext4**, so **pquota is not enabled and cannot be enabled**.  To use Docker storage quotas, you need **XFS mounted with `pquota` / `prjquota`**. 

**Note**: On ext4, `--storage-opt size` is accepted but **not enforced**.

---
#### Enabling  `pquota`

```shell
mount | grep xfs # YOU NEED TO BE XFS
```

```shell
sudo nano /etc/default/grub
```

```shell
GRUB_CMDLINE_LINUX_DEFAULT="rootflags=uquota,pquota"
```

```shell
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

```shell
reboot
```

---
#### Container Statistics

```shell
sudo docker stats [<container> ...] # Display a live stream of container(s) resource usage statistics
```

```shell
sudo docker stats --no-stream # No stream
```

---
#### Create alias 

```shell
sudo docker tag <repository:tag> <new-repository:new-tag>
```

---
#### Dangling Image

A **dangling image** is a Docker image that **has no tag** and is **not referenced by any container**, usually left behind after rebuilding images.

```shell
sudo docker images ls --filter dangling=true
```

**Note**: You get **dangling images** when Docker creates new image layers and the old ones **lose their tags**.

```shell
sudo docker image prune # Deletes dangling images only
```

Useful options:

- `-a` → remove **all unused images** (not just dangling ones)
- `-f` → skip confirmation

---
#### Search Docker Registry for images

```shell
sudo docker search <query>
```

---
#### Docker manifest

A **Docker manifest** is a metadata file that describes **one or more image variants** under a single image name.

- One image name (e.g. `nginx:latest`)
- Multiple images for different **architectures or OS** (amd64, arm64, etc.)
- Docker pulls the **correct image automatically** for your system

---
#### Low-level information on Docker objects

```shell
sudo docker inspect <object>
```

---
#### Container Deletion

```shell
sudo docker rm <id/name>
```

	Removes a stopped container from the system.

```shell
sudo docker rm -f <id/name> # Force deletion
```

	Force-stops a running container and removes the container immediately.

```shell
sudo docker rm -f $(sudo docker ps -aq)
```

	Force-stops **all running containers** and removes **all containers** on the system.

```shell
sudo docker rm -v <id/name>
```

	Deleting the container with anonymous volumes attached to it.

---
#### Image Deletion

```shell
sudo docker rmi <id/name>
```
	Removes an image tag; if the image is **tagged**, only that tag is deleted (layers stay if still referenced), but if **any container exists from the image**, removal is blocked unless forced, because the image is in use.
	

```shell
sudo docker rmi -f <id/name>
```

	Force-removes the image **and automatically stops and deletes any running containers** that were created from that image.

**Note**: When a container is **running**, Docker **cannot delete the image layers**, so with `docker rmi -f` it **only removes the tag**, not the actual image data.  
The image becomes **untagged (dangling)**, but the **layers stay on disk** because the running container still depends on them.

**Note**: As long as the container exists, **it is not possible to delete its image.**

---
#### Docker Attach

```shell
sudo docker attach <container-id/name>
```

	Attaches the current terminal to a running container’s standard input, output, and error streams.

---
#### Docker Exec

```shell
sudo docker busybox echo hello
```

```shell
sudo docker -it busybox sh
```

- **`-i`** keeps standard input open
- **`-t`** allocates a pseudo-terminal (TTY)
- **`busybox`** is the container name (or ID) where the command is executed
- **`sh`** is the command run inside the container (bash)

```shell
sudo docker exec -w /bin 18a8f1034db0 ls
```

-  **`-w /bin`** → set the working directory inside the container to `/bin`

```shell
sudo docker exec -it -e ALPH=1 18a8f1034db0 sh
```

- **`-e FLAG=1`** sets an environment variable `FLAG` with value `1` for this process

```shell
sudo docker exec -it -e FLAG=1 -e NAME=busybox busybox sh
```

---
#### Copy files/folders

```shell
sudo docker cp <source> <destination>
```

```shell
sudo docker cp text.txt busybox:/home
```

```shell
sudo docker cp busybox:/home/pass.txt ./
```

```shell
sudo docker cp -a text.txt busybox:/home
```

-  `-a ` preserves the file’s **attributes** (permissions, ownership, timestamps, symlinks), so `text.txt` inside the container stays **exactly the same** as on the host instead of being copied with default metadata.

**Note**: In newer versions, using the `-a` flag does not make any difference in the result, and it behaves as `-a` by default.

---


