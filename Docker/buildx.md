### Docker `buildx`

> **Docker Buildx** is an advanced build tool, and CLI plugin that extends `docker build` and uses **BuildKit** to build images **faster, smarter, and for multiple platforms**.

> **Buildx is a modern, powerful Docker build engine that enables fast, multi-platform, and optimized image builds.**

- Enables **multi-platform builds**
- Supports **advanced caching**
- Uses **parallel execution**
- Works with **BuildKit** under the hood

#### Why use Buildx?

> Multi-platform builds
> >Build images for different architectures at once

```shell
docker buildx build \
 --platform linux/amd64,linux/arm64 \
 -t myapp .
```

> Faster builds (parallel & cached)
> > Buildx reuses layers efficiently and runs steps in parallel.

> Advanced caching

```shell
--cache-from=type=registry,ref=myapp:cache
--cache-to=type=registry,ref=myapp:cache,mode=max
```

- local cache
- registry cache
- inline cache

> Better multi-stage builds
> > Buildx optimizes dependency reuse between stages.

> Output control

```shell
docker buildx build --output type=docker .
```

- Docker image
- OCI image
- Tar file
- Registry push

| **Enabling `buildkit`**

```shell
sudo nano /etc/docker/daemon.json
```

```json
{
	"features": {
		"buildkit": true
	}
}
```

```shell
sudo systemctl restart docker
```

> BuildKit is the engine that actually performs Docker builds, handling tasks like executing Dockerfile steps, caching layers, and optimizing performance, while Buildx is the user-facing tool that lets you interact with BuildKit to build images more efficiently, including support for multi-platform builds, advanced caching, and flexible output options; in short, BuildKit does the work, and Buildx gives you the controls to use it effectively.

> The traditional `docker build` uses the **classic Docker builder**, which is a **legacy, single-threaded build engine built directly into the Docker daemon**; it processes Dockerfile instructions **sequentially**, has limited caching, no native multi-platform support, and lacks advanced features like parallel execution or remote cache sharing - which is why it has largely been replaced by **BuildKit**, the modern engine that powers `buildx` and provides faster, smarter, and more flexible builds.


