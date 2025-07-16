---
Description: Tool to build and deploy applications and services in the form of containers, using the host kernel as opposed to traditional VMs.
URL: https://www.docker.com/
---

## Components

**images** --> they are like .iso used to created multiple containers
**containers** --> they are the applications/machines running from an image

## Commands

### docker main commands

| Command | Description |
| --- | --- |
| docker version | Show docker current version and info |
| systemctl start docker | Start the docker service |
| docker version | Show docker current version and info |
| docker images | List available images |
| docker pull \<img_name\>:\<version\> | Download the \<name\>:\<version\> image |
| docker rmi \<img_name\>:\<version\><br>docker rmi \<img_id\> | Remove the \<name\>:\<version\> image |
| docker run \<img_name\><br>docker run \<img_id\> | Create a new container from the specified image |
| docker ps -a | List all the available containers |
| docker start \<cont_id\> | Start the selected container |
| docker stop \<cont_id\> | Stop the selected container |
| docker rm \<cont_id\> | Delete the selected container |
| docker attach \<cont_id\> | Attach to the selected container |
| docker logs | Print full `stdout` and `stderr` |

### docker advanced commands

- **Build a docker image** from a script

```bash
docker build -t <name>
```

- Start a container in **interactive mode**

```bash
docker run --name -it <img_name> /bin/bash
```

- **Delete** the container **once exited** from it

```bash
docker run --rm <img_name>
```

- **Bind specific ports** from the container to the operating system (host:container)

```bash
docker run -p 3000:3000 <img_name>
```

- **Detach** the container once running

```bash
docker run -d <img_name>
```

- **Compose** a container **from a docker-compose** file

```bash
docker-compose up
```

- Mount a **shared folder** through host and container

```bash
docker run -v $PWD:/pwd
```

- **Execute a process** within a container

```bash
docker exec -it <cont_id> /bin/bash
```

### docker clenup

```bash
docker system prune -a --volumes
```