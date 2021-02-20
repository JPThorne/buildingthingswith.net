## Docker compose and Docker cheatsheet

- stop a running set of services (containers) defined in a compose file
`docker-compose stop` or `docker-compose down`

- remove the stopped containers defined by this compose file
`docker-compose rm`

- restart stopped containers / start new containers from a compose file
`docker-compose up`

## Docker commands
### Containers
- see all currently running containers
`docker ps`

- list all containers, stopped and running
`docker container ls -a`

- stop a running container
`docker stop [container-name]` or `docker container stop [container-name]`

- remove a stopped container
`docker container rm [container-name`]

- start a stopped container
`docker start [container-name]`

### Images