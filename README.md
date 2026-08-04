# The Complete Docker Guide (Theory + Commands)

> Note: I rebuilt this guide from your uploaded PDF. A few commands in that PDF were formatted incorrectly (e.g. it showed `docker -t build myapp .` instead of the correct `docker build -t myapp .`). I've corrected all of them here and added proper explanations for each concept.

---

## Table of Contents
1. [What is Docker?](#1-what-is-docker)
2. [Docker Images](#2-docker-images)
3. [Docker Containers](#3-docker-containers)
4. [Docker Volumes](#4-docker-volumes)
5. [Docker Networks](#5-docker-networks)
6. [Dockerfile Reference](#6-dockerfile-reference)
7. [Full Dockerfile Example](#7-full-dockerfile-example)
8. [Docker Compose](#8-docker-compose)
9. [Docker Compose File Reference](#9-docker-compose-file-reference)
10. [Full Docker Compose Example (MERN stack)](#10-full-docker-compose-example-mern-stack)
11. [Latest Docker Features](#11-latest-docker-features)
12. [Docker on Windows — What's Different](#12-docker-on-windows--whats-different)
13. [Interview Prep (Windows-Focused)](#13-interview-prep-windows-focused)
14. [Quick Cheat Sheet (Windows/PowerShell)](#14-quick-cheat-sheet-windowspowershell)
15. [Command-by-Command Interview Q&A](#15-command-by-command-interview-qa)
16. [Docker & Docker Compose Info/System Commands](#16-docker--docker-compose-infosystem-commands)

---

## 1. What is Docker?

Docker is a tool that lets you package an application together with everything it needs to run (code, libraries, settings, runtime) into a single unit called a **container**.

Think of it like a lunchbox: instead of hoping the kitchen (your server) has the exact same ingredients as your home kitchen (your laptop), you pack the whole meal in a box. Wherever you open that box, the food is exactly the same.

### Key terms you must know before anything else:

| Term | Simple meaning |
|---|---|
| **Image** | A read-only blueprint/template for a container (like a recipe or a class in programming). |
| **Container** | A running (or stopped) instance of an image (like an object created from a class). |
| **Dockerfile** | A text file with instructions to build an image. |
| **Docker Hub** | An online registry where Docker images are stored and shared (like GitHub, but for images). |
| **Volume** | A way to store data outside the container so it survives even if the container is deleted. |
| **Network** | A way for containers to talk to each other and to the outside world. |
| **Docker Compose** | A tool to define and run multiple containers together using one YAML file. |

**Why use Docker?**
- "It works on my machine" problem disappears — the container carries its own environment.
- Easy to scale, deploy, and share apps.
- Lightweight compared to full virtual machines (containers share the host OS kernel).

### 1.1 How Docker Actually Works (Architecture)

Docker uses a **client-server architecture**:

| Component | What it does |
|---|---|
| **Docker Client (CLI)** | The `docker` command you type — sends instructions to the daemon. |
| **Docker Daemon (`dockerd`)** | The background service that does the real work — builds images, runs containers, manages networks/volumes. |
| **Docker Registry** | Where images are stored remotely (Docker Hub is the default public one; companies also run private registries). |
| **Docker Host** | The machine (or VM) where the daemon and containers actually run. |

**Flow:** You type `docker run nginx` in the client → the client talks to the daemon over a REST API → the daemon checks if the `nginx` image exists locally → if not, it pulls it from the registry → the daemon then creates and starts a container from that image.

**How isolation works under the hood (good interview knowledge):**
Docker containers aren't mini VMs — they're regular processes on the host, isolated using two Linux kernel features:
- **Namespaces** — give each container its own isolated view of things like process IDs, network interfaces, and filesystem mounts, so it "thinks" it's alone on the machine.
- **cgroups (control groups)** — limit and account for how much CPU, memory, and I/O each container can use, so one container can't starve the others.

This is *why* containers are so much lighter and faster than VMs — there's no second OS kernel being booted, just isolated processes sharing the host kernel.

### 1.2 Docker Engine vs Docker Desktop
- **Docker Engine** — the core runtime (`dockerd` + CLI + API). This is what actually runs on Linux servers.
- **Docker Desktop** — a full application (mainly for Windows/Mac) that bundles Docker Engine + a Linux VM (WSL2 on Windows) + a GUI + Kubernetes toggle + Docker Compose, all in one installer. This is what most people use for local development.

### 1.3 Installing Docker on Windows (Quick Overview)
1. Enable **WSL2**: open PowerShell as Administrator and run `wsl --install`.
2. Download and install **Docker Desktop for Windows** from docker.com.
3. During setup, make sure **"Use WSL2 based engine"** is selected (it's the default on modern versions).
4. Restart, open Docker Desktop, and verify it's running (whale icon in the system tray).
5. Confirm from a terminal:
```powershell
docker --version
docker compose version
```

### 1.4 Verifying Your Installation
```bash
docker run hello-world
```
**Explanation:** This is the standard "Docker is installed correctly" test. It pulls a tiny test image, runs it, prints a confirmation message, and exits — proving the client can talk to the daemon, pull from a registry, and run a container successfully.

---

## 2. Docker Images

An **image** is the packaged, immutable snapshot of your app. Every container starts from an image.

### 2.1 Build an image from a Dockerfile
```bash
docker build -t image_name path_to_dockerfile

# Example (build using Dockerfile in current directory, tag it "myapp")
docker build -t myapp .
```
**Explanation:** `-t` means "tag" — it gives your image a readable name (`myapp`) instead of a random ID. The `.` means "use the current folder as the build context" (i.e., Docker looks for a `Dockerfile` here).

### 2.2 List all local images
```bash
docker images
# or the newer equivalent:
docker image ls
```
**Explanation:** Shows every image sitting on your machine, along with its size and image ID.

### 2.3 Pull an image from Docker Hub
```bash
docker pull image_name:tag

# Example
docker pull nginx:latest
```
**Explanation:** Downloads a ready-made image from Docker Hub (or another registry) without needing to build it yourself. `:latest` is a "tag" — think of it as a version label.

### 2.4 Remove a local image
```bash
docker rmi image_name:tag
# Example
docker rmi myapp:latest

# Or, using the image ID:
docker rmi image_id
# Example
docker rmi fd484f19954f
```
**Explanation:** Deletes an image from your disk. You can't remove an image that's currently being used by a running container (you'd need to stop/remove that container first).

### 2.5 Tag an image
```bash
docker tag source_image:tag new_image:tag
# Example
docker tag myapp:latest myapp:v1
```
**Explanation:** Gives an existing image an additional name/version. Useful before pushing to a registry (e.g., tagging as `yourusername/myapp:v1`).

### 2.6 Push an image to Docker Hub
```bash
docker push image_name:tag
# Example
docker push myapp:v1
```
**Explanation:** Uploads your image to a registry so others (or your servers) can pull it. You need to be logged in (`docker login`) and the image name usually needs your Docker Hub username prefix, e.g. `docker push username/myapp:v1`.

### 2.7 Inspect details of an image
```bash
docker image inspect image_name:tag
# Example
docker image inspect myapp:v1
```
**Explanation:** Dumps detailed JSON metadata about the image — layers, environment variables, exposed ports, etc. Useful for debugging.

### 2.8 Save an image to a tar archive
```bash
docker save -o image_name.tar image_name:tag
# Example
docker save -o myapp.tar myapp:v1
```
**Explanation:** Exports the image into a `.tar` file — handy for moving images between machines without a registry (e.g., via USB drive or offline transfer).

### 2.9 Load an image from a tar archive
```bash
docker load -i image_name.tar
# Example
docker load -i myapp.tar
```
**Explanation:** The reverse of `save` — imports an image from a `.tar` file back into Docker.

### 2.10 Remove unused images
```bash
docker image prune
```
**Explanation:** Cleans up "dangling" images (old, untagged layers left over from rebuilds) to free disk space. Add `-a` to remove all unused images, not just dangling ones: `docker image prune -a`.

---

## 3. Docker Containers

A **container** is a running instance of an image — this is where your app actually executes.

### 3.1 Run a container from an image
```bash
docker run image_name
# Example
docker run myapp
```
**Explanation:** Creates and starts a new container from the image. Every `docker run` creates a brand-new container (not reusing an old one).

### 3.2 Run a named container from an image
```bash
docker run --name container_name image_name:tag
# Example
docker run --name my_container myapp:v1
```
**Explanation:** `--name` lets you give the container a friendly name instead of Docker's random auto-generated one (like `festive_curie`), so it's easier to refer to later.

### 3.3 List all running containers
```bash
docker ps
```
**Explanation:** Shows only containers that are currently running, along with their ID, image, status, and ports.

### 3.4 List all containers (including stopped ones)
```bash
docker ps -a
```
**Explanation:** Same as above, but also shows containers that have stopped/exited.

### 3.5 Stop a running container
```bash
docker stop container_name_or_id
# Example
docker stop my_container
```
**Explanation:** Gracefully stops a running container (sends a shutdown signal, waits a bit, then force-kills if needed).

### 3.6 Start a stopped container
```bash
docker start container_name_or_id
# Example
docker start my_container
```
**Explanation:** Resumes a previously stopped container — it doesn't create a new one, it just restarts the same container with its data intact.

### 3.7 Run container in interactive mode
```bash
docker run -it image_name
# Example
docker run -it ubuntu
```
**Explanation:** `-i` keeps input open (interactive), `-t` allocates a terminal. Together, `-it` lets you actually type commands inside the container like a normal terminal session.

### 3.8 Run container in interactive shell mode
```bash
docker run -it image_name sh
# Example
docker run -it ubuntu sh
```
**Explanation:** Same as above, but explicitly tells the container to start a shell (`sh` or `bash`) so you land directly at a command prompt inside the container. Useful for poking around and debugging.

> Tip: To open a shell inside an **already running** container, use:
> ```bash
> docker exec -it container_name_or_id sh
> ```

### 3.9 Remove a stopped container
```bash
docker rm container_name_or_id
# Example
docker rm my_container
```
**Explanation:** Deletes a container permanently (it must be stopped first, unless you force it — see next).

### 3.10 Remove a running container (forcefully)
```bash
docker rm -f container_name_or_id
# Example
docker rm -f my_container
```
**Explanation:** `-f` (force) stops AND removes the container in one step, even if it's currently running.

### 3.11 Inspect details of a container
```bash
docker inspect container_name_or_id
# Example
docker inspect my_container
```
**Explanation:** Returns detailed JSON info: IP address, mounted volumes, environment variables, restart policy, etc.

### 3.12 View container logs
```bash
docker logs container_name_or_id
# Example
docker logs my_container
```
**Explanation:** Shows the console output (stdout/stderr) of the container — essential for debugging why an app crashed. Add `-f` to "follow" logs in real time: `docker logs -f my_container`.

### 3.13 Pause a running container
```bash
docker pause container_name_or_id
# Example
docker pause my_container
```
**Explanation:** Freezes all processes inside the container (like hitting pause on a video) without stopping it. Useful for temporarily freeing CPU without losing state.

### 3.14 Unpause a paused container
```bash
docker unpause container_name_or_id
# Example
docker unpause my_container
```
**Explanation:** Resumes a paused container from exactly where it left off.

---

## 4. Docker Volumes

By default, any data written inside a container disappears when the container is deleted. **Volumes** solve this by storing data outside the container's writable layer, on the host machine, managed by Docker.

### 4.1 Create a named volume
```bash
docker volume create volume_name
# Example
docker volume create my_volume
```
**Explanation:** Creates a persistent storage area that containers can attach to.

### 4.2 List all volumes
```bash
docker volume ls
```
**Explanation:** Shows every volume Docker currently manages on your machine.

### 4.3 Inspect details of a volume
```bash
docker volume inspect volume_name
# Example
docker volume inspect my_volume
```
**Explanation:** Shows the actual filesystem path on the host where the volume's data lives, plus other metadata.

### 4.4 Remove a volume
```bash
docker volume rm volume_name
# Example
docker volume rm my_volume
```
**Explanation:** Permanently deletes the volume and its data. Can't remove a volume that's currently attached to a container.

### 4.5 Run a container with a volume (mount)
```bash
docker run --name container_name -v volume_name:/path/in/container image_name:tag
# Example
docker run --name my_container -v my_volume:/app/data myapp:v1
```
**Explanation:** `-v volume_name:/path/in/container` mounts the volume `my_volume` to the `/app/data` folder inside the container. Anything the app writes to `/app/data` is actually saved in the volume — safe even if the container is removed.

### 4.6 Copy files between a container and a volume/host
```bash
docker cp local_file_or_directory container_name:/path/in/container
# Example
docker cp data.txt my_container:/app/data
```
**Explanation:** Copies files in either direction between your machine and a container. You can also reverse it: `docker cp my_container:/app/data/data.txt .` to copy a file OUT of the container.

---

## 5. Docker Networks

Networks control how containers communicate — with each other and with the outside world.

### 5.1 Run a container with port mapping
```bash
docker run --name container_name -p host_port:container_port image_name
# Example
docker run --name my_container -p 8080:80 myapp
```
**Explanation:** `-p 8080:80` means "requests to port 8080 on my computer get forwarded to port 80 inside the container." This is how you access a containerized web server from your browser (e.g., `http://localhost:8080`).

### 5.2 List all networks
```bash
docker network ls
```
**Explanation:** Shows all networks Docker knows about, including the default ones (`bridge`, `host`, `none`).

### 5.3 Inspect details of a network
```bash
docker network inspect network_name
# Example
docker network inspect bridge
```
**Explanation:** Shows which containers are connected to a network, their IPs, and network configuration.

### 5.4 Create a user-defined bridge network
```bash
docker network create network_name
# Example
docker network create my_network
```
**Explanation:** Custom networks let containers find each other **by name** (Docker's built-in DNS) instead of needing to know IP addresses. This is the standard way multi-container apps talk to each other.

### 5.5 Connect a container to a network
```bash
docker network connect network_name container_name
# Example
docker network connect my_network my_container
```
**Explanation:** Attaches an already-running container to an additional network.

### 5.6 Disconnect a container from a network
```bash
docker network disconnect network_name container_name
# Example
docker network disconnect my_network my_container
```
**Explanation:** Removes a container from a network without stopping it.

---

## 6. Dockerfile Reference

A **Dockerfile** is a script with step-by-step instructions Docker follows to build an image. Each instruction creates a new "layer" in the image (Docker caches layers, so rebuilding is fast if nothing changed).

### `FROM` — base image
```dockerfile
FROM image_name:tag
# Example
FROM ubuntu:20.04
```
Every Dockerfile starts here. It tells Docker what to build on top of (e.g., a minimal Linux OS, or a language runtime like `node`, `python`).

### `WORKDIR` — working directory
```dockerfile
WORKDIR /path/to/directory
# Example
WORKDIR /app
```
Sets the folder inside the container where subsequent commands (`COPY`, `RUN`, `CMD`) will execute. Equivalent to `cd /app`.

### `COPY` — copy files in
```dockerfile
COPY host_source_path container_destination_path
# Example
COPY . .
```
Copies files from your project folder (on your computer) into the image. `COPY . .` means "copy everything in the current build folder into the current WORKDIR inside the image."

### `RUN` — execute a command during build
```dockerfile
RUN command1 && command2
# Example
RUN apt-get update && apt-get install -y curl
```
Runs shell commands **while building the image** (not while the container runs later) — typically used to install dependencies or packages.

### `ENV` — set environment variables
```dockerfile
ENV KEY=VALUE
# Example
ENV NODE_VERSION=14
```
Sets environment variables that are available both during the build and when the container runs.

### `EXPOSE` — document a port
```dockerfile
EXPOSE port
# Example
EXPOSE 8080
```
This is just documentation/metadata — it tells anyone reading the image "this app listens on port 8080." It does **not** actually publish the port to your host; you still need `-p` in `docker run` for that.

### `CMD` — default command to run
```dockerfile
CMD ["executable", "param1", "param2"]
# Example
CMD ["npm", "start"]
```
Specifies what command runs when a container starts, if you don't override it. There can only be **one** `CMD` in a Dockerfile (the last one wins if you write multiple). It CAN be overridden at `docker run` time.

### `ENTRYPOINT` — fixed main command
```dockerfile
ENTRYPOINT ["executable", "param1", "param2"]
# Example
ENTRYPOINT ["node", "app.js"]
```
Similar to `CMD`, but it's meant to be the **fixed** main process of the container — harder to override. `CMD` and `ENTRYPOINT` are often combined: `ENTRYPOINT` sets the program, `CMD` supplies default arguments to it.

### `ARG` — build-time variables
```dockerfile
ARG VARIABLE_NAME=default_value
# Example
ARG VERSION=latest
```
Lets you pass values at build time using `docker build --build-arg VERSION=1.2`. Unlike `ENV`, these are **not** available in the running container unless you also assign them to an `ENV`.

### `VOLUME` — declare a mount point
```dockerfile
VOLUME /path/to/volume
# Example
VOLUME /data
```
Tells Docker that this path should be treated as a mount point for external, persistent data.

### `LABEL` — add metadata
```dockerfile
LABEL key="value"
# Example
LABEL version="1.0" maintainer="Adrian"
```
Adds descriptive metadata to the image (author, version, etc.) — purely informational.

### `USER` — set the running user
```dockerfile
USER user_name
# Example
USER app
```
By default, containers run as `root`, which is a security risk. `USER` switches to a non-root user for running the app — a good practice for production images.

### `ADD` — copy + extra powers
```dockerfile
ADD source_path destination_path
# Example
ADD ./app.tar.gz /app
```
Like `COPY`, but can also auto-extract `.tar` archives and fetch remote URLs. **Best practice:** prefer `COPY` unless you specifically need `ADD`'s extraction/URL features — `COPY` is more predictable.

---

## 7. Full Dockerfile Example

```dockerfile
# Use an official Node.js runtime as a base image
FROM node:20-alpine

# Set the working directory to /app
WORKDIR /app

# Copy package.json and package-lock.json to the working directory
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the current directory contents to the container at /app
COPY . .

# Expose port 8080 to the outside world
EXPOSE 8080

# Define environment variable
ENV NODE_ENV=production

# Run app.js when the container launches
CMD ["node", "app.js"]
```

**Why `COPY package*.json ./` happens before `COPY . .`?**
This is a caching trick. Docker caches each layer. If you copy dependency files (`package.json`) and run `npm install` first, Docker only re-runs the slow `npm install` step when your dependencies actually change — not every time you edit your source code.

---

## 8. Docker Compose

### 8.1 What is Docker Compose?

**Docker Compose** is a tool that lets you define and run **multi-container** applications using a single YAML file (`docker-compose.yml`), instead of manually running a separate `docker run` command for every container.

### 8.2 Why Does It Exist? (The Problem It Solves)

Imagine a typical backend app: a FastAPI service + a Postgres database + Redis for caching. Without Compose, starting this stack by hand looks like:

```bash
docker network create app_network
docker volume create pg_data
docker run -d --name db --network app_network -v pg_data:/var/lib/postgresql/data -e POSTGRES_PASSWORD=pass postgres
docker run -d --name cache --network app_network redis
docker run -d --name api --network app_network -p 8000:8000 -e DB_HOST=db myapi
```

That's a lot to remember, type correctly, and repeat every single time — and it's easy to typo a flag or forget a step. Docker Compose replaces all of that with **one command**: `docker compose up`, driven by one readable config file that lives in your repo and can be version-controlled.

### 8.3 How Docker Compose Works (Under the Hood)

1. You describe your desired state in `docker-compose.yml` — what services exist, what images/Dockerfiles they use, what ports/volumes/env vars/networks they need.
2. When you run `docker compose up`, Compose reads that file and talks to the same Docker Engine/daemon you already have — it doesn't have its own separate runtime. It's essentially a smart wrapper that translates your YAML into the right sequence of `docker network create`, `docker volume create`, `docker build`, and `docker run` calls for you.
3. Compose automatically creates a **default network** for your project (unless you define your own), and puts every service on it — this is why services can reach each other using their **service name** as a hostname (e.g., `db`, `api`, `cache`) instead of hardcoded IPs.
4. Every resource Compose creates is **labeled** with your project name (usually your folder name), so `docker compose down` knows exactly what belongs to this project and can clean it all up safely.

### 8.4 Anatomy of a Minimal `docker-compose.yml`

```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
```

**Breaking this down line by line:**
- `version: '3.8'` — the Compose file format version being used.
- `services:` — the section listing every container you want.
- `web:` — the name of this service (also becomes its hostname on the network).
- `image: nginx:latest` — which image to run for this service.
- `ports:` — maps host port 8080 → container port 80.

Running `docker compose up` here does the exact same thing as `docker run -p 8080:80 nginx:latest`, just declared in a file instead of typed as a command.

### 8.5 Step-by-Step: Writing a Real Compose File From Scratch

Let's build up a Node.js API + MongoDB stack, one step at a time — this is a great flow to describe out loud in an interview.

**Step 1 — start with just the services block and the database:**
```yaml
version: '3.8'
services:
  db:
    image: mongo:latest
```

**Step 2 — add persistent storage, so data survives a restart:**
```yaml
    volumes:
      - db_data:/data/db
```

**Step 3 — add your own API service, built from a local Dockerfile:**
```yaml
  api:
    build: .
    ports:
      - "5000:5000"
```

**Step 4 — make sure the API waits for the database, and tell it how to reach it:**
```yaml
    depends_on:
      - db
    environment:
      - MONGO_URI=mongodb://db:27017/mydb
```
Notice `db` is used as the hostname — not `localhost`, not an IP — because Compose's internal DNS resolves service names automatically.

**Step 5 — declare the named volume used above:**
```yaml
volumes:
  db_data:
```

**Putting it all together:**
```yaml
version: '3.8'
services:
  db:
    image: mongo:latest
    volumes:
      - db_data:/data/db

  api:
    build: .
    ports:
      - "5000:5000"
    depends_on:
      - db
    environment:
      - MONGO_URI=mongodb://db:27017/mydb

volumes:
  db_data:
```

Run it with:
```bash
docker compose up -d
```

This single command builds the `api` image (if needed), creates the `db_data` volume, creates a default network, starts both containers on that network, and wires them together — replacing what would otherwise be 4-5 manual `docker` commands.

### 8.6 Docker Compose vs Plain `docker run` — Quick Comparison

| | Plain `docker run` | Docker Compose |
|---|---|---|
| Multi-container setup | Manual, repetitive commands | One declarative YAML file |
| Networking between containers | You manually create/attach networks | Automatic default network + DNS by service name |
| Reproducibility | Easy to forget a flag | Config is version-controlled, consistent every time |
| Scaling | Manual, one `run` per instance | `docker compose up --scale service=3` |
| Best for | Quick one-off containers, debugging | Real applications with multiple services |

### 8.7 Docker Compose Commands

#### 8.7.1 Start everything defined in `docker-compose.yml`
```bash
docker compose up
```
**Explanation:** Reads `docker-compose.yml` in the current folder and creates/starts all defined services. Add `-d` to run in the background (detached mode): `docker compose up -d`.

#### 8.7.2 Stop and remove everything
```bash
docker compose down
```
**Explanation:** Stops and removes the containers, networks (and optionally volumes, with `-v`) that Compose created.

#### 8.7.3 Build or rebuild services
```bash
docker compose build
```
**Explanation:** Builds (or rebuilds) the images for services that use a `build:` section, without starting containers.

#### 8.7.4 List containers for the current Compose project
```bash
docker compose ps
```
**Explanation:** Shows the status of containers belonging to this specific `docker-compose.yml` project.

#### 8.7.5 View logs for all services
```bash
docker compose logs
```
**Explanation:** Streams combined logs from every service. Add `-f` to follow live: `docker compose logs -f`.

#### 8.7.6 Scale a service to multiple containers
```bash
docker compose up -d --scale service_name=number_of_containers
# Example
docker compose up -d --scale web=3
```
**Explanation:** Runs multiple copies of the same service (e.g., 3 instances of your `web` service) — useful for basic load handling/testing.

#### 8.7.7 Run a one-time command in a service
```bash
docker compose run service_name command
# Example
docker compose run web npm install
```
**Explanation:** Spins up a temporary container from a service's image just to run one command (great for things like running database migrations or installing packages) — separate from the main running containers.

#### 8.7.8 List all volumes
```bash
docker volume ls
```
**Explanation:** Compose automatically creates named volumes for services that declare them; this shows all volumes on your system, including Compose-created ones.

#### 8.7.9 Pause / unpause a service
```bash
docker compose pause service_name
docker compose unpause service_name
```
**Explanation:** Freezes/resumes a specific service defined in your Compose file, same idea as `docker pause`/`docker unpause` but scoped to Compose.

#### 8.7.10 View details of a specific service
```bash
docker compose ps service_name
```
**Explanation:** Shows status info for just one service instead of all of them.

---

## 9. Docker Compose File Reference

The `docker-compose.yml` file is written in **YAML** (indentation matters — use spaces, not tabs).

### `version`
```yaml
version: '3.8'
```
Declares which Compose file format version you're using. (Note: in newer Compose versions this field is optional/deprecated, but it's still common and safe to include.)

### `services`
```yaml
services:
  web:
    image: nginx:latest
```
The core section — each entry under `services` becomes one container. Here, `web` is the service name, and it uses the `nginx:latest` image.

### `networks`
```yaml
networks:
  my_network:
    driver: bridge
```
Defines custom networks so services can communicate by name.

### `volumes`
```yaml
volumes:
  my_volume:
```
Declares named volumes that services can attach to for persistent storage.

### `environment`
```yaml
environment:
  - NODE_ENV=production
```
Sets environment variables inside that specific service's container.

### `ports`
```yaml
ports:
  - "8080:80"
```
Maps `host_port:container_port`, same idea as `docker run -p`.

### `depends_on`
```yaml
depends_on:
  - db
```
Ensures the `db` service's container starts before this service starts. Note: this only controls **start order**, not whether the dependency is actually "ready" (e.g., a database might still be initializing).

### `build`
```yaml
build:
  context: .
  dockerfile: Dockerfile.dev
```
Tells Compose to build an image from a Dockerfile instead of pulling a pre-built one. `context` is the folder Docker uses as the build source, `dockerfile` lets you point to a custom-named Dockerfile.

### `volumes_from`
```yaml
volumes_from:
  - service_name
```
Mounts all volumes from another service into this one (less common in modern Compose files — usually replaced by shared named volumes).

### `command`
```yaml
command: ["npm", "start"]
```
Overrides the default `CMD` from the image for this specific service.

---

## 10. Full Docker Compose Example (MERN stack)

This example spins up MongoDB, a Node.js/Express API, and a React client — all networked together.

```yaml
version: '3.8'

# Define services for the MERN stack
services:

  # MongoDB service
  mongo:
    image: mongo:latest
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: admin
    networks:
      - mern_network

  # Node.js (Express) API service
  api:
    build:
      context: ./api
      dockerfile: Dockerfile
    ports:
      - "5000:5000"
    # Ensure the MongoDB service is running before starting the API
    depends_on:
      - mongo
    environment:
      MONGO_URI: mongodb://admin:admin@mongo:27017/mydatabase
    networks:
      - mern_network

  # React client service
  client:
    build:
      context: ./client
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    # Ensure the API service is running before starting the client
    depends_on:
      - api
    networks:
      - mern_network

# Define named volumes for persistent data
volumes:
  mongo_data:

# Define a custom network for communication between services
networks:
  mern_network:
```

**How to read this:** three services (`mongo`, `api`, `client`) all join the same custom network `mern_network`, so they can reach each other using their **service name** as a hostname (notice `mongo:27017` inside the `api` service's `MONGO_URI` — it uses the service name, not `localhost`). `mongo_data` is a named volume so your database data survives even if you tear down and rebuild the containers.

Run it with:
```bash
docker compose up -d
```

---

## 11. Latest Docker Features

### `docker init`
```bash
docker init
```
**Explanation:** An interactive wizard that auto-generates a `Dockerfile`, `.dockerignore`, and `docker-compose.yml` for your project based on the language/framework it detects. Great starting point if you don't want to write these files from scratch.

### `docker compose watch`
```bash
docker compose watch
```
**Explanation:** A newer Compose feature for local development. It watches your source files, and when you change something, it automatically syncs the changed files into the running container or rebuilds/restarts the service — no more manually re-running `docker compose up --build` every time you save a file.

Example config inside `docker-compose.yml`:
```yaml
services:
  web:
    build: .
    develop:
      watch:
        - action: sync
          path: ./src
          target: /app/src
        - action: rebuild
          path: package.json
```

---

## 12. Docker on Windows — What's Different

Docker doesn't run natively on Windows the way it does on Linux (containers need a Linux kernel). On Windows, **Docker Desktop** handles this for you, and understanding *how* is a common interview topic.

### How Docker Desktop works on Windows
- Docker Desktop runs a lightweight Linux VM in the background using **WSL2** (Windows Subsystem for Linux 2) by default on modern setups.
- All your Linux containers actually run inside that WSL2 VM, but Docker Desktop makes it feel seamless from PowerShell, CMD, or a WSL terminal.
- The older alternative was **Hyper-V backend** — mostly replaced by WSL2 now because WSL2 is faster and uses less memory.
- **Interview one-liner:** "Docker Desktop on Windows uses a WSL2-based lightweight Linux VM to run the Docker engine, since containers need a Linux kernel."

### Where you run Docker commands
- **PowerShell** or **CMD** — works fine for almost everything in this guide.
- **WSL2 terminal (Ubuntu, etc.)** — recommended for anything involving Linux-style paths, bind mounts, or shell scripts, since it avoids path-translation issues.

### Path differences (a very common gotcha)
Windows paths use backslashes and drive letters; Docker (Linux-based) expects forward-slash paths.

```powershell
# PowerShell / CMD - bind mount syntax
docker run -v C:\Users\Sujit\project:/app myapp

# WSL2 terminal - bind mount syntax (Linux-style path)
docker run -v /mnt/c/Users/Sujit/project:/app myapp
```
**Explanation:** Docker Desktop auto-translates the Windows path in PowerShell/CMD, but if you're inside WSL2, you address the same Windows drive via `/mnt/c/...`.

### Line-ending issues (CRLF vs LF) — classic Windows Docker bug
Windows text editors often save files with `CRLF` line endings. If your Dockerfile copies a shell script (like `entrypoint.sh`) written on Windows, Linux containers can fail with an error like:
```
exec ./entrypoint.sh: no such file or directory
```
even though the file clearly exists. This happens because the invisible `\r` character breaks the shebang line (`#!/bin/sh`).

**Fix:** Configure your editor (VS Code) to save shell scripts with `LF` line endings, or add a `.gitattributes` rule:
```
*.sh text eol=lf
```
**Interview one-liner:** "It's a classic Windows-to-Linux container issue — CRLF line endings break shell scripts inside Linux containers; the fix is enforcing LF endings for script files."

### Resource limits
On Windows, Docker Desktop lets you cap CPU/RAM given to the WSL2 VM via **Settings → Resources**, or a `.wslconfig` file in your Windows user folder:
```ini
[wsl2]
memory=6GB
processors=4
```
Useful to mention if asked "how do you manage Docker's resource usage on your machine."

### File-sharing performance note
Bind-mounting a Windows-side folder (e.g. `C:\Users\...`) into a Linux container is noticeably slower for large projects (heavy file I/O like `node_modules`) compared to keeping your project files **inside the WSL2 filesystem** (e.g. `\\wsl$\Ubuntu\home\...` or working directly from a WSL2 terminal). This is a real performance tip interviewers sometimes probe for practical experience.

---

## 13. Interview Prep (Windows-Focused)

### Conceptual questions you should be able to answer fluently

**Q: What's the difference between a container and a virtual machine?**
A VM virtualizes an entire operating system (its own kernel) on top of a hypervisor, so it's heavy (GBs, slow boot). A container shares the host machine's OS kernel and only isolates the application layer (process, filesystem, network) — so it's lightweight (MBs, starts in seconds). On Windows specifically, since Windows doesn't have a Linux kernel, Docker Desktop runs a small Linux VM (via WSL2) so Linux containers have a kernel to share.

**Q: CMD vs ENTRYPOINT — when do you use which?**
- `CMD` provides *default* arguments — easily overridden by anything you pass after `docker run image_name`.
- `ENTRYPOINT` sets the *fixed* main process — harder to override (needs `--entrypoint` flag).
- Best practice: use `ENTRYPOINT` for the actual executable, and `CMD` for default arguments to it, so users can override just the arguments easily.
```dockerfile
ENTRYPOINT ["node"]
CMD ["app.js"]
# docker run myimage  → runs: node app.js
# docker run myimage server.js → runs: node server.js
```

**Q: COPY vs ADD — why does everyone say "just use COPY"?**
`ADD` does everything `COPY` does, plus auto-extracts tar archives and can fetch remote URLs. That "magic" behavior is unpredictable and a minor security concern (fetching URLs during build). `COPY` is explicit and predictable, so it's the recommended default; use `ADD` only when you specifically need archive extraction.

**Q: Named volumes vs bind mounts — what's the difference?**
- **Named volume** (`-v my_volume:/app/data`): Docker manages the storage location; portable, works the same on any OS, ideal for persistent data like databases.
- **Bind mount** (`-v C:\path\on\host:/app/data`): maps a specific folder from your host machine directly; great for local development (live code editing) but ties you to that exact host path — less portable, and (as noted above) slower on Windows if the folder is outside WSL2.

**Q: What are the three main Docker network drivers?**
- `bridge` — default; isolated private network on a single host, containers can talk via their name if on a user-defined bridge.
- `host` — container shares the host's network stack directly (no isolation, best performance, not available the same way on Docker Desktop for Windows since it's already inside a VM).
- `none` — no networking at all, fully isolated.

**Q: What is a multi-stage build and why use it?**
It lets you use multiple `FROM` statements in one Dockerfile — one stage to build/compile your app (with all the heavy build tools), and a final, slim stage that only copies the compiled output. This dramatically reduces final image size.
```dockerfile
# Stage 1: build
FROM node:20 AS builder
WORKDIR /app
COPY . .
RUN npm install && npm run build

# Stage 2: run (much smaller final image)
FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/server.js"]
```
**Interview one-liner:** "Multi-stage builds keep the final image lean by discarding build-time dependencies — great for cutting image size in production."

**Q: Why does Dockerfile instruction order matter?**
Docker caches each layer. If a layer's inputs haven't changed, Docker reuses the cache instead of re-running that step. So you put things that change *rarely* (like installing dependencies) **before** things that change *often* (like your source code), so you don't invalidate the cache — and trigger a slow reinstall — on every code edit.

**Q: Your container exits immediately after `docker run` — how do you debug it?**
1. Check logs: `docker logs container_name`
2. Check if the main process actually keeps running (containers exit when their main process/`CMD` finishes — e.g. a script that just prints and exits).
3. Run interactively to explore: `docker run -it image_name sh`
4. Check exit code: `docker ps -a` shows the exit code, which hints at the cause (e.g., `137` = killed for OOM).

**Q: Two containers can't talk to each other — what do you check?**
1. Are they on the **same Docker network**? (`docker network inspect network_name`)
2. Are you using the container/service **name** as the hostname, not `localhost`? (`localhost` inside a container refers to itself, not another container.)
3. Is the target port actually exposed/listening inside that container?

**Q: How would you containerize a Python FastAPI + Postgres backend (tie this to your own RAGCORE-style project)?**
A strong answer, framed around your own work: "I'd write a Dockerfile for the FastAPI service — start from a `python:3.11-slim` base, copy `requirements.txt` first and `pip install` before copying the rest of the code (for caching), then `CMD` to run the app with `uvicorn`. For Postgres, I'd use the official `postgres` image with a named volume for data persistence. I'd tie both together with a `docker-compose.yml`, put them on a shared custom network, and use `depends_on` plus the Postgres service name as the `DATABASE_URL` host instead of `localhost`. On Windows, I'd develop from inside WSL2 for better file-sync performance instead of bind-mounting straight from `C:\`."

**Q: What's `.dockerignore` and why does it matter?**
Like `.gitignore`, but for the Docker build context — it excludes files (like `node_modules`, `.git`, `.env`) from being sent to the Docker daemon during build. Speeds up builds and avoids accidentally baking secrets or bloat into the image.

**Q: How do you avoid leaking secrets (API keys, DB passwords) into an image?**
Never hardcode them in the Dockerfile with `ENV`. Instead, pass them at runtime via `docker run -e` or a `docker-compose.yml` `environment:`/`.env` file, or use Docker's `--secret` mechanism for build-time secrets (in BuildKit). Also keep `.env` files listed in `.dockerignore` so they're never copied into the image layer itself.

### Rapid-fire one-liners (good for quick verbal answers)
| Question | Answer in one line |
|---|---|
| Image vs Container? | Image = blueprint, Container = running instance of it |
| Why is Docker "lightweight" vs a VM? | Shares host OS kernel instead of virtualizing a whole OS |
| What does `docker compose down -v` do extra vs `down`? | Also removes the named volumes (deletes persisted data) |
| How does Docker Desktop run Linux containers on Windows? | Via a lightweight Linux VM using the WSL2 backend |
| What causes "exec format error" or script-not-found errors from Windows-authored scripts? | CRLF line endings breaking the shebang line |
| How do containers on the same custom network find each other? | Docker's built-in DNS resolves service/container names to IPs |

---

## 14. Quick Cheat Sheet (Windows/PowerShell)

All commands below work identically in **PowerShell**, **CMD**, and a **WSL2 terminal** — the only thing that changes is how you write host file paths in `-v`/bind mounts (see [Section 12](#12-docker-on-windows--whats-different)).

```powershell
# IMAGES
docker build -t name .            # build image
docker images                     # list images
docker pull name:tag              # download image
docker rmi name:tag               # delete image
docker tag old new                # rename/tag image
docker push name:tag              # upload image
docker image prune                # clean unused images

# CONTAINERS
docker run --name c1 -p 8080:80 image    # run + name + port map
docker ps                        # running containers
docker ps -a                     # all containers
docker stop c1                   # stop
docker start c1                  # start
docker rm c1                     # remove (stopped)
docker rm -f c1                  # force remove (running)
docker exec -it c1 sh            # shell into running container
docker logs -f c1                # follow logs

# VOLUMES
docker volume create v1
docker volume ls
docker volume rm v1
docker run -v v1:/app/data image

# Bind mount from Windows (PowerShell/CMD)
docker run -v C:\Users\Sujit\project:/app image

# Bind mount from WSL2 terminal
docker run -v /mnt/c/Users/Sujit/project:/app image

# NETWORKS
docker network create net1
docker network ls
docker network connect net1 c1

# COMPOSE
docker compose up -d
docker compose down
docker compose build
docker compose logs -f
docker compose ps
```

---

## 15. Command-by-Command Interview Q&A

One interview-style question and answer for **every single command** covered in this guide. Great for rapid-fire self-testing — cover the answer column and quiz yourself.

### Docker Images

| Command | Interview Question | Answer |
|---|---|---|
| `docker build -t name .` | Why do we use `-t` when building an image? | It tags the image with a human-readable name/version instead of leaving it identified only by a random hash ID. |
| `docker images` / `docker image ls` | How do you check which images already exist on your machine before pulling one again? | Run `docker images` (or `docker image ls`) to list all locally stored images and avoid unnecessary re-downloads. |
| `docker pull nginx:latest` | What's the risk of always pulling `:latest` in production? | `:latest` is a moving target — it can silently point to a different, potentially breaking version later; production setups should pin an explicit version tag. |
| `docker rmi myapp:latest` | Why might `docker rmi` fail with "image is being used by a container"? | Docker won't delete an image that a container (even a stopped one) still references — you must remove that container first, or force-remove it. |
| `docker tag myapp:latest myapp:v1` | Does `docker tag` create a new copy of the image? | No — it just adds another name/pointer to the same underlying image layers, so it doesn't use extra disk space. |
| `docker push myapp:v1` | What has to happen before `docker push` will succeed? | You must be authenticated (`docker login`) and the image name must be prefixed with your registry username/namespace, e.g. `username/myapp:v1`. |
| `docker image inspect myapp:v1` | How would you find out what environment variables are baked into an image without running it? | `docker image inspect myapp:v1` returns JSON metadata, including any `ENV` values set in the image. |
| `docker save -o myapp.tar myapp:v1` | How do you move a Docker image to an offline/air-gapped server with no registry access? | Export it with `docker save -o myapp.tar myapp:v1`, transfer the `.tar` file, then load it on the target machine. |
| `docker load -i myapp.tar` | What command restores an image from a `.tar` file created by `docker save`? | `docker load -i myapp.tar` — it re-registers the image in the local Docker image list. |
| `docker image prune` | Your disk is filling up with old Docker layers — what's the quickest fix? | `docker image prune` removes dangling (untagged, unused) images; add `-a` to remove all unused images, not just dangling ones. |

### Docker Containers

| Command | Interview Question | Answer |
|---|---|---|
| `docker run myapp` | Does `docker run` reuse an existing container if one already exists from that image? | No — every `docker run` creates a brand-new container instance; to reuse one, you'd use `docker start` on its name/ID instead. |
| `docker run --name my_container myapp:v1` | Why bother naming a container with `--name`? | Without it, Docker assigns a random auto-generated name, making it harder to reference the container later in scripts or commands. |
| `docker ps` | How do you see only the containers that are currently running? | `docker ps` — it lists just active/running containers by default. |
| `docker ps -a` | How do you check if a container crashed or exited earlier? | `docker ps -a` shows all containers including stopped/exited ones, along with their exit status. |
| `docker stop my_container` | What actually happens when you run `docker stop`? | Docker sends a graceful shutdown signal (SIGTERM) to the main process, waits a grace period, then force-kills (SIGKILL) if it hasn't stopped. |
| `docker start my_container` | What's the difference between `docker start` and `docker run` on an existing container? | `docker start` resumes an already-created (stopped) container with its previous state/data intact; `docker run` always creates a fresh new one. |
| `docker run -it ubuntu` | What do the `-i` and `-t` flags do individually? | `-i` keeps STDIN open (interactive), `-t` allocates a pseudo-terminal — together they let you type into and see output from the container like a normal shell. |
| `docker run -it ubuntu sh` | What's the difference between `docker run -it image sh` and `docker exec -it container sh`? | `run` creates a brand-new container and opens a shell in it; `exec` opens a shell inside an already-running container. |
| `docker rm my_container` | Why does `docker rm` sometimes fail immediately after `docker stop`? | It shouldn't if the container is fully stopped — but if it's still running (stop didn't finish), `rm` will refuse unless you force it. |
| `docker rm -f my_container` | How do you delete a container in one step without stopping it first? | `docker rm -f my_container` — the `-f` flag force-stops and removes it together. |
| `docker inspect my_container` | How would you find a running container's internal IP address? | `docker inspect my_container` returns JSON that includes the container's network settings, including its internal IP. |
| `docker logs my_container` | Your app seems to be failing silently inside a container — first debugging step? | Check `docker logs my_container` (add `-f` to follow live) to see the app's stdout/stderr output. |
| `docker pause my_container` | When would you use `docker pause` instead of `docker stop`? | When you want to temporarily free up CPU without losing the container's in-memory state — pause freezes processes rather than terminating them. |
| `docker unpause my_container` | How do you resume a paused container exactly where it left off? | `docker unpause my_container` — it resumes all frozen processes with no data loss. |

### Docker Volumes

| Command | Interview Question | Answer |
|---|---|---|
| `docker volume create my_volume` | Why not just store a container's data on its own writable layer? | Data in a container's writable layer is lost when the container is deleted; a volume stores it outside the container so it persists independently. |
| `docker volume ls` | How do you check which volumes currently exist on your system? | `docker volume ls` lists every volume Docker is managing. |
| `docker volume inspect my_volume` | Where does a named volume's data physically live on disk? | `docker volume inspect my_volume` shows the actual host filesystem path (mountpoint) Docker uses to store that volume's data. |
| `docker volume rm my_volume` | Why might `docker volume rm` fail? | It fails if the volume is still attached to any container (running or stopped) — you must detach/remove that container first. |
| `docker run -v my_volume:/app/data myapp:v1` | How do you make sure a database container's data survives a container restart or removal? | Mount a named volume to the database's data directory, e.g. `-v my_volume:/app/data`, so the data lives outside the container's lifecycle. |
| `docker cp data.txt my_container:/app/data` | How do you get a file into a container that's already running, without rebuilding the image? | `docker cp local_file container_name:/path/in/container` copies files directly into (or out of) a running container. |

### Docker Networks

| Command | Interview Question | Answer |
|---|---|---|
| `docker run --name my_container -p 8080:80 myapp` | What does `-p 8080:80` actually do? | It forwards traffic from port 8080 on the host machine to port 80 inside the container, making the containerized service reachable from outside. |
| `docker network ls` | How do you see what networks exist, including Docker's defaults? | `docker network ls` lists all networks — including the built-in `bridge`, `host`, and `none`, plus any custom ones you've created. |
| `docker network inspect bridge` | How would you find out which containers are attached to a given network and their IPs? | `docker network inspect network_name` returns details of all connected containers and their assigned addresses. |
| `docker network create my_network` | Why create a custom network instead of using the default `bridge`? | A user-defined bridge network gives you automatic DNS resolution by container/service name, which the default `bridge` network doesn't provide. |
| `docker network connect my_network my_container` | Can a running container be added to a second network without restarting it? | Yes — `docker network connect network_name container_name` attaches it live, no restart required. |
| `docker network disconnect my_network my_container` | How do you remove a container from a network without stopping it? | `docker network disconnect network_name container_name` detaches it while it keeps running. |

### Dockerfile Instructions

| Instruction | Interview Question | Answer |
|---|---|---|
| `FROM` | Can a Dockerfile have more than one `FROM`? | Yes — that's how multi-stage builds work; each `FROM` starts a new build stage, and you selectively copy artifacts between stages. |
| `WORKDIR` | Why use `WORKDIR` instead of `RUN cd /app`? | `RUN cd` only affects that single layer/command; `WORKDIR` persistently sets the directory for all subsequent instructions in the Dockerfile. |
| `COPY` | What's the safest way to copy your whole project into an image? | `COPY . .` after first copying and installing dependency files separately, so Docker's layer cache isn't invalidated on every source code change. |
| `RUN` | Does `RUN` execute when the container starts or when the image is built? | During the image build — it's baked into the image as a new layer, unlike `CMD`/`ENTRYPOINT` which run at container start. |
| `ENV` | How do `ENV` variables differ from `ARG` variables? | `ENV` values persist into the running container and are visible at runtime; `ARG` values only exist during the build unless explicitly copied into an `ENV`. |
| `EXPOSE` | Does `EXPOSE 8080` in a Dockerfile actually make the port accessible from your host machine? | No — `EXPOSE` is just documentation/metadata; you still need `-p host:container` at `docker run` time to actually publish the port. |
| `CMD` | Can `CMD` be overridden without rebuilding the image? | Yes — anything you pass after the image name in `docker run image_name your_command` overrides the Dockerfile's `CMD`. |
| `ENTRYPOINT` | Why combine `ENTRYPOINT` and `CMD` together in the same Dockerfile? | `ENTRYPOINT` fixes the main executable so it's not accidentally overridden, while `CMD` supplies default arguments that users can still easily override. |
| `ARG` | How do you pass a custom value into an `ARG` at build time? | `docker build --build-arg VERSION=1.2 .` — the value becomes available to that `ARG` during the build. |
| `VOLUME` | What does declaring `VOLUME /data` in a Dockerfile actually guarantee? | It tells Docker to treat that path as a mount point for external storage, but doesn't create a *named* volume automatically bound to a specific host location. |
| `LABEL` | What's the practical use of `LABEL` in a production image? | It attaches searchable metadata (version, maintainer, build info) that tools and teammates can query later, e.g. via `docker inspect`. |
| `USER` | Why should you add `USER app` near the end of a production Dockerfile? | Containers run as `root` by default, which is a security risk; switching to a non-root user limits the blast radius if the container is compromised. |
| `ADD` | When would you actually need `ADD` instead of `COPY`? | Only when you need its extra behavior — like auto-extracting a local `.tar` archive during the copy — otherwise `COPY` is the safer, more predictable default. |

### Docker Compose

| Command | Interview Question | Answer |
|---|---|---|
| `docker compose up` | What's the difference between `docker compose up` and `docker compose up -d`? | Without `-d`, it runs in the foreground and streams logs to your terminal; `-d` (detached) runs everything in the background and returns your terminal immediately. |
| `docker compose down` | What's the difference between `docker compose down` and `docker compose down -v`? | Plain `down` removes containers and networks but keeps named volumes (data persists); adding `-v` also deletes those volumes, wiping the data. |
| `docker compose build` | When would you run `docker compose build` separately instead of just `docker compose up`? | When you want to build/rebuild images (e.g. after a Dockerfile change) without immediately starting containers — useful in CI pipelines. |
| `docker compose ps` | How do you check the status of just the services defined in your current project, not every container on the machine? | `docker compose ps` scopes the listing to only the containers belonging to the current `docker-compose.yml` project. |
| `docker compose logs -f` | How do you tail live logs from all services in a multi-container app at once? | `docker compose logs -f` streams and follows combined logs from every service defined in the Compose file. |
| `docker compose up -d --scale web=3` | How would you quickly test load-balancing behavior with multiple instances of the same service? | `docker compose up -d --scale web=3` spins up three containers for the `web` service under the same Compose project. |
| `docker compose run web npm install` | What's the difference between `docker compose run` and `docker compose up` for a service? | `run` spins up a one-off, temporary container to execute a single command (separate from the main service containers); `up` starts the actual long-running service containers. |
| `docker volume ls` (Compose context) | Where do the volumes for a Compose project actually get listed? | Compose-created named volumes still show up in the regular `docker volume ls` output — Compose doesn't have a separate volume registry. |
| `docker compose pause` / `docker compose unpause` | How would you temporarily free host resources without tearing down your whole Compose stack? | `docker compose pause` freezes all (or a specified) service's containers in place; `docker compose unpause` resumes them with state intact. |
| `docker compose ps service_name` | How do you check the status of just one service instead of the whole stack? | `docker compose ps service_name` filters the output to that single service. |


Save this file, and use the Table of Contents to jump straight to whatever you need. If you want, I can also make you a **printable one-page cheat sheet PDF**, or a set of **flashcards** for quick revision before interviews (this could tie in nicely with your GenAI/backend job prep).

---

## 16. Docker & Docker Compose Info/System Commands

These are "meta" commands — they don't manage a specific image/container, they tell you about Docker itself, its overall state, or resource usage. Very commonly overlooked, but frequently asked in interviews as quick-fire questions.

### Docker Engine Info Commands

#### `docker version`
```bash
docker version
```
**Explanation:** Shows the Docker Client and Server (Engine) version numbers, API version, Go version, and build details. Useful for checking compatibility issues (e.g., a Compose file feature that needs a minimum Docker version).

#### `docker info`
```bash
docker info
```
**Explanation:** Gives a full system-level overview: number of containers (running/stopped/paused), number of images, storage driver, cgroup driver, total CPUs and memory allocated to Docker, and — on Windows — confirms whether you're running on the WSL2 backend. This is often the very first command engineers run when debugging "something's wrong with my Docker setup."

#### `docker system df`
```bash
docker system df
```
**Explanation:** Shows disk space used by images, containers, volumes, and build cache — like a `du -h` summary just for Docker. Great for answering "why is my disk full?"

#### `docker system prune`
```bash
docker system prune
# Remove everything unused, including volumes:
docker system prune -a --volumes
```
**Explanation:** A one-shot cleanup command — removes stopped containers, unused networks, dangling images, and build cache all at once. Adding `-a` also removes all unused images (not just dangling ones), and `--volumes` also removes unused volumes. **Use with caution** — this can delete data you actually wanted to keep.

#### `docker events`
```bash
docker events
```
**Explanation:** Streams real-time events from the Docker daemon (containers starting, stopping, images being pulled, etc.) — useful for monitoring/debugging what's happening on a busy host live.

#### `docker stats`
```bash
docker stats
```
**Explanation:** Shows a live, continuously updating table of CPU %, memory usage, network I/O, and disk I/O for all running containers — like `top`/Task Manager, but for containers. A very common interview follow-up to "how do you check if a container is hogging resources?"

#### `docker top container_name`
```bash
docker top my_container
```
**Explanation:** Lists the actual running processes *inside* a specific container, similar to running `ps` on it from the outside, without needing to `exec` into it.

#### `docker login` / `docker logout`
```bash
docker login
docker logout
```
**Explanation:** Authenticates (or signs out) your Docker CLI against a registry like Docker Hub — required before you can `docker push` private images.

---

### Docker Compose Info Commands

#### `docker compose version`
```bash
docker compose version
```
**Explanation:** Shows the installed Docker Compose version — important because syntax and available features (like `compose watch`) differ across versions.

#### `docker compose config`
```bash
docker compose config
```
**Explanation:** Validates your `docker-compose.yml` and prints the fully resolved configuration (with all variables substituted, defaults filled in, and any `extends`/override files merged). The best way to catch a YAML mistake **before** running `docker compose up`.

#### `docker compose top`
```bash
docker compose top
```
**Explanation:** Like `docker top`, but shows the running processes for every service/container in the current Compose project at once.

#### `docker compose events`
```bash
docker compose events
```
**Explanation:** Streams real-time events (start, stop, health-check changes, etc.) scoped to just the containers in the current Compose project.

#### `docker compose stop` / `docker compose start`
```bash
docker compose stop
docker compose start
```
**Explanation:** `stop` halts all service containers without removing them (unlike `down`, which also removes containers/networks); `start` brings previously-stopped Compose containers back up. Use this pair when you want to pause a project temporarily but keep everything (containers, networks) intact for a quick restart.

#### `docker compose restart`
```bash
docker compose restart
# or a single service:
docker compose restart api
```
**Explanation:** Quickly restarts service containers — handy after changing an environment variable file or config that the app only reads on startup, without a full `down`/`up` cycle.

#### `docker compose kill`
```bash
docker compose kill
```
**Explanation:** Force-stops containers immediately (SIGKILL) instead of the graceful shutdown `stop` attempts — useful when a service is hung and won't respond to a normal stop signal.

#### `docker compose rm`
```bash
docker compose rm
```
**Explanation:** Removes stopped service containers (without removing images, volumes, or networks) — a lighter-weight cleanup than `docker compose down`.

---

### Quick Interview Q&A for these commands

| Command | Interview Question | Answer |
|---|---|---|
| `docker version` | How do you check if your Docker Client and Server versions match? | `docker version` prints both separately — mismatches can cause compatibility warnings, especially in remote/Docker-in-Docker setups. |
| `docker info` | What's the first command you'd run to sanity-check a broken Docker installation? | `docker info` — it gives a full snapshot of the engine's state, resource limits, and backend (e.g., confirms WSL2 is active on Windows). |
| `docker system df` | How do you find out what's actually eating your disk space in Docker? | `docker system df` breaks down usage by images, containers, volumes, and build cache. |
| `docker system prune` | How do you clean up Docker in one command, and what's the risk? | `docker system prune -a --volumes` removes unused images, containers, networks, and volumes — but you can permanently lose data if a volume you needed wasn't actually "in use" at that moment. |
| `docker stats` | How do you check which running container is using the most memory right now? | `docker stats` gives a live, auto-refreshing table of CPU/memory/network usage per container. |
| `docker top my_container` | How do you see what processes are running inside a container without opening a shell in it? | `docker top container_name` lists the container's internal processes from the host. |
| `docker compose config` | How do you catch a YAML syntax or variable-substitution mistake before actually starting containers? | `docker compose config` validates and prints the fully resolved configuration without starting anything. |
| `docker compose stop` vs `docker compose down` | What's the difference between `stop` and `down` in Compose? | `stop` halts containers but keeps them (and networks) around for a fast restart; `down` removes the containers and networks entirely. |
| `docker compose restart` | How would you pick up a changed `.env` value without a full teardown? | `docker compose restart` (optionally scoped to one service) restarts the containers so they re-read startup config, without removing anything. |
| `docker compose kill` | When would you use `kill` instead of `stop` in Compose? | When a service is unresponsive and a graceful `stop` (SIGTERM) isn't working — `kill` sends SIGKILL immediately. |

---

### That's the full guide!
Save this file, and use the Table of Contents to jump straight to whatever you need. If you want, I can also make you a **printable one-page cheat sheet PDF**, or a set of **flashcards** for quick revision before interviews (this could tie in nicely with your GenAI/backend job prep).
