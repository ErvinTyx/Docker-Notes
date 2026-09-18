---
tags:
  - docker
  - docker/dockerfile
aliases:
  - COPY COMMAND
  - WORKDIR
  - ENV
  - LABEL
  - EXPOSE COMMAND
---

# Dockerfile Instructions

## Summary

> [!SUMMARY]
> A Dockerfile is a text file of ordered build instructions that Docker reads to assemble an image. Instructions such as `COPY`, `WORKDIR`, `ENV`, `LABEL`, and `EXPOSE` define the image's files, defaults, metadata, and documented network ports.

## Key Concepts

### Dockerfile Purpose

A Dockerfile makes an image build repeatable. Docker processes its instructions in order, beginning from a base image declared with `FROM`. The build context controls which local files are available to `COPY`; files outside it cannot be copied into the image.

### Instruction Reference

| Instruction | Purpose |
| --- | --- |
| `COPY` | Copy files or directories from the build context into the image. |
| `WORKDIR` | Set the working directory for following Dockerfile instructions and the container's default command. |
| `ENV` | Set environment variables that persist in the built image and its containers. |
| `LABEL` | Add key-value metadata to an image. |
| `EXPOSE` | Document the ports on which the application intends to listen. |

## Instruction Syntax and Examples

### COPY

Syntax:

```Dockerfile
COPY [--chown=<user>:<group>] <src> ... <dest>
```

Example:

```Dockerfile
COPY ./app/ /usr/src/app/
```

`COPY` requires spaces between each source and destination argument. Its sources are resolved from the build context, not from arbitrary host paths.

### WORKDIR

Syntax:

```Dockerfile
WORKDIR /path
```

Example:

```Dockerfile
WORKDIR /usr/src/app
```

If the directory does not already exist, Docker creates it. The path is inside the image and container, not on the Docker host.

### ENV

Syntax:

```Dockerfile
ENV <key>=<value> ...
```

Example:

```Dockerfile
ENV action=ping target=8.8.8.8
```

`ENV` supplies defaults that a user can override when creating a container:

```bash
docker run --env action=curl --env target=https://example.com my-image
```

### LABEL

Syntax:

```Dockerfile
LABEL <key>=<value> ...
```

Example:

```Dockerfile
LABEL org.opencontainers.image.title="my-app" org.opencontainers.image.version="1.0"
```

Labels attach metadata to an image; they do not configure the running application.

### EXPOSE

Syntax:

```Dockerfile
EXPOSE <port> [<port>/<protocol>...]
```

Example:

```Dockerfile
EXPOSE 8080/tcp
```

`EXPOSE` documents the ports on which an application intends to listen, but it does not publish those ports to the host. Publish a port when starting a container, for example `docker run --publish 8080:8080 my-image`.

^f2eb86

## COPY and WORKDIR Example

Corrected project tree:

```text
.
├── Dockerfile
└── app/
    ├── main.py
    └── utils/
        └── helper.py
```

Python Dockerfile:

```Dockerfile
FROM python:3.10-slim
WORKDIR /usr/src/app
COPY ./app/ ./
CMD ["python", "main.py"]
```

`COPY ./app/ ./` places the application in the current `WORKDIR`, so `main.py` and `utils/helper.py` are both available to the command.

## Environment Variable Expansion

The exec form of `CMD`, such as `CMD ["echo", "$message"]`, does not perform shell variable expansion. Use a shell explicitly only when expansion is required:

```Dockerfile
CMD ["sh", "-c", "exec $action $target"]
```

## Common Mistakes

- Misspelling `WORKDIR`; use the instruction name exactly as written.
- Omitting spaces in `COPY`, such as `COPY ./app./`, instead of providing separate source and destination paths.
- Trying to copy a file outside the build context. Change the build context deliberately or reorganize the files instead.
- Expecting `$variable` expansion in exec-form `CMD` or `ENTRYPOINT`; exec form does not invoke a shell.
- Confusing `EXPOSE` with `--publish`: `EXPOSE` documents a port, while `--publish` maps a host port to a container port.

## Related Notes

- [[Docker]]
- [[Docker Images and Containers]]
- [[Lab - Dockerfile Instructions]]
- [[Project - Deploy an Nginx Website with Docker]]

## Official References

- [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)
- [Writing a Dockerfile](https://docs.docker.com/get-started/docker-concepts/building-images/writing-a-dockerfile/)
