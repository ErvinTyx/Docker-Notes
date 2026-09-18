# Docker Learning Vault

A practical collection of Docker notes, guided labs, and a capstone project. The material is organized as a learning path that starts with images and containers, progresses through core Docker workflows, and ends with deploying an Nginx website.

## Learning Path

Work through the topics in this order:

1. [Images and Containers](Docker%20Images%20and%20Containers.md)
2. [Dockerfile Instructions](Dockerfile%20Instructions.md)
3. [Docker Networking](Docker%20Networking.md)
4. [Docker Storage](Docker%20Storage.md)
5. [Docker Registry](Docker%20Registry.md)
6. [Docker Compose](Docker%20Compose.md)

The [Docker index](Docker.md) provides the same path as an Obsidian-friendly overview with links and a review checklist.

## Notes and Labs

| Topic | Concept notes | Hands-on practice |
| --- | --- | --- |
| Images and containers | [Docker Images and Containers](Docker%20Images%20and%20Containers.md) | Covered throughout the other labs and capstone |
| Dockerfiles | [Dockerfile Instructions](Dockerfile%20Instructions.md) | [Dockerfile lab](Lab%20-%20Dockerfile%20Instructions.md) |
| Networking | [Docker Networking](Docker%20Networking.md) | [Networking lab](Lab%20-%20Docker%20Networking.md) |
| Storage | [Docker Storage](Docker%20Storage.md) | [Storage lab](Lab%20-%20Docker%20Storage.md) |
| Registries | [Docker Registry](Docker%20Registry.md) | [Registry lab](Lab%20-%20Docker%20Registry.md) |
| Compose | [Docker Compose](Docker%20Compose.md) | [Compose lab](Lab%20-%20Docker%20Compose.md) |

Each concept note explains key ideas, useful commands, common mistakes, and links to official documentation. Each lab includes prerequisites, step-by-step exercises, verification checks, cleanup instructions, and troubleshooting guidance.

## Capstone Project

[Deploy an Nginx Website with Docker](Project%20-%20Deploy%20an%20Nginx%20Website%20with%20Docker.md) brings the core workflows together. You will build an image for a static website, run and verify it locally, and publish a versioned image to Docker Hub.

## Using This Repository

### On GitHub

Start with the [Docker index](Docker.md), follow the recommended order, and complete each lab after reading its corresponding concept note.

### In Obsidian

1. Clone or download this repository.
2. Open the repository directory as a vault in [Obsidian](https://obsidian.md/).
3. Open `Docker.md` to begin.

The notes use Obsidian wiki links and tags for navigation inside the vault. Standard Markdown links in this README also work on GitHub.

## Prerequisites

To complete the practical exercises, you will need:

- Docker Engine or Docker Desktop
- The Docker Compose plugin
- A terminal and a text editor
- `curl` for HTTP verification
- A Docker Hub account for the registry lab and capstone publishing steps

Run the commands in a practice environment where you are comfortable creating and removing Docker images, containers, networks, and volumes. Individual labs include scoped cleanup instructions.

## Suggested Workflow

1. Read a concept note.
2. Try its command examples in a safe environment.
3. Complete the paired lab and its verification checklist.
4. Clean up the resources created by the lab.
5. Finish with the capstone project.

For authoritative and current product details, follow the official Docker references included at the end of each note.
