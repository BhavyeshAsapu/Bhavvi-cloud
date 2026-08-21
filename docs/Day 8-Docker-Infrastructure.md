# Day 8 — Docker Infrastructure & Containerization

## Phase 2 — Docker Infrastructure

### Week 3 — Containerization

## Day 8 Goals

The goal of Day 8 was to install Docker on the Raspberry Pi and learn the core Docker concepts required before deploying real services.

Topics covered:

- Install Docker
- Verify Docker installation
- Run the first container
- Learn Docker images
- Learn Docker containers
- Learn Docker volumes
- Learn Docker networks
- Learn Docker logs
- Learn Docker Compose
- Configure Docker to start automatically

---

# 1. System Baseline

The Raspberry Pi is running:

```text
OS: Debian GNU/Linux 13 (Trixie)
Architecture: arm64
Kernel architecture: aarch64
Virtualization: none
Root filesystem: /dev/sda2
Available root storage: approximately 436 GB
```

Architecture verification:

```bash
dpkg --print-architecture
```

Result:

```text
arm64
```

```bash
uname -m
```

Result:

```text
aarch64
```

The Raspberry Pi is running directly on hardware rather than inside a virtual machine:

```bash
sudo systemd-detect-virt
```

Result:

```text
none
```

The root filesystem has sufficient storage for Docker:

```text
/dev/sda2       459G total
approximately   436G available
```

---

# 2. Docker Before Installation

Docker was not installed initially.

```bash
docker --version
```

Result:

```text
docker: command not found
```

Docker Compose was also not installed initially because the Docker command itself was unavailable.

The Debian repository contained a `docker.io` package, but Docker's official repository was selected for the installation so that the official Docker Engine packages could be used.

---

# 3. Docker Repository Configuration

Docker's official APT repository was configured for Debian 13 (Trixie) and the Raspberry Pi's ARM64 architecture.

The Docker GPG key was installed under:

```text
/etc/apt/keyrings/docker.asc
```

The Docker repository was then added to the APT configuration.

After running:

```bash
sudo apt update
```

the Docker packages became available.

The packages selected for installation were:

```text
docker-ce
docker-ce-cli
containerd.io
docker-buildx-plugin
docker-compose-plugin
```

---

# 4. Docker Installation

Docker Engine and its supporting components were installed using APT.

Installation command:

```bash
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

After installation, the Docker daemon was checked:

```bash
sudo systemctl status docker --no-pager
```

Result:

```text
docker.service - Docker Application Container Engine
Active: active (running)
```

Docker was also configured to start automatically:

```bash
sudo systemctl is-enabled docker
```

Result:

```text
enabled
```

This means Docker will start automatically when the Raspberry Pi boots.

---

# 5. Docker Version Verification

Docker Engine version:

```bash
docker --version
```

Result:

```text
Docker version 29.7.2, build a7dcaa6
```

Docker Compose version:

```bash
docker compose version
```

Result:

```text
Docker Compose version v5.5.0
```

Therefore:

```text
Docker Engine       → Installed
Docker CLI          → Installed
Docker Compose      → Installed
Docker Buildx       → Installed
Docker daemon       → Running
Docker startup      → Enabled
```

---

# 6. First Docker Container

The first Docker container was tested using the official `hello-world` image.

Command:

```bash
sudo docker run hello-world
```

Docker automatically:

1. Contacted the Docker daemon.
2. Pulled the `hello-world` image from Docker Hub.
3. Created a container from the image.
4. Started the container.
5. Displayed the test message.
6. The container exited after completing its task.

The output confirmed:

```text
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

The container was identified as an ARM64 container:

```text
arm64v8
```

This confirmed that Docker was working correctly on the Raspberry Pi's ARM64 architecture.

---

# 7. Docker Images

Images were inspected using:

```bash
sudo docker images
```

The first image was:

```text
hello-world:latest
```

The image ID was:

```text
5dd0d3e6e255
```

An image is a packaged template used to create containers.

Conceptually:

```text
Docker Image
     |
     | docker run
     v
Docker Container
```

The image remains available even after the container created from it exits.

---

# 8. Docker Containers

Running containers were checked using:

```bash
sudo docker ps
```

Initially, no containers were running after the `hello-world` container finished.

All containers, including stopped containers, were checked using:

```bash
sudo docker ps -a
```

The `hello-world` container appeared with:

```text
STATUS: Exited (0)
```

Exit code `0` means the container's command completed successfully.

The container was created from:

```text
hello-world:latest
```

---

# 9. Interactive Alpine Container

An Alpine Linux container was created to learn the container lifecycle.

Command:

```bash
sudo docker run -d --name day8-alpine alpine:latest sleep infinity
```

The container was then checked:

```bash
sudo docker ps
```

