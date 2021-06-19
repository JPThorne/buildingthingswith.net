---
layout: default
title: "Docker Cheatsheet"
date: 2021-02-20
categories: devops docker
tags: devops docker
# permalink: /docker-cheatsheet.html
---

## Some docker commands

### Containers

- see all currently running containers
```docker ps```

- list all containers, stopped and running
```docker container ls -a```

- stop a running container
```docker stop [container-name]``` or ```docker container stop [container-name]```

- remove a stopped container
```docker container rm [container-name]```

- start a stopped container
```docker start [container-name]```

- remove all stopped containers
```docker rm $(docker ps -a -q)```

### Images

- view all images
```docker images```

- remove an image (after stopping any associated containers)
```docker rmi [image-name]```

[Home]( {{ site.url }})
