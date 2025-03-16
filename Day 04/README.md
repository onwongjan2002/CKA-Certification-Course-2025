# Day 4: Docker Flags, Deep Dive into Dockerfile, and Exposing Containers | CKA Certification Course 2025

## Video reference for Day 4 is the following:

[![Watch the video](https://img.youtube.com/vi/34l1gRszQS4/maxresdefault.jpg)](https://youtu.be/34l1gRszQS4)

# Specifying a Custom Dockerfile Name and Understanding the Build Command in Docker

When working with Docker, the default name for a Dockerfile is `Dockerfile`. However, you can use a custom name for your Dockerfile. Here’s how to specify a custom name and understand the full `docker build` command, including the build context.

---

## **Why Use a Custom Name for Dockerfile?**
Sometimes, you may want to use different Dockerfiles for different purposes (e.g., production vs. development). Naming them accordingly (e.g., `Dockerfile.dev` or `Dockerfile.prod`) helps organize your project better.

---

## **Building an Image with a Custom Dockerfile Name**

To build an image using a custom-named Dockerfile, you need to use the `-f` flag with the `docker build` command.

### **Command Syntax**
```bash
docker build -t <image-name> -f <path-to-custom-dockerfile> <build-context>
```

`-t`: Tags the image
`-f`: Specifies the custom Dockerfile name
`<build-context>`: The directory containing files required for the build

### What is Build Context?

The directory Docker uses to locate files for the build. Files outside the build context are not accessible during the build.

If the custom-dockerfile and build context are in different directories:

```bash
docker build -t entry-image -f /path/to/custom-dockerfile /path/to/build-context
```


### ENTRYPOINT vs CMD

`CMD` and `ENTRYPOINT` may seem similar, but they serve distinct purposes. The key difference is that `ENTRYPOINT` is used to specify the primary executable for the container, essentially telling Docker that the container is designed to run a specific application. If you try to override it with another command or executable, it might not behave as expected or might fail. On the other hand, `CMD` provides default arguments or a command that can be overridden if desired.

| **Aspect**              | **`CMD`**                                                                                             | **`ENTRYPOINT`**                                                                                  | **`RUN`**                                                                                          |
|--------------------------|-------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------|
| **Purpose**             | Specifies the default command to execute when the container starts.                                   | Specifies the command that will always execute when the container starts (immutable part).        | Executes commands during the image build process.                                                  |
| **Execution Context**   | Runs when a container is started.                                                                    | Runs when a container is started.                                                                | Runs during the **image build** phase (at build time).                                              |
| **Default Behavior**    | Can be overridden by the user at runtime (`docker run <image> <command>`).                            | Cannot be fully overridden by the user unless `--entrypoint` is specified.                       | Used for preparing the image (e.g., installing dependencies, setting up the environment).          |
| **Command Type**        | Acts as the default "runtime" command for the container.                                              | Acts as the "always executed" entrypoint for the container.                                       | Executes commands to modify the image layers during build time.                                    |
| **Form Supported**      | Supports **shell form** and **exec form**.                                                           | Supports **exec form** only.                                                                      | Supports **shell form** and **exec form**.                                                         |
| **Overriding Behavior** | User can replace it entirely at runtime.                                                             | User can only append arguments to it at runtime (unless overridden explicitly with `--entrypoint`). | Once executed during image build, the result is baked into the image.                             |
| **Common Use Cases**    | Specify the default script or command to run, such as starting an app server.                        | Used for setting up an entry point (e.g., a wrapper script or command) that runs regardless of additional parameters. | Install dependencies, set up the environment, and make the image production-ready.                |
| **Example (Exec Form)** | `CMD ["python", "app.py"]`                                                                            | `ENTRYPOINT ["python", "app.py"]`                                                                | `RUN apt-get update && apt-get install -y python3`                                                 |
| **Example (Shell Form)**| `CMD python app.py`                                                                                   | Not supported.                                                                                    | `RUN apt-get update && apt-get install -y python3`                                                 |
| **Chaining with CMD**   | Only one `CMD` instruction is allowed per Dockerfile (the last one overrides previous ones).          | Can be combined with `CMD` to provide default arguments (e.g., `CMD ["arg1", "arg2"]`).           | Multiple `RUN` instructions are allowed, and each creates a new layer in the image.               |
| **When to Use**         | When you want a default command that users can override at runtime.                                   | When you want to ensure a specific command or script is always executed for the container.         | When you need to execute commands during the build phase to bake results into the image.           |

## Docker Commands

**Container Management:**

*   List all containers (including stopped ones): `docker ps -a`
*   Inspect a container’s metadata: `docker inspect <container_id>` or `docker inspect <image_id>`
*   List running processes in a container: `docker top <container_id>`
*   Stop a specific container: `docker stop <container_id>`
*   Start a stopped container: `docker start <container_id>`
*   Stop all running containers: `docker stop $(docker ps -q)`
*   Restart a container: `docker restart <container_id>`
*   Delete a specific container: `docker rm <container_id>`
*   Delete all stopped containers: `docker container prune`

**Image Management:**

*   Dangling images in Docker are images that are no longer tagged or associated with any container.
*   Delete a specific image: `docker rmi <image_id>`
*   Delete all unused images (including dangling images): `docker image prune -a`

Good Read: https://www.docker.com/blog/docker-best-practices-choosing-between-run-cmd-and-entrypoint/