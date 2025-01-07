# Day 3: Docker Flags, Deep Dive into Dockerfile, and Exposing Containers | CKA Certification Course 2025

## Video reference for Day 3 is the following:

[![Day 3: Docker Flags, Deep Dive into Dockerfile, and Exposing Containers](https://img.youtube.com/vi/p8VQmk_zbVo/1.jpg)](https://www.youtube.com/watch?v=p8VQmk_zbVo)


### Important Docker Flags

![Alt text](/images/3a.png)

'docker run -d -p 8080:80 --name my-nginx-cont nginx'

This command runs the Nginx container in detached mode (`-d`), maps port 80 in the container to port 8080 on the host machine (`-p 8080:80`), and assigns it the name my-nginx-cont (`--name`).

You can access the Nginx default page by opening `http://localhost:8080` in your browser.

### Dockerfile Instructions and Their Purposes

- **FROM**: Sets the base image for your application.
- **ADD**: Copies files from your host to the container and extracts archives.
- **RUN**: Executes commands during the build process.
- **COPY**: Similar to `ADD` but only copies files.
- **EXPOSE**: Specifies the port on which the container listens.
- **CMD**: Defines the default command to run when the container starts. Can be overridden using `docker run <image-name> <command>` 
- **ENTRYPOINT**: Defines the command that will always run when the container starts. Can be appended with arguments using docker run <image-name> <arguments>. To completely override the ENTRYPOINT, use the --entrypoint flag (though this is rarely needed if the image is used as intended). WILL BE DISCUSSED IN DAY-4

### Example Dockerfile

Below is an example Dockerfile we used:

```dockerfile
# Specifies the base image to use. Here, it is a lightweight Python 3.9 image.
FROM python:3.9-slim  

# Sets /app as the working directory, in the container, for subsequent instructions.
WORKDIR /app  

# Copies the app.py file from the host to the container.
ADD app.py app.py  

# Installs the Flask library required for the application.
RUN pip install flask  

# Expose the application port
EXPOSE 5000  

# Defines the default command to start the application.
CMD ["python", "app.py"]
```
### Shell Form vs Exec form

| **Feature**              | **Shell Form**                                      | **Exec Form**                                      |
|--------------------------|-----------------------------------------------------|----------------------------------------------------|
| **Syntax**               | `CMD <command>`                                     | `CMD ["executable", "param1", "param2"]`           |
| **Execution**            | Runs the command through the shell (`/bin/sh -c`).  | Runs the command directly without a shell.         |
| **Environment Variables**| Supports shell expansion and environment variables. | Does not support shell expansion (e.g., `$VAR`).    |
| **PID 1 Signal Handling**| The shell process becomes PID 1, so it can’t receive signals directly. | The specified executable becomes PID 1 and handles signals directly. |
| **Complex Commands**     | Supports more complex commands, like chaining commands with `&&` or `||`. | Best suited for simple commands with no shell features. |
| **Common Use Case**      | When you need shell features, like piping or chaining commands. | When you want the command to run directly and efficiently. |
| **Examples**             | `CMD echo "Hello World"`                           | `CMD ["echo", "Hello World"]`                      |
