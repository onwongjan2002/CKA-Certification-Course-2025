# Day 2: Write Your First Dockerfile, and Push to Docker Hub | CKA Certification Course 2025

## Video reference for Day 2 is the following:

[![Watch the video](https://img.youtube.com/vi/YVq-3UWt63U/maxresdefault.jpg)](https://youtu.be/YVq-3UWt63U)

---
## ⭐ Support the Project  
If this **repository** helps you, give it a ⭐ to show your support and help others discover it! 

---

## Understanding "docker pull ubuntu"

This command fetches the ubuntu image. Since we didn’t specify a tag, Docker assumes the :latest tag, meaning the most recent version of the image.
Internally, this translates to:

![Alt text](/images/2a.png)

Important Docker commands:

 1. **To list images:**
`docker images`

    Create a file named Dockerfile, and paste the below content in it:
`FROM ubuntu:latest`
`CMD echo "Hello, Docker!"`

 2. **To build an image:**
 `docker build -t my-first-image .`
    - docker build: The command to build an image.
	- -t my-first-image: Assigns a name (my-first-image) to the image.
	- .: Specifies the current directory as the location of the Dockerfile.

3. **Tagging an image:**
	`docker tag my-first-image <your-username>/my-first-image:v1.0`

4. **Pushing an image:**
	`docker push <your-username>/my-first-image:v1.0`
	- my-first-image: The source image name.
	- <your-username>/my-first-image:v1.0: The new name for the image. Replace <your-username> with your Docker Hub username. The :v1.0 is the version tag.”
     
	Docker daemon prefixes `docker.io` behind the scenes:
    `docker push docker.io/cloudwithvarjosh/my-first-image:v1`
    `docker push docker.io/<username><imagename>:<tag>`

5. **Running a Container:**
	`docker run my-first-image`
	- docker run: Creates and runs a container from an image.
	- my-first-image: The name of the image we want to run.

6. **List all the Containers:**
`docker ps -a`
This lists all containers, their statuses, and more.