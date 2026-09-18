---
tags:
  - docker
  - docker/lab
---

# Lab - Dockerfile Instructions

## Goal

Build and run a small Nginx image while practicing `WORKDIR`, `ENV`, `LABEL`, `COPY`, `EXPOSE`, and host port publishing.

## Prerequisites

- Docker Engine is installed and the current user can run `docker` commands.
- Port `8080` is free on the Docker host.
- Work in an empty directory where it is safe to create a `dockerfile-lab` subdirectory.

## Steps

1. Create the build context and a small web page.

   ```bash
   test ! -e dockerfile-lab || { echo "dockerfile-lab already exists; choose a clean working directory."; exit 1; }
   if docker container inspect lab-web >/dev/null 2>&1 || docker image inspect dockerfile-lab:1.0 >/dev/null 2>&1; then
     echo "A Dockerfile lab resource already exists; inspect it instead of overwriting it."
     exit 1
   fi
   mkdir -p dockerfile-lab/app
   cd dockerfile-lab
   printf '<h1>Dockerfile lab</h1>\n' > app/index.html
   ```

2. Create `Dockerfile` in `dockerfile-lab`. `COPY app/ ./` uses the current directory as the build context, and `EXPOSE` documents the container port; neither command publishes a host port.

   ```Dockerfile
   FROM nginx:alpine
   WORKDIR /usr/share/nginx/html
   ENV APP_NAME="dockerfile-lab"
   LABEL org.opencontainers.image.title="dockerfile-lab"
   COPY app/ ./
   EXPOSE 80
   CMD ["nginx", "-g", "daemon off;"]
   ```

3. Build the image. The final `.` is the build context containing both `Dockerfile` and `app/`.

   ```bash
   docker build --tag dockerfile-lab:1.0 .
   ```

4. Inspect the image metadata and configuration.

   ```bash
   docker image inspect dockerfile-lab:1.0
   ```

5. Start a named container and explicitly publish host port `8080` to the container's port `80`.

   ```bash
   docker run --detach --name lab-web --publish 8080:80 dockerfile-lab:1.0
   ```

6. Check the page from the Docker host.

   ```bash
   curl http://localhost:8080
   ```


## Verification

- `docker image inspect dockerfile-lab:1.0` shows the image, including its configured environment, label, working directory, and exposed port.
- `docker run` returns a container ID and `docker ps` shows `lab-web` with `0.0.0.0:8080->80/tcp` (or the equivalent IPv6 mapping).
- `curl http://localhost:8080` returns the `Dockerfile lab` heading.

## Cleanup

From inside `dockerfile-lab`, remove only the lab's explicitly named container and image:

```bash
docker stop lab-web 2>/dev/null || true
docker rm -f lab-web 2>/dev/null || true
docker image rm dockerfile-lab:1.0
```

Optionally return to the parent directory and remove `dockerfile-lab` after confirming it contains only files created for this lab.

## Troubleshooting

- If the build reports that `app/` is missing, run the build from inside `dockerfile-lab`, where the final `.` includes `app/` in the context.
- If port `8080` is already allocated, stop the service using it or choose another free host port and use the same port in the `curl` command.
- `EXPOSE 80` alone does not make the page reachable from the host; use `--publish 8080:80` when starting the container.

## Related Notes

- [[Docker]]
- [[Dockerfile Instructions]]
- [[Docker Images and Containers]]

## Official References

- [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)
- [docker container run reference](https://docs.docker.com/reference/cli/docker/container/run/)
