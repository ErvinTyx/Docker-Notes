---
tags:
  - docker
  - docker/images-containers
---

# Docker Images and Containers

## Summary

> [!SUMMARY]
> An image is an immutable, layered template that packages an application and its dependencies. A container is a runnable instance of an image with a writable container layer and its own runtime configuration.

## Key Concepts

### Lifecycle

An image can be downloaded from a registry with `docker pull` or built from a Dockerfile. `docker run` creates and starts a container from an image; `docker create` creates one without starting it. A stopped container can be started, restarted, or removed. Removing a container does not remove its image, and removing an image requires that no container still depends on it.

### Image Tags

A tag identifies a particular image version, such as `postgres:16.3`. If an image reference omits a tag, Docker uses the default tag `latest`. `latest` is only a conventional tag name; it does not guarantee that the image is the newest release or that it will remain unchanged. Use a specific version tag when reproducibility matters.

## Command Reference

| Command | Purpose |
| --- | --- |
| `docker pull ubuntu:22.04` | Download an image from a registry. |
| `docker build -t my-app:1.0 .` | Build and tag an image from the Dockerfile in the current build context. |
| `docker image ls` | List local images. |
| `docker image history my-app:1.0` | Show an image's layer history. |
| `docker run --name my-container my-app:1.0` | Create and start a container from an image. |
| `docker ps` | List running containers; add `-a` to include stopped containers. |
| `docker stop my-container` | Request a running container to stop. |
| `docker rm my-container` | Remove a stopped container. |
| `docker image rm my-app:1.0` | Remove a local image that is not used by a container. |

## Building an Image

The build context is the final argument to `docker build`; `.` sends the current directory to the builder. Keep it small with a `.dockerignore` file and use a stable tag when building an application image.

### Ubuntu Dockerfile Example

```Dockerfile
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y --no-install-recommends vim git && rm -rf /var/lib/apt/lists/*
CMD ["bash"]
```

The single `RUN` instruction keeps the package index, installation, and cleanup in one layer. This avoids retaining downloaded package lists in a previous layer.

## Layer Caching Practices

Docker builds an image as a sequence of cached layers. Put stable dependency-installation steps before application source that changes frequently, and group related operations when doing so avoids unnecessary files in an image layer. Changing an instruction or files it copies invalidates that layer and the layers after it.

| Goal | Better approach | Why |
| --- | --- | --- |
| Fast builds | Order layers for effective caching | Docker can reuse unchanged layers. |
| Smaller images | Combine related cleanup with the command that creates temporary files | Temporary files are not retained in an earlier layer. |
| Maintainability | Use readable, purposeful layers | Clear instructions are easier to review and change. |
| Layer reuse | Split independent, stable steps logically | Stable base and dependency layers can be reused from cache. |

## Common Mistakes

- Calling a running container an image. An image is the template; a container is its runtime instance.
- Omitting the build context, for example using `docker build -t my-app:1.0` without the final `.` or another context path.
- Assuming `latest` always identifies the newest image. It is a mutable default tag, not a release guarantee.
- Trying to remove an image while an existing container still references it.

## Related Notes

- [[Docker]]
- [[Dockerfile Instructions]]
- [[Docker Registry]]
- [[Docker Storage]]
- [[Project - Deploy an Nginx Website with Docker]]

## Official References

- [Docker image CLI reference](https://docs.docker.com/reference/cli/docker/image/)
- [Docker Build documentation](https://docs.docker.com/build/)
