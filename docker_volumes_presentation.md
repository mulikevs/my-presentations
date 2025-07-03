
# Sharing Data & Ensuring Persistence in Docker Containers

## Introduction to Docker and Data Persistence

### 🐳 What is Docker?
- A lightweight tool for packaging apps and dependencies into containers.
- Ensures consistent environments across dev, test, and production.

### 💾 Why Data Persistence Matters
- By default Containers are **ephemeral** - container data is lost when it stops or is removed.
- Important for: logs, app uploads, and configurations.

### 🎯 Objective
- Learn to persist and share data between containers.
- Understand volume commands and best practices.

---

## Docker Volumes Overview

### 📦 What Are Docker Volumes?
- Managed directories stored outside the container’s writable layer.
- Default location: `/var/lib/docker/volumes/`.

### ✅ Why Use Them?
- **Persistence**: Data outlives containers.
- **Sharing**: Allow multiple containers to use the same data.
- **Performance**: Optimized compared to host mounts.
- **CLI Management**: Easy to use and automate.

---

##  Types of Docker Persistence

| Method         | Description                            | Best For                      |
|----------------|----------------------------------------|-------------------------------|
| **Volumes**    | Docker-managed persistent storage      | App data, DBs, shared configs |
| **Bind Mounts**| Link host path to container            | Local dev & testing           |
| **tmpfs**      | RAM-based, in-memory mount             | Temp files, secrets           |

- **Named Volumes**: Created and reused by name.
- **Anonymous Volumes**: Auto-generated, not reused.
- **Bind Mounts**: Tied to host directories.
- **tmpfs**: Lost on reboot or stop.

---

## Creating Docker Volumes

```bash
# Create a named volume
docker volume create data-tank

# List volumes
docker volume ls

# Inspect volume
docker volume inspect data-tank
```

---

## Attaching Volumes to Containers

### 📌 Syntax
```bash
docker run -v volume_name:/container/path image
```

### ✅ Example
```bash
docker run -it --name testlab -v data-tank:/data busybox sh
echo "Hello" > /data/test.txt
```

- Volume `data-tank` is mounted to `/data`.
- Data is preserved even if `testlab` is removed.

---

## Sharing Volumes Across Containers

```bash
# Writer container
docker run -it --name writer -v data-tank:/data busybox sh
echo "Hello World" > /data/msg.txt

# Reader container
docker run -it --name reader -v data-tank:/data alpine
cat /data/msg.txt
```

### 🧠 Alternate: Inherit Volumes from Container
```bash
docker run -it --name reader2 --volumes-from writer alpine
cat /data/msg.txt
```

---

## Using Bind Mounts

```bash
mkdir -p ~/docker-data

docker run -it --name test-sandbox -v ~/docker-data:/data busybox sh
```

- Local folder is mapped to container.
- Use for dev, **not recommended** for production (low portability).

---

## Temporary Data with tmpfs

```bash
docker run -it --name alpine-test --tmpfs /tempdata alpine sh
```

- Creates an in-memory volume at `/tempdata`.

```bash
docker run -it --name alpine-tmp-50m --tmpfs /tempdata:size=50m alpine sh
```

- Limits to 50MB in RAM. Linux only.

---

## Docker Compose + Volumes

### 📄 docker-compose.yml
```yaml
version: "3.7"
services:
  app1:
    image: busybox
    command: sh
    tty: true
    stdin_open: true
    volumes:
      - data-tank:/data
  app2:
    image: alpine
    command: sh
    tty: true
    stdin_open: true
    volumes:
      - data-tank:/data
volumes:
  data-tank:
```
## 🚀 How to Launch the Application
> Clone the repository and navigate into the project directory.
```bash
git clone https://github.com/mulikevs/my-presentations.git
cd my-presentations
docker compose build
docker compose up
docker compose ps        # Check running services
docker compose logs      # View logs later
docker exec -it <app2_container_id> sh
ls /data
docker-compose down
```

---

## Volume Types Compared

| Type           | Example                          | Description                    |
|----------------|----------------------------------|--------------------------------|
| Named Volume   | `-v data-tank:/data`             | Portable & reusable            |
| Anonymous Vol. | `-v /data`                       | Auto-created, container-tied   |
| Bind Mount     | `-v /host/path:/data`            | Host-linked, less portable     |

---


##  Managing Docker Volumes

```bash
docker volume ls                 # List
docker volume inspect data-tank  # Inspect
docker volume rm data-tank       # Remove
docker volume prune              # Clean unused volumes
```

---

##  Real Use Case - Jenkins

```bash
docker volume create jenkinsvol

docker run --name jenkins -p 8080:8080 -p 50000:50000   -v jenkinsvol:/var/jenkins_home jenkins/jenkins:lts
```

```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

- Volume stores plugins, configs, jobs, etc.

---

##  Troubleshooting

- Check permissions:
  ```bash
  docker volume inspect jenkinsvol
  sudo ls -l /var/lib/docker/volumes/jenkinsvol/_data
  ```

- Logs:
  ```bash
  docker logs container_name
  ```

- Disk full?
  ```bash
  df -h /var/lib/docker
  ```

---

##  Summary

- **Volumes** are essential for persistent data.
- Share data across containers with `-v` or `--volumes-from`.
- Use Compose to orchestrate containers with shared volumes.

---

### 🎬 Demo Suggestions:
```bash
# Show volume data survives container deletion
docker rm alpine-persist
docker run -it --name alpine-persist -v data-tank:/data alpine sh
cat /data/test.txt

# Temporary vs Persistent
docker rm alpine-temp
docker run -it --name alpine-temp alpine sh
echo 'temp data' > /tmp/temp.txt
docker rm alpine-temp
docker run -it --name alpine-temp-check alpine sh
cat /tmp/temp.txt # Inside the container, check for temporary file (will likely fail)
```

## 📘 About This Guide

This serves as a comprehensive guide to understanding volumes in Docker Containers.


---
> If you find this guide helpful, please consider giving the repository a ⭐️.

---

### 🤝 Author
**mulikevs**
