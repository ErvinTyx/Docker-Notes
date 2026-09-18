---
tags:
  - docker
  - docker/compose
---

# Docker Compose

## Summary

> [!summary]
> Docker Compose defines and runs a multi-container application from a Compose file. Its model declares services and the shared resources they use, so the application's containers can be created and managed as one project.

## Key Concepts

- **Services** define the containers that make up the application, including their image or build settings, ports, mounts, networks, and runtime configuration.
- **Networks** define how services communicate. Compose creates a project-scoped default network when a service does not declare another network.
- **Volumes** provide storage that outlives a container. Compose can use named volumes declared at the top level or anonymous volumes declared only by a service.
- **Configs** provide non-sensitive configuration data to services. **Secrets** provide sensitive data to services; do not place credentials directly in a Compose file when a secret mechanism is appropriate.

## A Two-Service `compose.yaml`

Save this as `compose.yaml` and run `docker compose up`. Both services join the automatically created default network and can resolve each other by service name, such as `web` or `worker`.

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
  worker:
    image: alpine:3.20
    command: ["sh", "-c", "while true; do sleep 3600; done"]
```

## Compose File Syntax

### Use an Existing Image

```yaml
services:
  app:
    image: my-namespace/my-app:1.0
```

### Build Context

```yaml
services:
  app:
    build:
      context: .
```

### Alternate Dockerfile

```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
```

`context` is the directory sent to the builder. `dockerfile` is the Dockerfile path relative to that context (or an absolute path).

### Set a Container Name

```yaml
services:
  app:
    image: nginx:alpine
    container_name: my-app
```

`container_name` overrides Compose's generated container name. Avoid it when a service must be scaled, because a fixed container name prevents multiple replicas of that service.

### Publish a Port

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
```

Quote port mappings so YAML preserves them as strings. The mapping above exposes container port `80` through host port `8080`.

### Bind Mount

```yaml
services:
  web:
    image: nginx:alpine
    volumes:
      - ./site:/usr/share/nginx/html:ro
```

The `./site` source is resolved relative to the Compose file's parent directory. A bind mount exposes a host path to the container; use `:ro` when the service only needs to read it.

### Named Volume

```yaml
services:
  database:
    image: postgres:16
    volumes:
      - database-data:/var/lib/postgresql/data

volumes:
  database-data:
```

A named volume must be declared at the top level when a service refers to it. Docker manages its location and it persists independently of the service container.

### Anonymous Volume

```yaml
services:
  cache:
    image: redis:7-alpine
    volumes:
      - /data
```

The container path alone creates an anonymous volume. It has no top-level declaration, so it is harder to identify and reuse deliberately than a named volume.

### Custom Network

```yaml
services:
  web:
    image: nginx:alpine
    networks:
      - application
  api:
    image: alpine:3.20
    command: ["sh", "-c", "while true; do sleep 3600; done"]
    networks:
      - application

networks:
  application:
    driver: bridge
```

Services on the same Compose network can discover one another by service name. A published port is for access through the host; it is not required for one service to reach another on their shared network.

## Default Networking and Discovery

If a Compose file has services but no top-level `networks` declaration, Compose creates a default network for the project and attaches every service to it. Docker's embedded DNS makes a service name resolvable from other services on that network. Use the service name and the container port for internal connections, for example `http://web:80`, rather than a host-published port such as `http://localhost:8080`.

## Command Reference

| Command | Purpose |
| --- | --- |
| `docker compose config` | Resolve and display the effective Compose model; use it to validate configuration before starting services. |
| `docker compose build` | Build images for services that declare `build`. |
| `docker compose up` | Create and start services, attaching logs to the terminal. |
| `docker compose up -d` | Create and start services in the background. |
| `docker compose run SERVICE` | Start a one-off container for the named service; `SERVICE` is required. |
| `docker compose stop` | Stop running service containers without removing them. |
| `docker compose restart` | Restart service containers. |
| `docker compose down` | Stop and remove the project's containers and networks. |
| `docker compose down --volumes` | Also remove named volumes declared by the Compose file and attached anonymous volumes. |
| `docker compose down --remove-orphans` | Also remove containers for services no longer defined in the Compose file. |

## Common Mistakes

- Indenting YAML with tabs or inconsistent spacing. Use spaces and keep each nested level aligned; the examples use two spaces.
- Omitting the required space after a colon, such as `image:nginx:alpine`. YAML needs `image: nginx:alpine`.
- Running `docker compose run` without a service argument. Use `docker compose run SERVICE`, for example `docker compose run app`.
- Running `docker compose down --volumes` without checking which volumes hold data. It removes the project's named volumes and attached anonymous volumes, which can permanently delete application data.
- Assuming a host port is needed for service-to-service traffic. Services on a shared Compose network use the service name and container port.
- Declaring a named volume under a service but forgetting its top-level `volumes` declaration.

## Related Notes

- [[Docker]]
- [[Docker Networking]]
- [[Docker Storage]]
- [[Lab - Docker Compose]]
- [[Project - Deploy an Nginx Website with Docker]]

## Official References

- [Docker Compose overview](https://docs.docker.com/compose/)
- [Compose application model](https://docs.docker.com/compose/intro/compose-application-model/)
- [Compose file reference](https://docs.docker.com/reference/compose-file/)
- [Services reference](https://docs.docker.com/reference/compose-file/services/)
- [Volumes reference](https://docs.docker.com/reference/compose-file/volumes/)
- [Networking reference](https://docs.docker.com/reference/compose-file/networks/)
- [docker compose CLI reference](https://docs.docker.com/reference/cli/docker/compose/)
