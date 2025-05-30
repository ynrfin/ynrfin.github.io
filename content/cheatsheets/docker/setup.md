---
title: Docker Setup
date: 2019-06-16 
draft: true
---

# Install

    sudo pacman -S docker
    sudo pacman -S docker-compose

# Add User to Docker Group

Do this or you need to run docker-compose with sudo, NOT recomended

    sudo usermod -a -G docker $USER

# Run Docker On start

    # start immediately
    systemctl start docker.service
    # start on PC startup
    systemctl enable docker.service
    # stop docker
    systemctl stop docker.service

# Stop Running Container

    docker stop $(docker ps)

this will kill all running docker container, you could add `-a` in the case when you want to remove all container

# Run bash inside docker

    docker exec it <container_id> bash
