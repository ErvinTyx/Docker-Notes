---
tags:
  - docker
  - docker/project
---

# Project - Deploy an Nginx Website with Docker

## Overview

Package a static school website in an Nginx image, run it locally, confirm that it responds on the host, and publish a versioned image to Docker Hub. This capstone combines image creation, containers, port publishing, and registry workflows.

## Outcome

By the end, the website is available at `http://localhost:8080`, and the image is tagged as `my-namespace/school-website:1.0` for publication to a Docker Hub repository that you own or can write to.

## Prerequisites

- Docker Engine or Docker Desktop is installed and running.
- Git and `curl` are available on the host.
- You have a Docker Hub account and permission to push to the `my-namespace/school-website` repository. Replace `my-namespace` with your Docker Hub username or organization when running the commands.

## 1. Clone and inspect the website

Clone the project, then enter the repository root. Keep the Dockerfile in this directory so that `school-website` is inside the build context.

```bash
git clone https://github.com/DolfinED-HoLabs/Docker-Capstone-Project1.git
cd Docker-Capstone-Project1
find school-website -maxdepth 2 -type f | sort
```

The listing should show the static site files and directories. Do not move `school-website`; the Dockerfile will copy that directory into Nginx's default document root.

## 2. Create the Dockerfile

Create a file named `Dockerfile` in the repository root with the following contents:

```Dockerfile
FROM nginx:latest
COPY ./school-website /usr/share/nginx/html
EXPOSE 80
```

`EXPOSE 80` documents the port used by Nginx inside the container. It does not make the site reachable from the host; the run command publishes that port.

## 3. Build the image

Run the build from the repository root, where both `Dockerfile` and `school-website` are present:

```bash
docker build -t school-website:1.0 .
docker image ls school-website
```

The final `.` is the build context. Docker can copy only files included in that context.

## 4. Run the website

Start a detached container named `school-website`. The host port `8080` maps to Nginx's container port `80`; `--rm` removes the container automatically after it stops.

```bash
docker run --detach --rm --publish 8080:80 --name school-website school-website:1.0
docker ps
```

## 5. Verify the site

Request the published host port:

```bash
curl http://localhost:8080
```

The response should contain HTML from the school website. You can also open `http://localhost:8080` in a browser.

## 6. Tag and push the image

Sign in to Docker Hub, add a repository-qualified tag, and push that exact tag. Create the `school-website` repository in the selected namespace first if it does not already exist.

```bash
docker login
docker tag school-website:1.0 my-namespace/school-website:1.0
docker push my-namespace/school-website:1.0
```

After the push completes, check the Docker Hub repository and confirm that tag `1.0` is listed.

## 7. Clean up local resources

Stop the named container. Because it was started with `--rm`, Docker removes the container after it stops. Then remove both the local build tag and the repository-qualified tag if they are no longer needed.

```bash
docker stop school-website
docker image rm school-website:1.0
docker image rm my-namespace/school-website:1.0
```

If Docker reports that an image is still in use, list containers with `docker ps --all`, stop and remove the container that references it, then repeat the relevant `docker image rm` command.

## Troubleshooting

### Build context error

If `COPY ./school-website /usr/share/nginx/html` cannot find the source directory, confirm that you are in the cloned repository root and build with the final `.`. The `school-website` directory must be inside the build context.

### Port 8080 is occupied

Find the process or container using the port, or choose another unused host port. For example, change `--publish 8080:80` to `--publish 8081:80` and then request `http://localhost:8081`.

### Docker Hub authentication fails

Run `docker login` again and complete the credential or access-token prompt. Confirm that the account has permission to push to the target namespace and repository.

### Repository name or push is rejected

The repository-qualified tag must use a namespace you own or can write to. Retag the local image with your actual Docker Hub username or organization, for example `docker tag school-website:1.0 your-namespace/school-website:1.0`, then push that same reference.

## Related Notes

- [[Docker]]
- [[Docker Images and Containers]]
- [[Dockerfile Instructions]]
- [[Docker Networking]]
- [[Docker Registry]]

## Official References

- [Build images](https://docs.docker.com/build/)
- [Dockerfile reference](https://docs.docker.com/reference/dockerfile/)
- [docker container run CLI reference](https://docs.docker.com/reference/cli/docker/container/run/)
- [Push images to Docker Hub](https://docs.docker.com/docker-hub/repos/manage/hub-images/push/)
