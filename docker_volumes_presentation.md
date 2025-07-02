
# Sharing Data & Ensuring Persistence in Docker Containers

## 1. Why This Matters

- Containers are **temporary**—data disappears when they stop.
- You need **volumes** to keep your app data, logs, and configs alive.
- This guide will show you how to:
  - Create Docker volumes
  - Attach them to containers
  - Share data across containers
  - Backup & restore data

---

## 2. What Are Docker Volumes?

- Volumes are **special directories** managed by Docker.
- Stored outside the container's main file system.
- Benefits:
  - Persist data across container restarts
  - Share data between containers
  - Better performance than host directory mounts

---

## 3. Types of Data Storage

| Type        | Best For                      |
|-------------|-------------------------------|
| **Volumes** | Persistent, shared data       |
| **Bind Mounts** | Dev use (host file access)   |
| **tmpfs**   | Temporary, in-memory storage  |

---

## 4. Creating and Using Volumes

```bash
# Create a named volume
docker volume create mydata

# List volumes
docker volume ls

# Run a container with a volume
docker run -it -v mydata:/data busybox sh
```

- Anything written to `/data` stays even if the container is deleted.

---

## 5. Sharing Volumes Between Containers

```bash
# First container writes data
docker run -d --name writer -v mydata:/data busybox sh -c "echo hello > /data/file.txt; sleep 3600"

# Second container reads it
docker run -it --name reader -v mydata:/data alpine cat /data/file.txt
```

---

## 6. Bind Mounts (Advanced Use)

```bash
# Mount a local folder to the container
mkdir ~/mydata
docker run -it -v ~/mydata:/data busybox sh
```

- Good for **development** but not portable.

---

## 7. Temporary (In-Memory) Data with tmpfs

```bash
docker run -it --tmpfs /tempdata alpine sh
```

- Everything in `/tempdata` disappears after the container stops.

---

## 8. Docker Compose Example

```yaml
version: "3.9"
services:
  app:
    image: busybox
    command: sleep 3600
    volumes:
      - mydata:/data

volumes:
  mydata:
```

```bash
docker-compose up -d
```

---

## 9. Backup & Restore Volumes

```bash
# Backup
docker run --rm -v mydata:/volume -v $(pwd):/backup busybox tar czvf /backup/data.tar.gz -C /volume .

# Restore
docker run --rm -v mydata:/volume -v $(pwd):/backup busybox tar xzvf /backup/data.tar.gz -C /volume
```

---

## 10. Volume Management Tips

- List volumes: `docker volume ls`
- Inspect volume: `docker volume inspect mydata`
- Remove unused: `docker volume prune`
- Remove volume: `docker volume rm mydata`

---

## 11. Best Practices

✅ Use **named volumes** (`-v mydata:/data`)  
❌ Avoid bind mounts in production  
🔒 Use `:ro` for read-only mounts  
🗑️ Clean up unused volumes regularly  
💾 Backup critical data

---

## 12. Final Thoughts

- Volumes = **Data persistence** for your containers.
- Easy to use, powerful in practice.
- Explore further with `docker-compose` or cloud volume drivers.

---

## Questions or Demos?

Try:
- Deleting a container but keeping the volume.
- Comparing `/data` (persistent) vs. `/tmp` (ephemeral).
- Sharing files between containers via volumes.