The container was running with:

```text
NAME: day8-alpine
STATUS: Up
```

A shell was opened inside the container:

```bash
sudo docker exec -it day8-alpine sh
```

Inside the container:

```bash
cat /etc/os-release
```

Result:

```text
NAME="Alpine Linux"
ID=alpine
VERSION_ID=3.24.1
PRETTY_NAME="Alpine Linux v3.24"
```

The container hostname was also checked:

```bash
hostname
```

The hostname matched the container ID.

The shell was exited with:

```bash
exit
```

The container continued running because its main process was still:

```text
sleep infinity
```

The container was then stopped:

```bash
sudo docker stop day8-alpine
```

After stopping:

```bash
sudo docker ps
```

showed no running containers.

However:

```bash
sudo docker ps -a
```

still showed the stopped container.

The stopped container showed:

```text
Exited (137)
```

This occurred after the container was deliberately stopped. It does not indicate a Docker installation failure.

---

# 10. Docker Container Lifecycle

The following commands were practiced:

```bash
docker run
docker ps
docker ps -a
docker exec
docker stop
```

Basic lifecycle:

```text
Image
  |
  | docker run
  v
Container
  |
  ├── running
  |
  └── stopped/exited
```

A container can be stopped without deleting the container.

---

# 11. Docker Volumes

Docker volumes were introduced to demonstrate persistent data.

Initially:

```bash
sudo docker volume ls
```

returned no volumes.

A named volume was created:

```bash
sudo docker volume create day8-data
```

Result:

```text
day8-data
```

The volume was verified:

```bash
sudo docker volume ls
```

Result:

```text
local    day8-data
```

---

# 12. Inspecting a Docker Volume

The volume was inspected:

```bash
sudo docker volume inspect day8-data
```

Docker reported the mount point:

```text
/var/lib/docker/volumes/day8-data/_data
```

The volume driver was:

```text
local
```

The volume was managed by Docker.

The Docker-managed directory was not modified manually.

---

# 13. Docker Volume Persistence Test

A temporary container was created using the volume:

```bash
sudo docker run --rm \
  -v day8-data:/data \
  alpine:latest \
  sh -c 'echo "Docker volume persistence test" > /data/test.txt'
```

The container used:

```text
-v day8-data:/data
```

This mounted the Docker volume at:

```text
/data
```

inside the container.

The `--rm` option caused the temporary container to be automatically removed after the command completed.

The volume itself remained:

```bash
sudo docker volume ls
```

Result:

```text
local    day8-data
```

A completely new container was then used to read the file:

```bash
sudo docker run --rm \
  -v day8-data:/data \
  alpine:latest \
  cat /data/test.txt
```

Result:

```text
Docker volume persistence test
```

This demonstrated that the data survived the original container.

Conceptually:

```text
Container A
     |
     | writes test.txt
     v
Docker Volume
     |
     | container A removed
     |
     v
Container B
     |
     | reads test.txt
     v
Docker volume data still exists
```

This demonstrated the difference between container storage and persistent volume storage.

---

# 14. Docker Networks

Docker's default networks were inspected:

```bash
sudo docker network ls
```

The default networks were:

```text
bridge
host
none
```

The default bridge network was inspected:

```bash
sudo docker network inspect bridge
```

The default bridge used:

```text
Subnet: 172.17.0.0/16
Gateway: 172.17.0.1
```

---

# 15. Creating a User-Defined Docker Network

A custom bridge network was created:

```bash
sudo docker network create day8-network
```

The network was verified:

```bash
sudo docker network ls
```

It appeared as:

```text
day8-network
```

Two Alpine containers were then created on this network:

```bash
sudo docker run -d \
  --name network-test-1 \
  --network day8-network \
  alpine:latest \
  sleep infinity
```

```bash
sudo docker run -d \
  --name network-test-2 \
  --network day8-network \
  alpine:latest \
  sleep infinity
```

Both containers were running.

---

# 16. Container-to-Container Networking

A shell was opened inside the first container:

```bash
sudo docker exec -it network-test-1 sh
```

The second container was reached by its container name:

```bash
ping -c 3 network-test-2
```

Result:

```text
PING network-test-2 (172.18.0.3)
3 packets transmitted
3 packets received
0% packet loss
```

This demonstrated Docker's built-in DNS/service discovery on a user-defined network.

Conceptually:

```text
network-test-1
      |
      | Docker network
      | DNS name: network-test-2
      v
network-test-2
```

This is important for future multi-container applications because services can communicate using service/container names instead of hard-coded container IP addresses.

---

# 17. Docker Logs

Docker logs were tested using a continuously running Alpine container.

The container was created with:

```bash
sudo docker run -d \
  --name day8-logs \
  alpine:latest \
  sh -c 'while true; do echo "Docker log test $(date)"; sleep 5; done'
```

