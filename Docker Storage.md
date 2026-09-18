---
aliases:
  - Persisting data in Docker
tags:
  - docker
  - docker/storage
---

# Docker Storage

> [!summary]
> Keep persistent application data outside a container's writable layer. Use Docker-managed named volumes for durable container data and bind mounts when a container must use a specific path on the Docker host.

## Key concepts

- Files written in a container's writable layer are removed with that container. Do not use that layer as the only home for database files, uploads, logs that must survive, or other persistent application data.
- A named volume is created and managed by Docker, persists independently of containers, and is the usual choice for durable application data.
- A bind mount maps a path on the Docker daemon host into a container. It is useful for source code, generated artifacts, and host-specific configuration, but it couples the container to that host path.
- Prefer the explicit `--mount` syntax. It makes the mount type, source, destination, and read-only setting clear.

## Named volumes and bind mounts

| Mount type | Source | Best fit | Important trade-off |
| --- | --- | --- | --- |
| Named volume | A Docker-managed data store, such as `my-volume`. | Persistent application data, backups, and sharing data between containers. | Docker controls the storage location; access the contents through a mounted container. |
| Bind mount | A file or directory on the Docker daemon host. | Development source files, build artifacts, or host-provided configuration. | Container processes can modify host files by default, and the path must exist on compatible hosts. |

## Volume command reference

| Goal | Command |
| --- | --- |
| Create a named volume | `docker volume create my-volume` |
| List volumes | `docker volume ls` |
| Inspect a volume | `docker volume inspect my-volume` |
| Remove an unused volume | `docker volume rm my-volume` |

## Syntax and examples

Mount a named volume with the recommended explicit syntax:

```bash
docker volume create my-volume
docker run --detach --name my-container \
  --mount type=volume,src=my-volume,dst=/var/lib/app \
  nginx:latest
```

The short equivalent uses `-v` with the volume name first and the container path second:

```bash
docker run --detach --name my-container \
  -v my-volume:/var/lib/app \
  nginx:latest
```

Mount a host directory as a bind mount. The host path is the source; the container path is the destination:

```bash
docker run --rm --name my-container \
  --mount type=bind,src="$(pwd)",dst=/project \
  nginx:latest

docker run --rm --name my-container \
  -v "$(pwd)":/project \
  nginx:latest
```

Make a mount read-only when the container only needs to read it:

```bash
docker run --rm --name my-container \
  --mount type=volume,src=my-volume,dst=/data,readonly \
  nginx:latest

docker run --rm --name my-container \
  -v my-volume:/data:ro \
  nginx:latest
```

Two containers can mount the same named volume. For example, one can write while another reads it:

```bash
docker run --detach --name writer \
  --mount type=volume,src=my-volume,dst=/data \
  nginx:latest
docker run --detach --name reader \
  --mount type=volume,src=my-volume,dst=/data,readonly \
  nginx:latest
```

Inspect volume data by mounting it into a temporary container rather than reading Docker's internal volume directory directly:

```bash
docker run --rm --mount type=volume,src=my-volume,dst=/data,readonly \
  alpine:latest ls -la /data
```

### Legacy: `--volumes-from`

`--volumes-from` inherits every volume mount from another container, for example `docker run --volumes-from my-container alpine:latest`. It is a legacy convenience option; declare each required mount explicitly with `--mount` or `-v` so a container's storage dependencies remain visible.

## Common mistakes

- Running `docker inspect my-volume` instead of `docker volume inspect my-volume`.
- Assuming `--rm` removes named volumes. It removes the container; however, it removes an anonymous volume associated with that container. Remove named volumes explicitly when they are no longer needed.
- Reversing mount paths. For both `--mount` and `-v`, the source is on the host (or is the named volume) and the destination is inside the container.
- Confusing read-only syntax between mount forms. With `-v`, put `:ro` (or `:readonly`) after the container path; with `--mount`, use `readonly` in the comma-separated option list.
- Reading or changing `/var/lib/docker/volumes` directly. Docker manages that storage; mount the volume into a container to inspect its contents.

## Related notes

- [[Docker]]
- [[Docker Images and Containers]]
- [[Docker Compose]]
- [[Lab - Docker Storage]]

## Official references

- [Docker storage](https://docs.docker.com/engine/storage/)
- [Docker volumes](https://docs.docker.com/engine/storage/volumes/)
- [Docker bind mounts](https://docs.docker.com/engine/storage/bind-mounts/)
