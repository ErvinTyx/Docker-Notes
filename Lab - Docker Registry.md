---
tags:
  - docker
  - docker/lab
---

# Lab - Docker Registry

## Goal

Build a local image, tag it for a Docker Hub repository, and push the exact versioned tag.

## Prerequisites

- Docker Engine is installed and the current user can run `docker` commands.
- A Docker Hub account is available.
- Replace every occurrence of `my-namespace` with your Docker Hub username or an organization namespace where you have permission to push.

## Steps

1. In Docker Hub, create a repository named `my-app` under your namespace. Keep the repository page open for browser verification later.

2. Create a small build context and Dockerfile locally.

   ```bash
   test ! -e registry-lab || { echo "registry-lab already exists; choose a clean working directory."; exit 1; }
   if docker image inspect my-app:1.0 >/dev/null 2>&1; then
     echo "my-app:1.0 already exists; inspect it instead of overwriting it."
     exit 1
   fi
   mkdir -p registry-lab
   cd registry-lab
   printf '<h1>Registry lab</h1>\n' > index.html
   ```

   ```Dockerfile
   FROM nginx:alpine
   COPY index.html /usr/share/nginx/html/index.html
   ```

3. Build the local source image with the stable tag `my-app:1.0`.

   ```bash
   docker build --tag my-app:1.0 .
   ```

4. Authenticate to Docker Hub. Complete the interactive login prompt; use an access token when your account requires one.

   ```bash
   docker login
   ```

5. Add the Docker Hub-qualified tag. Replace `my-namespace` before running the command.

   ```bash
   if docker image inspect my-namespace/my-app:1.0 >/dev/null 2>&1; then
     echo "The qualified lab tag already exists; inspect it instead of overwriting it."
     exit 1
   fi
   docker tag my-app:1.0 my-namespace/my-app:1.0
   ```

6. Push the exact qualified tag.

   ```bash
   docker push my-namespace/my-app:1.0
   ```

7. In the Docker Hub browser page for `my-namespace/my-app`, confirm that tag `1.0` appears. Optionally pull the same reference to check that Docker can resolve it from the registry.

   ```bash
   docker pull my-namespace/my-app:1.0
   ```

## Verification

- `docker image ls` lists both `my-app:1.0` and `my-namespace/my-app:1.0` before local cleanup.
- `docker push my-namespace/my-app:1.0` completes without an authorization error.
- The Docker Hub repository page shows tag `1.0` for `my-namespace/my-app`.
- If used, `docker pull my-namespace/my-app:1.0` completes successfully.

## Cleanup

Remove only the two local tags created for this lab. This does not delete the image from Docker Hub:

```bash
docker image rm my-namespace/my-app:1.0 my-app:1.0
```

Optionally run `docker logout` if you are using a shared machine, and remove `registry-lab` only after confirming it contains only lab-created files.

## Troubleshooting

- A denied push usually means `my-namespace` was not replaced with a namespace you own or can write to, or the repository has not been created.
- Tag and push the fully qualified reference `my-namespace/my-app:1.0`; pushing only `my-app:1.0` does not target your Docker Hub repository.
- If `docker login` fails, verify the account name and use a Docker Hub access token when required by the account's security settings.

## Related Notes

- [[Docker]]
- [[Docker Registry]]
- [[Docker Images and Containers]]

## Official References

- [Push images to Docker Hub](https://docs.docker.com/docker-hub/repos/manage/hub-images/push/)
- [docker login reference](https://docs.docker.com/reference/cli/docker/login/)
- [docker image tag reference](https://docs.docker.com/reference/cli/docker/image/tag/)
