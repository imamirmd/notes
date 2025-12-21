
**DevOps** is a way of working where developers and operations teams work closely together to build, test, and release software faster and more reliably.

A **DevOps engineer** is a person who helps developers and operations teams work together by automating processes, managing systems, and making sure software is built, tested, and deployed smoothly and reliably.

An **artifact** is a file or result produced during software development, such as a build file, package, or compiled code, that is stored and used later for testing or deployment.

A **pipeline** is an automated sequence of steps that code goes through — such as build, test, and deploy — from the moment it is committed until it is released.

**Staging** is a testing environment that is very similar to production, where code is checked one last time before being released to real users.

**On-demand** means something happens **only when it is requested**, not automatically or continuously.

---
### CI/CD
-  **CI (Continuous Integration)** is a practice where developers regularly merge their code into a shared repository and automatically test it to catch problems early.
-  **CD (Continuous Delivery / Continuous Deployment)** is a practice where tested code is automatically prepared or released to production, so new changes can reach users quickly and safely.

**CI (Continuous Integration)** focuses on **testing code**.  
**CD (Continuous Delivery / Deployment)** focuses on **releasing code**.

- **CI**: Code is merged and **automatically built and tested** to find bugs early. (_check if the code works_)
- **CD**: After CI passes, code is **automatically delivered or deployed** to servers.  (_send the code to users_)

---
### Architecture 

-  **Monolithic architecture** is a software design where the entire application is built and deployed as one single, tightly connected unit.

-  **Microservice architecture** is a software design where an application is built as a collection of small, independent services that communicate with each other and can be developed, deployed, and scaled separately.

-  **Layered architecture** is a software design where an application is organized into separate layers, and each layer has a specific responsibility (such as presentation, business logic, and data).

---
**Bare metal** refers to running software directly on physical hardware without a virtualization layer, meaning the operating system runs straight on the server itself.

**Virtualization** is a technology that allows multiple virtual machines to run on a single physical server by sharing its hardware resources.

**Containerization** is a method of running applications in lightweight, isolated containers that package the app with everything it needs to run consistently across different environments.

**Bare metal**

- Software runs **directly on physical hardware**
- Very fast and powerful
- Harder to manage and scale

**Virtualization**

- Runs **multiple virtual machines**, each with its own OS
- Better isolation than bare metal
- Uses more resources because each VM has a full OS

**Containerization**

- Runs apps in **containers that share the same OS**
- Very lightweight and fast
- Easy to deploy, scale, and manage

**In short:**  
Bare metal = direct and powerful  
Virtualization = isolated but heavy  
Containerization = lightweight and efficient

---
**Docker** is a platform that lets you build, package, and run applications inside containers so they work the same way on any system.

-  A **Docker image** is a read-only template that contains an application and everything it needs to run, used to create Docker containers.

	-  The **application code**
	-  The **runtime** (for example, Python or Node.js)
	-  **Libraries and dependencies**
	-  **System tools** and required files
	-  **Configuration instructions**

-  A **Docker container** is a running instance of a Docker image that executes an application in an isolated environment on the host system.

	-  A **small writable layer** for runtime changes (To the upper layer of the image)

A **Docker host** is the machine (physical or virtual) that runs Docker and provides the resources needed to run Docker containers.

A **Docker registry** is a storage service where Docker images are saved, shared, and downloaded for use by Docker hosts.

A **Docker registry structure** is organized like this:

- **Registry** → the server that stores images
- **Repositories** → collections of related images (usually one per app)
- **Images** → specific builds inside a repository
- **Tags** → labels for image versions (like `latest`, `v1.0`)
- **Layers** → reusable filesystem layers that make up an image

**In short:**  
Registry → Repository → Image → Tag → Layers

---
#### Show the Docker version information

```shell
docker --version
```

```shell
sudo docker version
```
---
#### Display system-wide information

```shell
sudo docker info
```

---
#### Download an image from a registry

```shell
sudo docker pull <repository[:tag]>
```

-  **repository** *is the name of the image collection, usually representing an application or service (for example: `nginx`, `python`, `myapp/backend`).*

-  **Tag** *is the version or variant of the image inside the repository (for example: `latest`, `1.25`, `3.12-slim`).*

```shell
sudo docker pull nginx:latest
```

**Note**: Do not use ***latest tag*** in production. 

---
#### List images

```shell
sudo docker images
```

```shell
sudo docker images -a # Show all images (default hides intermediate images)
```

```shell
sudo docker images -q # Only show image IDs
```

```shell
sudo docker ps -n <int> # Show n last created containers (includes all states) (default -1)
```

```shell
sudo docker ps -l # Show the latest created container (includes all states)
```

```shell
sudo docker ps -s # Display total file sizes
```

```shell
sudo docker ps -a -f status=<status>
```

-  **created** - *The container is created but **has not started** yet.*
-  **running** - *The container’s main process is **currently running**.*
-  **paused** - *The container is temporarily **frozen** (processes are stopped using cgroups).*
-  **restarting** - *The container is **stopping and starting again** (usually due to a restart policy).*
-  **removing** - *The container is in the process of **being deleted**.*
-  **exited** - *The container has **stopped** because its main process ended or crashed.*
-  **dead** - *The container **failed to stop or be removed cleanly** and is in an inconsistent state.*

---
#### Create and run a new container (instance) from an image

```shell
sudo docker run <repository[:tag]>
```

---
#### Using different docker registry

**Note**: Docker Hub is the default registry.

```shell
sudo docker pull <docker-hub>/<repository[:tag]>
```

```shell
sudo docker run <docker-hub>/<repository[:tag]>
```
---
#### List containers

```shell
sudo docker ps
```

```shell
sudo docker ps -a # Show all containers (default shows just running)
```
---

A **layer ID** is a unique identifier (hash) that represents one specific filesystem layer in a Docker image.

- Each **layer** created from a Dockerfile instruction gets its own **layer ID**
- The ID is a **hash**, based on the layer’s content
- If two layers have the same content, they get the **same layer ID** and can be shared

Why it matters:

- Docker uses layer IDs to **reuse and cache layers**
- It avoids downloading or rebuilding identical layers
- Multiple images can **share the same layer** safely

---

**Container lifetime** is the period of time a container exists - from when it is created until it stops or is removed.

-  **Created** → *container is set up from an image*
-  **Running** → *the main process is active*
-  **Stopped** → *the process ends or is stopped*
-  **Removed** → *the container is deleted*

Key point:

-  A container lives **as long as its main process runs**
-  When that process stops, the container stops

Container lifetime depends on the **lifecycle of the main process inside it**
