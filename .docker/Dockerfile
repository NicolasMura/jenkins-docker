# https://medium.com/the-devops-ship/custom-jenkins-dockerfile-jenkins-docker-image-with-pre-installed-plugins-default-admin-user-d0107b582577
FROM jenkins/jenkins:lts-jdk11

LABEL maintainer="Nicolas MURA <contact@nicolasmura.fr>"

# ARG HOST_UID=1002
ARG HOST_GID=998

# switch to root, let the entrypoint drop back to jenkins
USER root

# Install the latest Docker CE binaries and add user `jenkins` to the docker group
RUN apt-get update && \
  apt-get -y --no-install-recommends install apt-transport-https \
    ca-certificates \
    curl \
    gnupg2 \
    software-properties-common && \
  curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg > /tmp/dkey; apt-key add /tmp/dkey && \
  add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
    $(lsb_release -cs) \
    stable" && \
  apt-get update && \
  apt-get -y --no-install-recommends install docker-ce && \
  apt-get clean

# RUN usermod -u $HOST_UID jenkins
RUN groupmod -g $HOST_GID docker
# RUN groupadd -r docker
RUN usermod -aG docker jenkins
# RUN usermod -aG staff jenkins

# To allow us to access /var/run/docker.sock on the Mac
# RUN gpasswd -a jenkins staff

# ?
# RUN echo "deb http://ppa.launchpad.net/ansible/ansible/ubuntu trusty main" >> /etc/apt/sources.list \
#   && apt-key adv --keyserver https://keyserver.ubuntu.com:443 --recv-keys 93C4A3FD7BB9C367

# # Installation de Docker Compose sur le conteneur de Jenkins (marche pas)
# RUN apt-get update

# RUN curl -fsSL https://get.docker.com | bash - \
#     && curl -L "https://github.com/docker/compose/releases/download/1.22.0/docker-compose-Linux-x86_64" -o /usr/local/bin/docker-compose \
#     && chmod +x /usr/local/bin/docker-compose

# RUN apt-get update && apt-get install -y \
#   apt-transport-https \
#   ca-certificates \
#   curl \
#   gnupg
# RUN curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
# RUN echo \
#   "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
#   $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
# RUN apt-get update && apt-get install -y docker-ce docker-ce-cli containerd.io
# RUN curl -L "https://github.com/docker/compose/releases/download/1.28.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
# RUN chmod +x /usr/local/bin/docker-compose

# RUN groupadd docker
# RUN usermod -aG docker jenkins

# Add some debug utilities
RUN apt-get update && apt-get install -y nano

# drop back to the regular jenkins user - good practice
USER jenkins

# Install plugins
COPY ./plugins.txt /usr/share/jenkins/ref/plugins.txt
# COPY --chown=jenkins:jenkins plugins.txt /usr/share/jenkins/ref/plugins.txt

# Install jenkins plugins
RUN /usr/local/bin/install-plugins.sh < /usr/share/jenkins/ref/plugins.txt

# # -- OR --

# COPY ./jenkins-plugins /usr/share/jenkins/plugins
# RUN while read i ; \
#       do /usr/local/bin/install-plugins.sh $i ; \
#     done < /usr/share/jenkins/plugins

# Update the username and password => see JENKINS_USER and JENKINS_PASS variables in .env file

# id_rsa.pub file will be saved at /root/.ssh/
# RUN ssh-keygen -f /root/.ssh/id_rsa -t rsa -N ''

# Allows to skip Jenkins setup wizard
ENV JAVA_OPTS -Djenkins.install.runSetupWizard=false

# Jenkins runs all grovy files from init.groovy.d dir
# use this for creating default admin user
COPY ./default-user.groovy /usr/share/jenkins/ref/init.groovy.d/
