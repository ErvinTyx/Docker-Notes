---
tags:
  - docker
  - docker/lab
---

# Lab - Docker Storage

## Goal

Persist and share data through a Docker-managed named volume, then use a bind mount from a host directory created for this lab.

## Prerequisites

- Docker Engine is installed and the current user can run `docker` commands.
- Work in an empty directory where it is safe to create a `storage-lab` subdirectory.

## Steps

1. Create a dedicated working directory, create the named volume, and inspect its metadata. Docker manages the volume's storage location.

   ```bash
   test ! -e storage-lab || { echo "storage-lab already exists; choose a clean working directory."; exit 1; }
   if docker volume inspect my-volume >/dev/null 2>&1 || docker container inspect volume-writer >/dev/null 2>&1; then
     echo "A storage-lab Docker resource already exists; inspect it instead of overwriting it."
     exit 1
   fi
   mkdir -p storage-lab
   cd storage-lab
   docker volume create my-volume
   docker volume inspect my-volume
   ```

2. Start a writer container that creates data on `my-volume` and remains running. The volume can be mounted by more than one container.

   ```bash
   docker run --detach --name volume-writer \
     --mount type=volume,src=my-volume,dst=/data \
     alpine:3.20 sh -c 'printf "shared volume data\\n" > /data/message.txt; tail -f /dev/null'
   ```

3. Read the data through a temporary helper container with a read-only mount. This is the supported way to inspect Docker-managed volume data; do not read Docker's internal volume directory directly.

   ```bash
   docker run --rm --name my-volume-reader \
     --mount type=volume,src=my-volume,dst=/data,readonly \
     alpine:3.20 cat /data/message.txt
   ```

4. Remove the writer, then use another read-only helper to confirm that the named volume retained its data independently of that container.

   ```bash
   docker rm -f volume-writer
   docker run --rm --name my-volume-persistence-check \
     --mount type=volume,src=my-volume,dst=/data,readonly \
     alpine:3.20 cat /data/message.txt
   ```

5. Create a host directory for the bind-mount exercise and write a file on the host.

   ```bash
   mkdir -p bind-data
   printf 'host bind-mount data\n' > bind-data/host-message.txt
   ```

6. Mount that explicit host path into a temporary container and append a line. The change is made to the host directory because this is a bind mount.

   ```bash
   docker run --rm \
     --mount type=bind,src="$(pwd)/bind-data",dst=/data \
     alpine:3.20 sh -c 'printf "written by container\\n" >> /data/host-message.txt'
   cat bind-data/host-message.txt
   ```

7. Verify the bind-mounted content through a read-only container mount.

   ```bash
   docker run --rm \
     --mount type=bind,src="$(pwd)/bind-data",dst=/data,readonly \
     alpine:3.20 cat /data/host-message.txt
   ```

## Verification

- `docker volume inspect my-volume` reports a volume named `my-volume`.
- Both read-only helper containers print `shared volume data`, including the second helper after `volume-writer` has been removed.
- The host `cat` command and the final read-only bind-mount command both show the line written by the temporary container.

## Cleanup

From inside `storage-lab`, remove the named volume and only the host directory created by this lab:

```bash
docker rm -f volume-writer 2>/dev/null || true
docker volume rm my-volume
test "$(basename "$PWD")" = "storage-lab" && rm -r -- bind-data
```

Optionally return to the parent directory and remove `storage-lab` after confirming it contains only lab-created files.

## Troubleshooting

- If `docker volume rm my-volume` says the volume is in use, remove any container that still mounts it, then retry.
- If Docker rejects the bind mount source, confirm that `$(pwd)/bind-data` exists and that the command is run from inside `storage-lab`.
- Do not use paths under Docker's managed volume storage for inspection or editing. Use a short-lived mounted helper container instead.

## Related Notes

- [[Docker]]
- [[Docker Storage]]
- [[Docker Images and Containers]]

## Official References

- [Volumes](https://docs.docker.com/engine/storage/volumes/)
- [Bind mounts](https://docs.docker.com/engine/storage/bind-mounts/)
