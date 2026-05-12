# Module 7 - Containers with Docker

**Demo Project:** Containerize a Node.js application with Docker and Docker Compose (MongoDB)

**Technologies used:** Docker, Docker Compose, Linux, Node.js, MongoDB

**Project Description:**

- Understand Docker containers and images, and how they compare to virtual machines
- Install Docker and use core CLI commands (run, pull, start, stop, logs, exec)
- Map container ports to the host and run multi-container stacks with Docker Compose
- Build images from a Dockerfile and integrate images into CI/CD workflows
- Authenticate to registries, tag images, and pull public or private images on a server
- Persist data with Docker volumes (host, anonymous, named) and use Docker best practices

*Module 7: Containers with Docker*

---

## Description

### What is Docker? What is a container?

**Docker** is an open-source **containerization** platform. It helps you **package applications with dependencies and configuration** into a **portable, standardized artifact** for development, shipping, and deployment. Containers existed before Docker; Docker made the workflow mainstream.

### Container registries

Container images are stored in **container registries**. Registries can be **public** (for example [Docker Hub](https://hub.docker.com/)) or **private** (used inside companies). You **pull** images to run them and **push** images you built so others or your servers can consume them.

### Development and deployment before and after containers

**Before containers:** installing and configuring apps on each machine was error-prone—many steps, dependency conflicts, and textual runbooks that teams interpreted differently.

**After containers:** the app ships in an **isolated environment** with its dependencies. You still need a **container runtime** on the server, but you avoid most per-server application configuration.

### Other container technologies

Docker is the most common container toolchain in many teams. Other runtimes and ecosystems exist; the ideas—images, registries, isolation—transfer broadly.

### Docker image vs Docker container

- **Image:** the **packaged artifact** (layered file system, metadata). It is **not** a running process. You move images between machines and registries.
- **Container:** a **running instance** created from an image: your process, virtual filesystem, and **port bindings** so traffic can reach the app inside.

Images **become** containers at **runtime** when you start them.

### Containers vs virtual machines

Both provide isolation, but differently:

- **VMs** virtualize **hardware** and include a **full guest OS** per VM—larger images, slower startup, strong OS flexibility (for example Linux guests on Windows hosts).
- **Containers** share the **host kernel** and virtualize at the **application layer**—typically **smaller** and **faster** to start than VMs.

**Compatibility note:** Linux containers target a Linux kernel. On **macOS** and **Windows**, **Docker Desktop** runs a Linux VM (or equivalent) so you can build and run Linux containers locally.

### Docker architecture (client, server, registry)

Docker bundles many concerns in one product:

- **Docker CLI** on your machine sends commands to the **Docker daemon** (engine).
- The engine **pulls** and **caches images**, **creates and runs containers**, manages **networks** and **volumes**, and more.
- **Registries** host images; `docker pull` / `docker push` interact with them.

If you only need a minimal **container runtime**, other tools exist; Docker also covers **image build** (`docker build` from a Dockerfile) and developer workflows.

## Prerequisites

- **Docker** installed locally: [Docker Desktop](https://docs.docker.com/desktop/) on macOS or Windows, or **Docker Engine** on Linux (see [Install Docker Engine](https://docs.docker.com/engine/install/)).
- **Node.js** (optional for this repo’s hybrid flow): the sample [js-app](./js-app/) can run the Express server on the host while databases run in containers—useful while learning.

**Example project:** [js-app](./js-app/) — Node.js + Express + MongoDB with `Dockerfile`, [docker-compose.yaml](./js-app/docker-compose.yaml), and [mongo.yaml](./js-app/mongo.yaml).

---

## Install Docker

Installation differs by OS; follow the official guide for your platform (handout: **Docker Desktop** on Mac and Windows, **native** Docker on Linux). Verify:

```bash
docker version
```

---

## Main Docker commands

These map to the handout’s core and debug commands:

| Command | Purpose |
|--------|---------|
| `docker run` | Create and start a container from an image |
| `docker pull` | Download an image from a registry |
| `docker start` / `docker stop` | Start or stop existing container(s) |
| `docker images` | List images stored locally |
| `docker ps` | List running containers; `docker ps -a` includes stopped |
| `docker logs` | View logs from a container |
| `docker exec -it` | Run a shell or command inside a running container (for debugging) |

Examples:

```bash
docker pull mongo:7
docker run -d --name mongodb -p 27017:27017 mongo:7
docker ps
docker logs mongodb
docker stop mongodb
```

---

## Port mapping

Multiple containers can listen on the same **container port**, but your **host** has one address per port. **Port mapping** maps **`hostPort:containerPort`** so traffic to the host reaches the correct container.

- **Container port:** the port the process listens on **inside** the container.
- **Host port:** the port on **your machine** (or server) you open to the outside.

Example: `-p 3001:3000` sends host `3001` to container `3000` (as in the handout’s `some-app://localhost:3001` style illustration).

---

## Workflow with Docker

A typical flow: **write a Dockerfile** → **`docker build`** to produce an image → **`docker run`** (or orchestration) on a laptop or server → optionally **`docker tag`** and **`docker push`** to a registry so other environments **pull** the same artifact.

---

## Docker Compose

**Docker Compose** defines and runs **multi-container** applications from a **YAML** file.

- Services, ports, environment variables, and dependencies stay **versioned** with the project—easier to maintain than long `docker run` lines.
- Compose creates a **default network** for services in the file so containers can resolve each other by **service name** (similar to `--net` patterns with user-defined networks).

This repo includes:

- [js-app/docker-compose.yaml](./js-app/docker-compose.yaml) — MongoDB, **named volume** `mongo-data`, and **mongo-express** (UI on host port **8081** per the file).
- [js-app/mongo.yaml](./js-app/mongo.yaml) — example including a tagged **custom app image**; replace the image reference with your own registry host and tag when you deploy.

Start the stack from `js-app`:

```bash
cd js-app
docker-compose -f docker-compose.yaml up -d
```

Docker Compose V2 also accepts `docker compose` (with a space) if your Docker CLI includes the plugin. Use `docker-compose down` (or `docker compose down`) to stop and remove containers (volumes persist unless you remove them explicitly).

---

## Dockerfile

A **Dockerfile** is a text file of instructions **`docker build`** uses to create an **image**. Each instruction tends to create a **layer**; layers are cached to speed rebuilds.

- **Build context:** the directory you pass at the end of `docker build` (often `.`). Only copy what you need; use a **`.dockerignore`** to exclude build artifacts and secrets.
- **Base image:** most Dockerfiles start `FROM` an existing image (for example `node:20-alpine` in [js-app/Dockerfile](./js-app/Dockerfile)).

Build the sample image from `js-app` (the trailing `.` is the context path):

```bash
cd js-app
docker build -t my-app:1.0 .
```

In CI/CD, the same Dockerfile usually builds the image artifact that is then **pushed** to a registry and **pulled** on servers or developer laptops.

---

## Private registries and image references

Working with a **private** registry (for example **AWS ECR**, **Azure Container Registry**, or a registry hosted on **Nexus**) usually follows:

1. **`docker login`** to the registry (authenticate).
2. **`docker tag`** so the image name includes **`registryDomain/imageName:tag`**.
3. **`docker push`** to upload the image.

**Default registry:** if you omit a registry host, Docker assumes **`docker.io`** (Docker Hub). For example:

```bash
docker pull mongo:4.2
```

is equivalent to pulling from Docker Hub’s library namespace. For a private registry you must include the host (and port if non-default), for example:

```text
165.245.222.56:8083/my-app:1.0
```

Replace with **your** registry DNS or IP and repository path.

---

## Deploy Docker containers on a remote server

On a deployment host you often **pull** a mix of:

- **Public** images (for example databases from Docker Hub).
- **Private** images (your application built in CI).

Ensure the server has Docker (or your orchestrator), **network access** to the registry, and correct **firewall** rules for published **host ports**.

---

## Docker volumes

By default, container filesystem data is **ephemeral** for application data patterns—**removing** a container can **delete** its writable layer. **Volumes** persist data **outside** the container’s lifecycle.

- **Host volume:** you choose the host path, e.g. `-v /home/mount/data:/var/lib/mysql/data`.
- **Anonymous volume:** Docker picks a directory under `/var/lib/docker/volumes/…`.
- **Named volume:** Docker manages storage under a **name** you choose—good for **production** and Compose (`volumes:` in YAML).

Named volume example from [js-app/docker-compose.yaml](./js-app/docker-compose.yaml):

```yaml
volumes:
  mongo-data:
    driver: local
```

mounted into MongoDB’s data path so database files survive container recreation.

---

## Docker best practices (security and maintainability)

From the handout—apply as your projects grow:

- Prefer **official or trusted** base images; pin **specific tags** (not only `latest`).
- Prefer **minimal** bases (for example **Alpine** variants) when appropriate to reduce attack surface and image size.
- Optimize **layer caching**; use **`.dockerignore`**; consider **multi-stage builds** for smaller final images.
- Run as a **non-root user** inside the image when possible; **scan** images for vulnerabilities; avoid baking **secrets** into images—use env vars or secret stores at runtime.

---

## Hands-on with the sample app

Detailed step-by-step commands (including optional `docker network create` and `docker run` variants) live in [js-app/README.md](./js-app/README.md).

Summary paths:

1. **Docker Compose:** from `js-app`, bring up MongoDB and mongo-express, create the `user-account` database and `users` collection in the UI, then run the Node app locally or build/run the app container once you have an image.
2. **Image build:** `docker build -t my-app:1.0 .` from `js-app` using [js-app/Dockerfile](./js-app/Dockerfile).
3. **Compose with a private app image:** adapt [js-app/mongo.yaml](./js-app/mongo.yaml) to point `my-app` at **your** registry image and ports.

If the embedded js-app README mentions a mongo-express URL port that differs from [docker-compose.yaml](./js-app/docker-compose.yaml), trust the **published host port in the YAML** (`8081` in the current file).

---

## References

- README structure guidance: [Make a README](https://www.makeareadme.com/)
- Docker documentation: [docs.docker.com](https://docs.docker.com/) (Get Docker, Engine, CLI, Compose, Dockerfile reference)
