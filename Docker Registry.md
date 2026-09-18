---
tags:
  - docker
  - docker/registry
---

# Docker Registry

## Summary

> [!summary]
> A Docker registry stores and distributes images. Before pushing an image, authenticate to the destination registry and tag the local image with the destination's repository name.

## Key Concepts

- A **registry** is an image distribution service, such as Docker Hub or a registry at `registry.example.com:5000`.
- A **repository** is a named collection of related images in a registry. A repository can contain multiple tags.
- A **tag** is the mutable name portion of an image reference, such as `1.0` in `my-namespace/my-app:1.0`. Use a specific release tag when reproducibility matters; do not treat `latest` as a guarantee of recency or immutability.

### Docker Hub Naming

Docker Hub is the default registry when an image reference does not include a registry host. For an image in a personal or organization namespace, use:

```text
my-namespace/my-app:1.0
```

Docker Hub also hosts Docker Official Images in the `library` namespace; Docker normally lets users omit that namespace, for example `ubuntu:24.04` rather than `library/ubuntu:24.04`.

## Push an Image to Docker Hub

Tag the local image with a repository that you own or can write to, then push that exact tag. This ordered workflow assumes `my-app:1.0` already exists locally.

```bash
docker login
docker tag my-app:1.0 my-namespace/my-app:1.0
docker push my-namespace/my-app:1.0
```

After a successful push, open the `my-namespace/my-app` repository in Docker Hub and confirm that tag `1.0` appears in its tag list. Pulling the same reference from another machine is an additional end-to-end check:

```bash
docker pull my-namespace/my-app:1.0
```

## Command Reference

| Command | Purpose |
| --- | --- |
| `docker login` | Authenticate to Docker Hub. |
| `docker login registry.example.com:5000` | Authenticate to a non-Hub registry host. |
| `docker tag my-app:1.0 my-namespace/my-app:1.0` | Add a Docker Hub-qualified tag to a local image. |
| `docker push my-namespace/my-app:1.0` | Upload that tag to Docker Hub. |
| `docker push registry.example.com:5000/my-namespace/my-app:1.0` | Upload a tag to a non-Hub registry. |

## Non-Hub Registry Syntax

For a registry other than Docker Hub, include its host, and port when applicable, in both the tag and push reference:

```bash
docker login registry.example.com:5000
docker tag my-app:1.0 registry.example.com:5000/my-namespace/my-app:1.0
docker push registry.example.com:5000/my-namespace/my-app:1.0
```

The registry host is part of the image reference. Tagging only as `my-namespace/my-app:1.0` targets Docker Hub, not `registry.example.com:5000`.

## Common Mistakes

- Pushing `my-app:1.0` before adding the destination repository name. Docker needs a tag that identifies the target registry and repository.
- Using a namespace that is not your Docker Hub username or organization, or a repository that has not been created or made available to you.
- Logging in to Docker Hub when the destination is a private registry. Authenticate to the same registry host that appears in the image reference.
- Omitting the registry host for a non-Hub push; Docker interprets the reference as Docker Hub.
- Replacing a release tag with `latest` and assuming it identifies one fixed image. Tags can be moved to another image.

## Related Notes

- [[Docker]]
- [[Docker Images and Containers]]
- [[Lab - Docker Registry]]
- [[Project - Deploy an Nginx Website with Docker]]

## Official References

- [Push images to Docker Hub](https://docs.docker.com/docker-hub/repos/manage/hub-images/push/)
- [docker login CLI reference](https://docs.docker.com/reference/cli/docker/login/)
- [docker image tag CLI reference](https://docs.docker.com/reference/cli/docker/image/tag/)
- [docker image push CLI reference](https://docs.docker.com/reference/cli/docker/image/push/)
