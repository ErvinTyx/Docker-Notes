---
tags:
  - docker
  - docker/networking
---

# Docker Networking

> [!summary]
> Docker networking gives containers controlled paths to communicate with each other, the Docker host, and external networks. Choose a network driver for the topology, then use user-defined networks to make an application's boundaries and service discovery explicit.

## Key concepts

- A network driver defines how Docker connects containers. Docker includes `bridge`, `host`, `none`, `overlay`, `ipvlan`, and `macvlan` drivers.
- Bridge networks apply to containers on the same Docker daemon host. Containers on the same bridge can communicate; separate bridge networks provide isolation by default.
- A container can belong to more than one user-defined network. This is useful for a component that must communicate with two otherwise isolated groups.
- Bridge networking uses outbound masquerading by default, so external networks see the Docker host's address. Publish a port to make a container port reachable through a host address.

## Network drivers

| Driver | Use it when | Notes |
| --- | --- | --- |
| `bridge` | Containers on one Docker host need to communicate. | Default driver for new networks; user-defined bridges are usually the best choice for standalone applications. |
| `host` | A container must use the host network stack directly. | Removes network isolation between the container and host; port publishing is not used. |
| `none` | A container needs no external networking. | Completely isolates the container from the host and other containers. |
| `overlay` | Swarm services or containers on multiple Docker daemons need to communicate. | Connects Docker daemons across nodes. |
| `ipvlan` | An underlay network needs controlled IPv4/IPv6 addressing without a unique MAC per container. | Useful where an interface or port has MAC-address limits. |
| `macvlan` | A container must appear as a physical device on the network. | Assigns the container its own MAC address; useful for some legacy network expectations. |

## Default bridge and user-defined bridges

Docker creates the default `bridge` network when it starts. A container with no `--network` option joins it by default. Its subnet is configurable, so inspect the local daemon instead of assuming `172.17.0.0/16`:

```bash
docker network inspect bridge
```

| Capability | Default `bridge` | User-defined bridge |
| --- | --- | --- |
| Container discovery | Containers normally reach each other by IP address. | Automatic DNS lets containers resolve a custom container name or network alias. |
| Isolation | All containers that omit `--network` share it. | Scope access to only containers attached to that network. |
| Attach or detach a running container | Recreate the container to change its default-bridge membership. | Use `docker network connect` and `docker network disconnect` while it runs. |
| Configuration | Daemon-level configuration and restart are required. | Configure each network at creation time. |

Prefer a user-defined bridge for application containers. It provides name-based service discovery and a deliberate isolation boundary. Ports are available to peers on the same user-defined bridge; use `--publish` to expose a port to the host or other networks.

## Network command reference

| Goal | Command |
| --- | --- |
| List networks | `docker network ls` |
| Create a bridge network | `docker network create my-network` |
| Create with an IPv4 subnet | `docker network create --subnet 10.0.0.0/16 my-network` |
| Inspect a network | `docker network inspect my-network` |
| Start a container on a network | `docker run --name my-container --network my-network nginx:latest` |
| Add a running container | `docker network connect my-network my-container` |
| Remove a running container | `docker network disconnect my-network my-container` |

## Syntax and examples

Create an isolated bridge network, then start a web container on it and publish the web port through the host:

```bash
docker network create my-network
docker run --detach --name my-container --network my-network --publish 8080:80 nginx:latest
```

Attach a running container to a second user-defined network when it needs to reach services in both places:

```bash
docker network create second-network
docker network connect second-network my-container
docker network disconnect second-network my-container
```

## Common mistakes

- Confusing a container name with an image name: `my-container` identifies a created container, while `nginx:latest` identifies the image used to create it.
- Typing `docker networks ls`; the command is `docker network ls` (singular `network`).
- Treating the default bridge subnet as universal. Docker daemon configuration can change it; check `docker network inspect bridge` on the relevant host.
- Expecting containers on different bridge networks to communicate directly. Connect a container to both networks only when that connection is intentional, or publish a port for controlled access.

## Related notes

- [[Docker]]
- [[Docker Compose]]
- [[Lab - Docker Networking]]
- [[Project - Deploy an Nginx Website with Docker]]

## Official references

- [Docker network drivers](https://docs.docker.com/engine/network/drivers/)
- [Docker bridge network driver](https://docs.docker.com/engine/network/drivers/bridge/)
