Sharing Data in Docker Containers and Ensuring Data Persistence
Slide 1: Introduction to Docker Volumes

What are Docker Volumes?
Storage mechanisms to persist and share data in Docker containers.
Allow data to exist outside the container’s ephemeral filesystem.


Why Use Volumes?
Containers are stateless by default; data is lost when containers are removed.
Volumes enable persistence, sharing, and management of data.


Key Benefits:
Data persists across container lifecycles.
Share data between containers or with the host.
Improve performance and portability.




Slide 2: Types of Docker Volumes

1. Named Volumes:
Docker-managed, stored in /var/lib/docker/volumes/.
Reusable across containers, explicitly named.
Example: my-volume.


2. Anonymous Volumes:
Docker-managed, no specific name (random ID).
Tied to a single container’s lifecycle unless reused.


3. Bind Mounts:
Map a host directory/file to a container path.
Direct control over host storage.


4. tmpfs Mounts (Linux only):
In-memory storage, non-persistent.
For sensitive data that shouldn’t be saved to disk.




Slide 3: Creating Docker Volumes

Command to Create a Named Volume:docker volume create my-volume


Creates a volume named my-volume.


Verify Volume Creation:docker volume ls


Output:DRIVER    VOLUME NAME
local     my-volume




Note:
Named volumes are created automatically when used in docker run if they don’t exist.
Anonymous volumes are created automatically when no volume name is specified.




Slide 4: Attaching Volumes to Containers

Syntax for docker run:docker run [options] -v volume_name:/container/path image [command]


-v volume_name:/container/path: Mounts volume_name to /container/path.


Example: Named Volume:docker run -it --name my-container -v my-volume:/data alpine sh


Mounts my-volume to /data in the container.


Example: Bind Mount:mkdir -p /home/user/my-data
docker run -it --name my-container -v /home/user/my-data:/data alpine sh


Mounts host directory /home/user/my-data to /data.


Example: Anonymous Volume:docker run -it --name my-container -v /data alpine sh


Creates and mounts an anonymous volume to /data.




Slide 5: Testing Data Persistence

Write Data to a Volume:Inside the container:echo "Persistent data" > /data/test.txt
ls /data


Exit the container: exit.


Verify Persistence:Run a new container with the same volume:docker run -it --rm -v my-volume:/data alpine sh

Check data:cat /data/test.txt


Output: Persistent data.


Key Point:
Named and bind-mounted volumes persist data even after containers are removed.
Anonymous volumes persist until explicitly removed or no longer referenced.




Slide 6: Sharing Data Between Containers

Using Named Volumes:
Multiple containers can mount the same named volume.
Example:docker run -d --name container1 -v my-volume:/data alpine sleep 3600
docker run -it --name container2 -v my-volume:/data alpine sh


container2 can access/modify data in /data written by container1.




Use Case:
Share configuration, logs, or application data (e.g., database files, Jenkins jobs).


Bind Mounts for Sharing:
Containers can share a host directory:docker run -d --name container1 -v /home/user/shared:/data alpine sleep 3600
docker run -it --name container2 -v /home/user/shared:/data alpine sh






Slide 7: Managing Docker Volumes

List Volumes:docker volume ls


Shows all volumes (named and anonymous).


Inspect a Volume:docker volume inspect my-volume


Displays details like mount point (e.g., /var/lib/docker/volumes/my-volume/_data).


Remove a Volume:docker volume rm my-volume


Caution: Deletes all data in the volume.
Cannot remove volumes in use by containers.


Clean Up Unused Volumes:docker volume prune


Removes all anonymous volumes not attached to any container.




Slide 8: tmpfs Mounts for Non-Persistent Data

What is tmpfs?:
In-memory storage, lost when the container stops or host reboots.
Useful for temporary or sensitive data (e.g., secrets, caches).


Command:docker run -it --name my-container --tmpfs /tmpdata alpine sh


Mounts a tmpfs volume at /tmpdata.


Optional Size Limit:docker run -it --name my-container --tmpfs /tmpdata:size=100m alpine sh


Limits tmpfs to 100 MB.


Note:
Only available on Linux hosts.
Limited by host RAM/swap.




Slide 9: Volume Capacity and Limits

No Explicit Docker Limit:
Volume size is constrained by the host filesystem (e.g., ext4: up to 1 exabyte).
Bind mounts depend on host directory disk space.
tmpfs depends on host RAM/swap.


Check Host Disk Space:df -h /var/lib/docker


Storage Drivers:
Docker uses drivers like overlay2, btrfs, or zfs.
Capacity depends on the driver and host configuration.


Inode Limits:
Filesystems have inode limits (max number of files).
Check with:df -i






Slide 10: Best Practices for Volumes

Use Named Volumes for Persistence:
More portable than bind mounts.
Example: Use my-volume instead of /home/user/data.


Avoid Bind Mounts for Portability:
Bind mounts tie containers to specific host paths.


Backup Volumes:
Copy volume data to another volume or host:docker run --rm -v my-volume:/data -v /backup:/backup alpine cp -r /data /backup




Clean Up Regularly:
Remove unused volumes to save space:docker volume prune




Security:
Avoid mounting sensitive host directories (e.g., /etc).
Use read-only mounts when possible:docker run -it -v my-volume:/data:ro alpine sh






Slide 11: Example: Jenkins with Persistent Volume

Scenario:
Run Jenkins with a volume to persist job configurations.


Create Volume:docker volume create jenkinsvol


Run Jenkins Container:docker run -d --name jenkins -p 8080:8080 -v jenkinsvol:/var/jenkins_home jenkins/jenkins:lts


Access Jenkins:
Open http://localhost:8080.
Get initial password:docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword




Why Persistent?:
jenkinsvol stores Jenkins configs, jobs, and plugins.
Data persists even if the container is removed.




Slide 12: Troubleshooting Volumes

Permission Issues:
Ensure user is in the docker group to run commands without sudo:sudo usermod -aG docker $USER
newgrp docker


Check volume permissions:docker volume inspect my-volume
sudo ls -l /var/lib/docker/volumes/my-volume/_data




Volume Not Found:
Verify with docker volume ls.
Recreate if missing: docker volume create my-volume.


Container Fails to Start:
Check logs:docker logs container_name


Ensure volume paths are correct and host directories exist (for bind mounts).


Disk Full:
Monitor disk usage:df -h /var/lib/docker






Slide 13: Key Takeaways

Volumes Enable Persistence:
Named volumes and bind mounts ensure data survives container lifecycles.


Sharing Data:
Use named volumes to share data between containers.


Commands to Master:
Create: docker volume create my-volume
Attach: docker run -v my-volume:/data
List: docker volume ls
Remove: docker volume rm my-volume


Best Practice:
Use named volumes for portability and persistence.
Regularly clean up unused volumes.


Explore More:
Docker Compose for multi-container setups.
Volume plugins for cloud storage (e.g., AWS EBS, NFS).




Slide 14: Q&A

Questions about volumes or persistence?
Need help with specific use cases?
Contact: [Your Name/Email] for further assistance.
