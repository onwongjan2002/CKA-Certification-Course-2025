# Day 5: Docker Multi-Stage Builds & Image Optimization | CKA Certification Course 2025

## Video reference for Day 5 is the following:

[![Watch the video](https://img.youtube.com/vi/8q3pJfE6Z_E/maxresdefault.jpg)](https://www.youtube.com/watch?v=8q3pJfE6Z_E&ab_channel=CloudWithVarJosh)

---
## ⭐ Support the Project  
If this **repository** helps you, give it a ⭐ to show your support and help others discover it! 

---

## Compiled vs. Interpreted Languages

## Compiled Languages

* Source code is translated into machine code (binary) before execution.
* Requires a build stage with compilers and tools.
* Offers faster execution due to direct machine code interaction.
* Less portable as the generated binary may not run on different architectures.
* Examples: Java, C/C++, Go, etc..

## Interpreted Languages

* Source code is executed directly by an interpreter at runtime.
* No separate build stage is generally required.
* Typically slower execution due to the interpretation process.
* More portable as they can run on different systems with the interpreter available.
* Examples: Python, JavaScript (Node.js), PHP, etc..

## Why Multi-Stage Builds? What are Build And Runtime Stage?

In production environments, applications often require a build process to prepare the application for deployment. This process can involve tools like compilers, additional libraries, and dependencies. However, many of these tools are only needed during the **build phase** and not when the application is actually **running**. This is where multi-stage builds shine—they help create lightweight production images by separating the build stage from the runtime stage.

The build stage includes everything needed to prepare your app for deployment, while the runtime stage focuses on running the app efficiently. For small Python apps like ours, the build tools aren’t necessary. However, in production environments with complex applications, separating build and runtime stages is critical for creating optimized, secure Docker images.

| **Aspect**           | **Build Stage**                                 | **Runtime Stage**                            |
|-----------------------|------------------------------------------------|---------------------------------------------|
| **Purpose**           | Prepares the application for deployment.       | Provides an environment to run the app.     |
| **Tools/Dependencies**| Includes tools for compiling or building (e.g., `build-essential`). | Only essential libraries for running the app (e.g., Flask). |
| **Image Size**        | Larger, as it includes unnecessary tools.      | Smaller, optimized for deployment.          |
| **Example in Our Dockerfile** | Install `build-essential` in this stage.              | Exclude `build-essential` for a lightweight runtime. |

### **Pizza Analogy**

Think of the build and runtime stages like making and eating a pizza:

1. **Build Stage:**
   - When making the pizza, you need tools measuring cup, a microwave, and ingredients (dough, cheese, toppings, etc.).
   - These tools are essential for preparing the pizza but won’t be served with it.

2. **Runtime Stage:**
   - Once the pizza is baked, you only need the finished pizza to enjoy.
   - The tools and raw ingredients used during preparation are no longer needed.

### By separating the build and runtime stages:
- You create a smaller image for production, which reduces deployment time and resource usage.
- You improve security by excluding unnecessary tools and dependencies from the runtime image.
- You simplify debugging by isolating the build process from the runtime environment.

# Dockerfile Example

```dockerfile
# Build Stage: Preparing the application for production
FROM python:3.9-slim AS build  

# Set the working directory inside the container to /app
WORKDIR /app  

# Copy the app.py file from the host into the container
COPY app.py app.py  

# Install Flask (required for the app) and build tools (for demonstration purposes)
RUN pip install flask && apt update && apt install -y build-essential  

# Runtime Stage: Lightweight image for running the application
FROM python:3.9-slim  

# Set the working directory inside the container to /app
WORKDIR /app  

# We copy only the necessary files from the build stage
COPY --from=build /app .  

# Install Flask (to ensure it’s available in the runtime image)
RUN pip install flask  

# Expose port 5000 for the container
EXPOSE 5000  

# Define the executable and arguments for the container
ENTRYPOINT [ "python" ]  
CMD [ "app.py" ]  
```

### **Docker Multi-Stage Builds**  

When building a Docker image, certain instructions create layers that increase the image size. Tools, dependencies, and intermediate files needed only during the **build process** often bloat the final image, leading to:  
- **Slower downloads and deployments**  
- **Larger attack surfaces**  

**Multi-stage builds** solve this by allowing multiple `FROM` instructions in a Dockerfile. Each stage isolates its purpose, and we copy only the required files into the final image, discarding unnecessary layers.  

#### **Key Benefits of Multi-Stage Builds:**  
- **Smaller Image Size**: Separates build and runtime, keeping the final image lean.  
- **Efficient Deployment**: Faster push, pull, and deployment times.  
- **Improved Security**: Reduces the attack surface by excluding unneeded tools.  

**Instructions that Create New Layers:** FROM, RUN, COPY, ADD

**Instructions that Do Not Create New Layers:** CMD, ENTRYPOINT, WORKDIR, EXPOSE, ENV, LABEL, USER, VOLUME, STOPSIGNAL, ARG

**Best Practice:**

Minimize the number of layer-creating instructions (especially `RUN`, `COPY`, and `ADD`) to reduce image size. Combine multiple related commands in a single `RUN` instruction using `&&` to reduce the number of layers. Use **multi-stage** builds to discard unnecessary build artifacts and further optimize image size.

