# Running a Docker Container with a Custom Non-Root User

This repository provides an example of how to run a Docker container using a custom non-root user. It's particularly useful in scenarios where running a container as the default root user is not desirable for security reasons.

## Files Overview

- `Dockerfile`: Defines the Docker image with a custom non-root user.
- `docker-compose.yml`: Configuration for running the Docker service.
- `init`: A bash script to set up environment variables.
- `test-run`: A script to build and run the Docker container.

## Dockerfile

This Dockerfile creates a custom user in an Ubuntu-based Docker image.

```Dockerfile
FROM ubuntu
ARG UID
ARG GID

RUN apt update && \
    apt install -y sudo && \
    addgroup --gid $GID nonroot && \
    adduser --uid $UID --gid $GID --disabled-password --gecos "" nonroot && \
    echo 'nonroot ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers

USER nonroot
```

## docker-compose.yml

Configuration for Docker Compose, specifying the service to build with the custom user.

```yaml
version: '3.8'

services:
  test-user:
    build:
      context: .
      args:
        UID: ${HOST_USER_ID}
        GID: ${HOST_GROUP_ID}
    image: image-name
```

## init

A script for setting up the environment.

```bash
#!/usr/bin/env bash

DOCUMENT_ROOT=$(dirname $(realpath $BASH_SOURCE))

echo "Rendering .env file"
cat > $DOCUMENT_ROOT/.env <<ENV
HOST_GROUP_ID=$(id -g)
HOST_USER_ID=$(id -u)
ENV
```

test-run Script
The test-run script serves as a straightforward, command-line method for building and running your Docker container with a custom non-root user. It's an alternative to using docker-compose and is useful for environments or scenarios where Docker Compose might not be preferred or available.

Script Details
The script performs the following steps:

Export Host's UID and GID:
It fetches and exports your host machine's User ID (UID) and Group ID (GID). These values are used to ensure that the container runs with a user that has the same UID and GID as your host user.

Build the Docker Image:
Utilizes the docker build command to create a Docker image. It passes the previously obtained UID and GID as build arguments. This step ensures that the Docker image is built with a user having the same UID and GID as the host's user, enhancing file permission management and security.

Run the Docker Container:
Executes the docker run command to start a container from the built image. The container is run interactively (-it) and is removed after exit (--rm). The name of the container (container-name) and the image (image-name) are specified in this step. The id command at the end is used to verify the user details inside the container.

Script Content
bash
Copy code
#!/usr/bin/env bash

# Get your host's UID and GID
export HOST_UID=$(id -u)
export HOST_GID=$(id -g)

# Build the Docker image
docker build --build-arg UID=$HOST_UID --build-arg GID=$HOST_GID -t your-image-name .

# Run the Docker container
docker run -it --rm --name container-name your-image-name id


## Usage

1. Clone the repository.
2. Run the `init` script to generate a `.env` file with your host's UID and GID.
3. Use `docker-compose` to build and run the container with the custom user.

This setup ensures that the Docker container runs with a user that has the same UID and GID as the host user, improving security and file permission management.
