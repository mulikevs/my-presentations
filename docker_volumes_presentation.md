
# Sharing Data & Ensuring Persistence in Docker Containers

## Slide 1: Introduction to Docker and Data Persistence

### 🐳 What is Docker?
- A lightweight tool for packaging apps and dependencies into containers.
- Ensures consistent environments across dev, test, and production.

### 💾 Why Data Persistence Matters
- By default Containers are **ephemeral** - container data is lost when it stops or is removed.
- Important for: databases, logs, app uploads, and configurations.

### 🎯 Objective
- Learn to persist and share data between containers.
- Understand volume commands and best practices.

---

## Slide 2: Docker Volumes Overview

### 📦 What Are Docker Volumes?
- Managed directories stored outside the container’s writable layer.
- Default location: `/var/lib/docker/volumes/`.

### ✅ Why Use Them?
- **Persistence**: Data outlives containers.
- **Sharing**: Allow multiple containers to use the same data.
- **Performance**: Optimized compared to host mounts.
- **CLI Management**: Easy to use and automate.

---

## Slide 3: Types of Docker Persistence

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

## Slide 4: Creating Docker Volumes

```bash
# Create a named volume
docker volume create mydata

# List volumes
docker volume ls

# Inspect volume
docker volume inspect mydata
```

---

## Slide 5: Attaching Volumes to Containers

### 📌 Syntax
```bash
docker run -v volume_name:/container/path image
```

### ✅ Example
```bash
docker run -it --name c1 -v mydata:/data busybox sh
echo "Hello" > /data/test.txt
```

- Volume `mydata` is mounted to `/data`.
- Data is preserved even if `c1` is removed.

---

## Slide 6: Sharing Volumes Across Containers

```bash
# Writer container
docker run -d --name writer -v mydata:/data busybox sh -c "echo Hello > /data/msg.txt; sleep 3600"

# Reader container
docker run -it --name reader -v mydata:/data alpine cat /data/msg.txt
```

### 🧠 Alternate: Inherit Volumes from Container
```bash
docker run -it --name reader2 --volumes-from writer alpine cat /data/msg.txt
```

---

## Slide 7: Using Bind Mounts

```bash
mkdir -p ~/docker-data

docker run -it -v ~/docker-data:/data busybox sh
```

- Local folder is mapped to container.
- Use for dev, **not recommended** for production (low portability).

---

## Slide 8: Temporary Data with tmpfs

```bash
docker run -it --tmpfs /tempdata alpine sh
```

- Creates an in-memory volume at `/tempdata`.

```bash
docker run -it --tmpfs /tempdata:size=50m alpine sh
```

- Limits to 50MB in RAM. Linux only.

---

## Slide 9: Docker Compose + Volumes

### 📄 docker-compose.yml
```yaml
version: "3.9"
services:
  app1:
    image: busybox
    command: sleep 3600
    volumes:
      - mydata:/data
  app2:
    image: alpine
    command: sh
    tty: true
    stdin_open: true
    volumes:
      - mydata:/data
volumes:
  mydata:
```

```bash
docker-compose up -d
docker exec -it <app2_container_id> sh
ls /data
```

---

## Slide 10: Volume Types Compared

| Type           | Example                          | Description                    |
|----------------|----------------------------------|--------------------------------|
| Named Volume   | `-v mydata:/data`                | Portable & reusable            |
| Anonymous Vol. | `-v /data`                       | Auto-created, container-tied   |
| Bind Mount     | `-v /host/path:/data`            | Host-linked, less portable     |

---

## Slide 11: Backup & Restore

### 🔄 Backup
```bash
docker run --rm   -v mydata:/volume   -v $(pwd):/backup   busybox tar czvf /backup/backup.tar.gz -C /volume .
```

### ♻️ Restore
```bash
docker run --rm   -v mydata:/volume   -v $(pwd):/backup   busybox tar xzvf /backup/backup.tar.gz -C /volume
```

---

## Slide 12: Managing Docker Volumes

```bash
docker volume ls               # List
docker volume inspect mydata  # Inspect
docker volume rm mydata       # Remove
docker volume prune           # Clean unused volumes
```

---

## Slide 13: Volume Capacity & Limits

- Depends on host disk or memory.
- Check storage:
  ```bash
  df -h /var/lib/docker
  df -i       # Inodes
  docker system df
  docker info --format '{{.Driver}}'
  ```

---

## Slide 14: Real Use Case - Jenkins

```bash
docker volume create jenkinsvol

docker run -d --name jenkins -p 8080:8080 -p 50000:50000   -v jenkinsvol:/var/jenkins_home jenkins/jenkins:lts
```

```bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

- Volume stores plugins, configs, jobs, etc.

---

## Slide 15: Best Practices

- ✅ Prefer **named volumes**.
- ❌ Avoid **bind mounts** in production.
- 🔐 Use **read-only** mounts if applicable.
- 🧼 Clean unused data: `docker volume prune`.
- 💾 **Back up** critical data frequently.

---

## Slide 16: Troubleshooting

- Add user to Docker group:
  ```bash
  sudo usermod -aG docker $USER && newgrp docker
  ```

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

## Slide 17: Summary

- **Volumes** are essential for persistent data.
- Share data across containers with `-v` or `--volumes-from`.
- Backup & restore with `tar`.
- Use Compose to orchestrate containers with shared volumes.

---

## Slide 18: Demos & Q&A

### 🎬 Demo Suggestions:
```bash
# Show volume data survives container deletion
docker rm container1
docker run -it -v mydata:/data alpine cat /data/test.txt

# Temporary vs Persistent
docker run -it alpine sh -c "echo temp a> /tmp/temp.txt"
docker run -it -v mydata:/data alpine sh -c "echo persist > /data/data.txt"
```

### 🤝 Contact
For more help, reach out to: **mulikevs**
