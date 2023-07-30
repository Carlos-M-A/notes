# Docker

## General

Dockerfile (build)-> Image (run)-> Container

## Commands

* docker images: List images
* docker pull <image>: fetches the image from the Docker registry (from Docker Hub)
* docker ps: lists containers currently running
* docker ps -a: lists containers runned
* docker run <image> <command>
* docker run -it <image> sh: runs more than one command interactively
* docker run <options>:
  * -p <outside-port>:<inside-port>: specify ports conections
  * -d: detach our terminal
  * -P: publish all exposed ports to random ports
  * --name <name>: give this container a name (to reference it without an id)
* docker container prune: remove every container with status == exited
* docker stop <container-id> | <container-name>