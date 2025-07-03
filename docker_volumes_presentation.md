
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
docker run -it --tmpfs /tempdata alpine sh
```

- Creates an in-memory volume at `/tempdata`.

```bash
docker run -it --tmpfs /tempdata:size=50m alpine sh
```

- Limits to 50MB in RAM. Linux only.

---

## Docker Compose + Volumes

### 📄 docker-compose.yml
```yaml
version: "3.9"
services:
  app1:
    image: busybox
    command: sleep 3600
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

```bash
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
| Named Volume   | `-v mydata:/data`                | Portable & reusable            |
| Anonymous Vol. | `-v /data`                       | Auto-created, container-tied   |
| Bind Mount     | `-v /host/path:/data`            | Host-linked, less portable     |

---

##  Backup & Restore

### 🔄 Backup
```bash
docker run --rm   -v mydata:/volume   -v $(pwd):/backup   busybox tar czvf /backup/backup.tar.gz -C /volume .
```

### ♻️ Restore
```bash
docker run --rm   -v mydata:/volume   -v $(pwd):/backup   busybox tar xzvf /backup/backup.tar.gz -C /volume
```

---

##  Managing Docker Volumes

```bash
docker volume ls               # List
docker volume inspect mydata  # Inspect
docker volume rm mydata       # Remove
docker volume prune           # Clean unused volumes
```

---

## Volume Capacity & Limits

- Depends on host disk or memory.
- Check storage:
  ```bash
  df -h /var/lib/docker
  df -i       # Inodes
  docker system df
  docker info --format '{{.Driver}}'
  ```

---

##  Real Use Case - Jenkins

```bash
docker volume create jenkinsvol

docker run -d --name jenkins -p 8080:8080 -p 50000:50000   -v jenkinsvol:/var/jenkins_home jenkins/jenkins:lts
```

```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

- Volume stores plugins, configs, jobs, etc.

---

## Best Practices

- ✅ Prefer **named volumes**.
- ❌ Avoid **bind mounts** in production.
- 🔐 Use **read-only** mounts if applicable.
- 🧼 Clean unused data: `docker volume prune`.
- 💾 **Back up** critical data frequently.

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
- Backup & restore with `tar`.
- Use Compose to orchestrate containers with shared volumes.

---

### 🎬 Demo Suggestions:
```bash
# Show volume data survives container deletion
docker rm container1
docker run -it -v mydata:/data alpine cat /data/test.txt

# Temporary vs Persistent
docker run -it alpine sh -c "echo temp a> /tmp/temp.txt"
docker run -it -v mydata:/data alpine sh -c "echo persist > /data/data.txt"
```

### 🤝 
**mulikevs**