Logs were viewed using:

```bash
sudo docker logs day8-logs
```

Example output:

```text
Docker log test Fri Aug 21 08:52:58 UTC 2026
Docker log test Fri Aug 21 08:53:03 UTC 2026
```

Live logs were followed using:

```bash
sudo docker logs -f day8-logs
```

The output continued to appear every five seconds.

The live log stream was stopped with:

```text
Ctrl+C
```

The most recent five lines were viewed using:

```bash
sudo docker logs --tail 5 day8-logs
```

This demonstrated the basic Docker logging workflow.

The command:

```bash
sudo docker logs CONTAINER
```

was also tested literally and returned:

```text
No such container: CONTAINER
```

This was expected because `CONTAINER` was a placeholder rather than an actual container name.

---

# 18. Cleaning Temporary Docker Resources

The temporary network used for the networking exercise was removed.

The temporary containers used for the networking and logging exercises were also removed.

After cleanup:

```bash
sudo docker network ls
```

showed only the default networks:

```text
bridge
host
none
```

The temporary `day8-network` was no longer present.

The remaining test containers were:

```text
day8-alpine
hello-world
```

Both were stopped containers.

The persistent test volume was intentionally kept:

```text
day8-data
```

---

# 19. Docker Compose

Docker Compose was installed as part of:

```text
docker-compose-plugin
```

Compose version verification:

```bash
docker compose version
```

Result:

```text
Docker Compose version v5.5.0
```

Docker Compose is used to define and manage multiple related containers as an application stack.

A typical Compose architecture can look like:

```text
Docker Compose
      |
      +--- Frontend container
      |
      +--- Backend container
      |
      +--- Database container
      |
      +--- Redis container
      |
      +--- Application network
```

Compose allows the configuration of services, networks, volumes, ports, environment variables, and other container settings in a YAML file.

The planned test Compose project was to be stored under:

```text
/srv/docker/day8-compose
```

The Day 8 work focused on understanding Compose concepts and the installed Compose tool.

---

# 20. Docker Automatic Startup

Docker was configured to start automatically:

```bash
sudo systemctl is-enabled docker
```

Result:

```text
enabled
```

Docker service status:

```bash
sudo systemctl status docker --no-pager
```

Result:

```text
Active: active (running)
```

Therefore Docker is configured to start automatically when the Raspberry Pi boots.

---

# 21. Docker Concepts Learned

## Image

An image is a packaged template used to create containers.

Example:

```text
alpine:latest
hello-world:latest
```

## Container

A container is an instance created from an image.

Example:

```text
day8-alpine
day8-logs
network-test-1
network-test-2
```

## Volume

A volume provides persistent storage managed by Docker.

Example:

```text
day8-data
```

## Network

A network allows containers to communicate with each other.

Example:

```text
day8-network
```

## Logs

Docker logs provide access to the output generated by a container.

Example:

```bash
docker logs day8-logs
```

## Compose

Docker Compose allows multiple related services to be defined and managed together.

Example:

```text
compose.yaml
```

---

# 22. Important Docker Commands Learned

## Images

```bash
docker images
docker pull IMAGE
```

## Containers

```bash
docker run
docker ps
docker ps -a
docker exec
docker stop
docker rm
```

## Volumes

```bash
docker volume ls
docker volume create
docker volume inspect
```

## Networks

```bash
docker network ls
docker network inspect
docker network create
docker network rm
```

## Logs

```bash
docker logs CONTAINER
docker logs -f CONTAINER
docker logs --tail N CONTAINER
```

## Compose

```bash
docker compose version
docker compose up -d
docker compose ps
docker compose logs
docker compose down
```

---

# 23. Day 8 Checklist

```markdown
### Docker

- [x] Install Docker
- [x] Verify Docker installation
- [x] Run first container
- [x] Learn Docker images
- [x] Learn Docker containers
- [x] Learn Docker volumes
- [x] Learn Docker networks
- [x] Learn Docker logs
- [x] Learn Docker Compose
- [x] Configure Docker to start automatically
```

---

# 24. Final Day 8 Status

Docker infrastructure is successfully installed on the Raspberry Pi.

Verified components:

```text
Docker Engine       → 29.7.2
Docker Compose      → 5.5.0
Architecture        → ARM64
Docker daemon       → Running
Docker startup      → Enabled
First container     → Successful
Images              → Understood
Containers          → Understood
Volumes             → Tested
Networks             → Tested
Container logs      → Tested
Compose             → Installed and understood
```

The Raspberry Pi is now ready for the next stage of Docker infrastructure work and eventually real application deployments.

# Day 8 Result

**Docker installation and core containerization fundamentals completed successfully.**
