---
tags:
  - docker
  - docker/lab
---

# Lab - Docker Networking

## Goal

Create a user-defined bridge network and verify that two named Ubuntu containers can reach each other by name.

## Prerequisites

- Docker Engine is installed and the current user can run `docker` commands.
- The Docker host can pull `ubuntu:22.04` and the containers can reach Ubuntu package repositories.

## Steps

1. Create the user-defined bridge network.

   ```bash
   if docker network inspect lab-net >/dev/null 2>&1 \
     || docker container inspect lab-net-client-a >/dev/null 2>&1 \
     || docker container inspect lab-net-client-b >/dev/null 2>&1; then
     echo "A lab-net resource already exists; inspect it instead of overwriting it."
     exit 1
   fi
   docker network create lab-net
   ```

2. Start two named containers from the `ubuntu:22.04` image on that network. The names after `--name` identify containers; `ubuntu:22.04` identifies the image.

   ```bash
   docker run --detach --name lab-net-client-a --network lab-net ubuntu:22.04 sleep infinity
   docker run --detach --name lab-net-client-b --network lab-net ubuntu:22.04 sleep infinity
   ```

3. Install the network tools in each running container.

   ```bash
   docker exec lab-net-client-a sh -c 'apt-get update && apt-get install -y iputils-ping iproute2'
   docker exec lab-net-client-b sh -c 'apt-get update && apt-get install -y iputils-ping iproute2'
   ```

4. Verify name-based connectivity from the first container to the second. A user-defined bridge provides DNS resolution for the container name.

   ```bash
   docker exec lab-net-client-a ping -c 3 lab-net-client-b
   docker exec lab-net-client-a ip addr
   ```

5. Inspect the network and confirm that both containers appear in its container list.

   ```bash
   docker network inspect lab-net
   ```

6. Disconnect the second container, then inspect again to see that it is no longer attached.

   ```bash
   docker network disconnect lab-net lab-net-client-b
   docker network inspect lab-net
   ```

## Verification

- The `ping -c 3 lab-net-client-b` command resolves `lab-net-client-b` and receives replies.
- Before the disconnect, `docker network inspect lab-net` lists both named containers; afterward, it lists only `lab-net-client-a`.
- `ip addr` shows the container's loopback and network interfaces. Do not assume a fixed subnet; the inspect output is authoritative for the local Docker daemon.

## Cleanup

Remove only the two named lab containers, then remove the now-unused lab network:

```bash
docker rm -f lab-net-client-a lab-net-client-b
docker network rm lab-net
```

## Troubleshooting

- If a name is already in use, remove the old lab container with `docker rm -f lab-net-client-a` or `docker rm -f lab-net-client-b`, then repeat the matching start command.
- If `apt-get update` cannot reach repositories, check the Docker host's network and any proxy configuration before retrying.
- If `ping` cannot resolve the other container, confirm both containers are attached to `lab-net` with `docker network inspect lab-net`; name-based discovery is a feature of user-defined networks, not an image property.

## Related Notes

- [[Docker]]
- [[Docker Networking]]

## Official References

- [Bridge network driver](https://docs.docker.com/engine/network/drivers/bridge/)
- [docker network CLI reference](https://docs.docker.com/reference/cli/docker/network/)
