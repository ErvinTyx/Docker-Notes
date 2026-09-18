---
tags:
  - docker
  - docker/lab
---

# Lab - Docker Compose

## Goal

Use a valid `compose.yaml` file to run Nginx, verify it through host port `8080`, and observe Compose's default network.

## Prerequisites

- Docker Engine and the Docker Compose plugin are installed; confirm with `docker compose version`.
- Port `8080` is free on the Docker host.
- Work in an empty directory where it is safe to create a `compose-lab` subdirectory.

## Steps

1. Create the project directory and enter it.

   ```bash
   test ! -e compose-lab || { echo "compose-lab already exists; choose a clean working directory."; exit 1; }
   if docker network inspect compose-lab_default >/dev/null 2>&1; then
     echo "compose-lab_default already exists; inspect it instead of overwriting it."
     exit 1
   fi
   mkdir -p compose-lab
   cd compose-lab
   ```

2. Create `compose.yaml` using spaces for YAML indentation.

   ```yaml
   services:
     web:
       image: nginx:alpine
       ports:
         - "8080:80"
   ```

3. Validate and view the resolved Compose configuration before starting services.

   ```bash
   docker compose --project-name compose-lab config
   ```

4. Start the service in the background, then inspect its status.

   ```bash
   docker compose --project-name compose-lab up -d
   docker compose --project-name compose-lab ps
   ```

5. Verify the Nginx welcome page from the Docker host with a command or in a browser.

   Startup is asynchronous, so retry briefly until the published port is ready:

   ```bash
   for attempt in $(seq 1 20); do
     if curl --fail --silent http://localhost:8080 >/dev/null; then
       break
     fi
     sleep 1
   done
   curl http://localhost:8080
   ```

6. Review service logs.

   ```bash
   docker compose --project-name compose-lab logs web
   ```

7. Observe the default network. Because the file declares no custom network, Compose creates `compose-lab_default` and attaches `web` to it. Services on this network can reach one another by service name; a host-published port is not required for service-to-service traffic.

   ```bash
   docker network inspect compose-lab_default
   ```

## Verification

- `docker compose --project-name compose-lab config` prints a normalized configuration with the `web` service and port mapping `8080:80`.
- `docker compose --project-name compose-lab ps` shows the `web` service running with host port `8080` mapped to container port `80`.
- `curl http://localhost:8080` or a browser at `http://localhost:8080` returns the Nginx welcome page.
- `docker network inspect compose-lab_default` shows the project-scoped default network and the Compose service container.

## Cleanup

From inside `compose-lab`, stop and remove only this Compose project's service container and default network:

```bash
docker compose --project-name compose-lab down
```

This Compose file declares no volumes, so `down` is sufficient and does not require the data-removing `--volumes` option. Optionally remove `compose-lab` after confirming it contains only lab-created files.

## Troubleshooting

- If `docker compose` is unavailable, install or enable the Docker Compose plugin and confirm it with `docker compose version`.
- If `docker compose config` reports YAML errors, check that indentation uses spaces and that `ports` is nested under `web`.
- If `curl` cannot connect, check `docker compose --project-name compose-lab ps` and `docker compose --project-name compose-lab logs web`; also ensure port `8080` is free.

## Related Notes

- [[Docker]]
- [[Docker Compose]]
- [[Docker Networking]]

## Official References

- [Docker Compose overview](https://docs.docker.com/compose/)
- [Compose application model](https://docs.docker.com/compose/intro/compose-application-model/)
- [docker compose reference](https://docs.docker.com/reference/cli/docker/compose/)
