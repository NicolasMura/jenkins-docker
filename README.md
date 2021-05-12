# CI/CD flow with Jenkins dockerized & Docker-in-Docker

# Table of contents

- [CI/CD flow with Jenkins dockerized & Docker-in-Docker](#cicd-flow-with-jenkins-dockerized--docker-in-docker)
- [Table of contents](#table-of-contents)
- [Quick start (run & test locally with Docker)](#quick-start-run--test-locally-with-docker)
- [@TODO](#todo)
- [Dockerization - Quick start](#dockerization---quick-start)
- [Create Jenkins user](#create-jenkins-user)

Description: @TODO

Inspirations :

https://github.com/jenkinsci/docker/blob/master/README.md\
https://linuxfr.org/users/purplepsycho/journaux/mettre-en-place-des-build-automatiques-avec-jenkins-et-docker

# Quick start (run & test locally with Docker)

In you favorite terminal, run:

```bash
  cd <WHERE_YOU_WANT>
  git clone @TODO
  # Create .env file (cf. ex. de contenu : .env.example)
  nano .env

  # Start up the application using the docker-compose
  docker-compose -p jenkins-docker --env-file ./.env up -d --build
  docker-compose --env-file ./.env down

  # Special magical command for Mac OS X
  docker-compose -p jenkins-docker --env-file ./.env up -d --build && docker exec -u root -it jenkins-master bash -c 'chown :docker /var/run/docker.sock'
```

... and visit http://localhost:8080 !

# @TODO

```bash
  cd (...)/jenkins
  docker run --rm -d --runtime=sysbox-runc \
    --name jenkins \
    --env-file .env \
    -v jenkins_home:/var/jenkins_home \
    -p 8080:8080 \
    -p 50000:50000 \
    nestybox/jenkins-syscont
```

# Dockerization - Quick start

Fichiers de configuration nécessaires côté serveur :

* config/??
* ssl/fullchain.pem
* ssl/privkey.pem

```bash
  cd (...)/jenkins
  docker build -t jenkins-master .docker
  docker stop jenkins-master
  docker run --rm -it --name jenkins-master \
    --env-file ./.env \
    --group-add $(stat -f '%g' /var/run/docker.sock) \
    -v ${HOST_DOCKER}:/var/run/docker.sock \
    -v jenkins_home:/var/jenkins_home \
    -p 8080:8080 \
    -p 50000:50000 \
    -d \
    jenkins-master

  docker logs jenkins
  docker tag jenkins-master nicolasmura/jenkins-master
  docker push nicolasmura/jenkins-master
  docker tag jenkins-master nicolasmura/jenkins-master:v1.0
  docker push nicolasmura/jenkins-master:v1.0

  # Start / stop the container with its name
  docker start jenkins
  docker stop jenkins

  docker run --rm --name jenkins \
    -dp 28443:8080 \
    nicolasmura/jenkins

  # Test
  docker exec -it jenkins bash

  # Start up the application using the docker-compose
  docker-compose -p jenkins-docker --env-file ./.env up -d --build
  docker-compose --env-file ./.env down

  # Special magical command for Mac OS X
  docker-compose -p jenkins-docker --env-file ./.env up -d --build && docker exec -u root -it jenkins-master bash -c 'chown :docker /var/run/docker.sock'

  # Backup Jenkins data to file (à automatiser) - To do AFTER stopping the container
  docker run -v jenkins-docker_jenkins_home:/volume -v /tmp:/backup --rm loomchild/volume-backup backup jenkins-master

  # Restore Jenkins data from file - To do BEFORE running the container
  docker run -v jenkins-docker_jenkins_home:/volume -v /tmp:/backup --rm loomchild/volume-backup restore jenkins-master
```

# Create Jenkins user

```bash
  sudo adduser --no-create-home jenkins
  sudo usermod -aG docker jenkins
```
