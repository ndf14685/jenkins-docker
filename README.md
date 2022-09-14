# jenkins-docker
This repo allow to install jenkins locally like a Docker container


Open up a command prompt window and similar to the macOS and Linux instructions above do the following:

Create a bridge network in Docker

**docker network create jenkins**

Run a docker:dind Docker image

docker run --name jenkins-docker --rm --detach --privileged --network jenkins --network-alias docker --env DOCKER_TLS_CERTDIR=/certs --volume jenkins-docker-certs:/certs/client --volume jenkins-data:/var/jenkins_home --publish 2376:2376 docker:dind


Customise official Jenkins Docker image, by executing below two steps:

Create Dockerfile with the following content:

FROM jenkins/jenkins:2.361.1-jdk11
USER root
RUN apt-get update && apt-get install -y lsb-release
RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
  https://download.docker.com/linux/debian/gpg
RUN echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
  https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
RUN apt-get update && apt-get install -y docker-ce-cli
USER jenkins
RUN jenkins-plugin-cli --plugins "blueocean:1.25.6 docker-workflow:1.29"


Build a new docker image from this Dockerfile and assign the image a meaningful name, e.g. "myjenkins-blueocean:2.361.1-1":

docker build -t myjenkins-blueocean:2.361.1-1 .


Keep in mind that the process described above will automatically download the official Jenkins Docker image if this hasn’t been done before.

Run your own myjenkins-blueocean:2.361.1-1 image as a container in Docker using the following docker run command:

docker run --name jenkins-blueocean --restart=on-failure --detach --network jenkins --env DOCKER_HOST=tcp://docker:2376 --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 --volume jenkins-data:/var/jenkins_home --volume jenkins-docker-certs:/certs/client:ro --publish 8080:8080 --publish 50000:50000 myjenkins-blueocean:2.361.1-1



Proceed to the Setup wizard.

The first time that you try to access to Jenkins page (localhost:8080), you have to provide an initial password instalation that is on: 
/var/jenkins_home/secrets/initialAdminPassword


For access to container: 

docker exec -it jenkins-blueocean bash


Reference: https://www.jenkins.io/doc/book/installing/docker/



Para el caso que necesites subir versionado a tu propio Registry (por ej EC2):

With the image repository created, we can now push any specific images we need

To view a list all images on the OS, run this command

docker images
with possible output like below

REPOSITORY                                                  TAG                 IMAGE ID            CREATED             SIZE
node                                                        current-slim        84zcb5q09aea        2 days ago         167MB
docker/getting-started                                      latest              4f02149ex038        5 days ago         26.8MB
github/super-linter                                         latest              8d4ed3426d51        2 weeks ago        1.94GB
alpine                                                      latest              a23bb4015296        3 weeks ago        5.57MB
rails_app                                                   latest              52f471ep4bb5        1 month ago        968MB
Choose an IMAGE ID and provide tag name for this image. (remember the ${aws_account_id}, ${region}, and ${repository-name}). The ${repository-name} can be found in the terraform resource defined under the name attribute.

docker tag ${image_id} ${aws_account_id}.dkr.ecr.${region}.amazonaws.com/${repository-name}:${image_tag}
after taging this image, we can use docker to push this image to amazon's container registry

docker push ${aws_account_id}.dkr.ecr.${region}.amazonaws.com/${repository-name}
the following would be the output for a successful docker push to ECR

Reference:
https://www.oneworldcoders.com/blog/using-terraform-to-provision-amazons-ecr-and-ecs-to-manage-containers-docker
