# Day 5: Docker Multi-Stage Builds & Image Optimization | CKA Certification Course 2025

## Video reference for Day 5 is the following:

[![Day 5: Docker Multi-Stage Builds & Image Optimization](https://img.youtube.com/vi/p8VQmk_zbVo/1.jpg)](https://www.youtube.com/watch?v=p8VQmk_zbVo)

## Docker Multi-Stage Builds: 

When building a Docker image, certain instructions in the Dockerfile create new layers. These layers contribute to the overall image size. We often include build tools, dependencies, and intermediate files needed only during the build process, not at runtime. These unnecessary files bloat the final image. A large image size implies slower download and deployment times and a larger attack surface.

Multi-stage builds allow us to use multiple `FROM` statements within a single Dockerfile. Each `FROM` instruction starts a new "stage." We can copy artifacts (files) from one stage to another, effectively discarding anything not needed in the final image. This reduces the image size, resulting in faster push, pull, and deployment times, as well as a smaller attack surface.

**Benefits of Multi-Stage Builds:**

*   Optimize image size by separating build and runtime stages.
*   Include tools and dependencies needed only for the build process in one stage and exclude them from the final image.

## Dockerfile Instructions and Layer Creation

When building a Docker image, some instructions create new layers, while others only add metadata or configuration without adding to the image's size. Understanding this is crucial for optimizing image size and build performance.

**Instructions that Create New Layers:**

These instructions modify the filesystem, resulting in a new layer being added to the image:

*   `FROM`: This instruction *always* starts a new base layer.
*   `RUN`: Executes commands inside the container and modifies the filesystem (e.g., installing packages, creating files). This is the most common layer-creating instruction.
*   `COPY`: Copies files and directories from the host into the image's filesystem. The size of the copied files directly impacts the layer size.
*   `ADD`: Similar to `COPY`, but with additional features like handling remote URLs and extracting archives. Like `COPY`, the added content affects the layer size.

**Instructions that Do *Not* Create New Layers (Metadata/Configuration):**

These instructions only add metadata or configuration to the image and do not modify the filesystem, therefore they do not create new layers:

*   `CMD`: Sets the default command to run when the container starts.
*   `ENTRYPOINT`: Sets the main executable for the container.
*   `WORKDIR`: Sets the working directory for subsequent instructions.
*   `EXPOSE`: Declares the ports the container will listen on.
*   `ENV`: Sets environment variables inside the container.
*   `LABEL`: Adds metadata (key-value pairs) to the image.
*   `USER`: Sets the user and group ID for running commands and the container's main process.
*   `VOLUME`: Declares a mount point for volumes. This doesn't create a layer in the image itself but affects how the container interacts with the host's filesystem at runtime.
*   `STOPSIGNAL`: Sets the signal that will be used to stop the container.
*   `ARG`: Defines build-time variables that can be used within the Dockerfile.

**Key Takeaway:**

Minimize the number of layer-creating instructions (especially `RUN`, `COPY`, and `ADD`) to reduce image size. Combine multiple related commands in a single `RUN` instruction using `&&` to reduce the number of layers. Use multi-stage builds to discard unnecessary build artifacts and further optimize image size.

