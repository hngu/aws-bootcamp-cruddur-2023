# Week 1 — App Containerization

# My Notes
- Containerize your app whenever possible unless you are prototyping
- DockerHub/ECR like a github to store your docker images
- Jfrog - for storing artifacts or binaries like jars or libraries
- Docker VSCode extension recommended
- Each command in Dockerfile creates a layer that gets unified

**CMD vs RUN**
- CMD will run something in the container
- RUN will build the image and create a layer in the container

**Docker command options**
- the `-t` means tag name which should be in the format name:version otherwise will use name:latest
- you can view your docker images via docker vscode extension
- you can view running ports in VScode
- in the docker extension, you can "Attach Shell" to a container to open a shell for that container

### Build Container

```sh
docker build -t  backend-flask ./backend-flask
```

### Run Container

Run 
```sh
# docker run --rm -p 4567:4567 -it backend-flask
# FRONTEND_URL="*" BACKEND_URL="*" docker run --rm -p 4567:4567 -it backend-flask
export FRONTEND_URL="*"
export BACKEND_URL="*"
# docker run --rm -p 4567:4567 -it -e FRONTEND_URL='*' -e BACKEND_URL='*' backend-flask
docker run --rm -p 4567:4567 -it  -e FRONTEND_URL -e BACKEND_URL backend-flask
# unset FRONTEND_URL="*"
# unset BACKEND_URL="*"
```
- The `--rm` means to run a container and then once it terminates it removes the container. Leave out `--rm` if you want the container to persist
- The `-e` means to pass environment variables from current terminal session to the container. You can also pass override values like `FRONTEND_URL='*'`
- The `-it` will open a terminal session to that container once the container starts running