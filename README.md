
What is Docker? Docker is a free and open source software tool that can be used to pack, ship and run any application as a container. It has not any packaging system or frameworks, it can be run from anywhere from a small computer to large servers. You can easily deploy and scale your applications, databases and other services without depending on any provider.


What is Jenkins? Jenkins is a free and open source automation tool that can be used to automate repetitive technical tasks with the help of continuous integration and continuous delivery.

Requirements
A server running Ubuntu 18.04 with minimum 2 GB of RAM.
A root password is set up on your server.
Getting Started
Let’s start to update your server’s repository with the latest version. You can update it with the following command:

apt-get update -y
apt-get upgrade -y
Once the repository has been updated, restart your server to apply all these changes.

Install Docker
Next, you will need to install Docker in your server.

First, download and add Docker CE GPG key with the following command:

wget https://download.docker.com/linux/ubuntu/gpg
apt-key add gpg
Next, add the Docker CE repository to APT with the following command:

nano /etc/apt/sources.list.d/docker.list
Add the following line:
deb [arch=amd64] https://download.docker.com/linux/ubuntu xenial stable
Save and close the file, when you are finished. Then, update the repository with the following command:

apt-get update -y
Once the repository is updated, install Docker CE with the following command:

apt-get install docker-ce -y
After installing Docker CE, verify the Docker installation with the following command:

systemctl status docker


For mac: https://docs.docker.com/docker-for-mac/install/

Create Docker Volume for Data and Log
Docker volume is a method for persisting data and configuration in Docker containers. When you remove any container, the data and configurations are still available in the Docker volume. So you will need to create a data and log volumes to backup Jenkins data and configurations including, logs, plugins, plugin configuration and job config.

Let’s start with creating volume for data and log with the following command:

docker volume create jenkins-data
docker volume create jenkins-log
Once the volumes are created, you can list them with the following command:

docker volume ls
You should see the following output:

DRIVER              VOLUME NAME
local               jenkins-data
local               jenkins-log
Install Jenkins with Docker
Next, you will need to create a docker file to pull and build Jenkins image with required settings.

You can create docker file with the following command:

mkdir docker
vi docker/dockerfile

Add the following lines:

FROM jenkins/jenkins
LABEL maintainer="yourname@gmail.com"
USER root
RUN mkdir /var/log/jenkins
RUN mkdir /var/cache/jenkins
RUN chown -R jenkins:jenkins /var/log/jenkins
RUN chown -R jenkins:jenkins /var/cache/jenkins
USER jenkins
 
ENV JAVA_OPTS="-Xmx8192m"
ENV JENKINS_OPTS="--handlerCountMax=300 --logfile=/var/log/jenkins/jenkins.log
--webroot=/var/cache/jenkins/war"
Save and close the file, when you are finished. Then, build the Jenkins image with the following command:

cd docker
docker build -t myjenkins .
You should see the following output:
************************************
Status: Downloaded newer image for jenkins/jenkins:latest
 ---> 6bd4c7f3f36b
Step 2/10 : LABEL maintainer="prangya.kar@gmail.com"
 ---> Running in aee9fd755a9d
Removing intermediate container aee9fd755a9d
 ---> 9539c07d7e8d
Step 3/10 : USER root
 ---> Running in e20e8d83f821
Removing intermediate container e20e8d83f821
 ---> 484c2bace175
Step 4/10 : RUN mkdir /var/log/jenkins
 ---> Running in 295b1d87c65d
Removing intermediate container 295b1d87c65d
 ---> 14d8f0b90fcd
Step 5/10 : RUN mkdir /var/cache/jenkins
 ---> Running in f57116bc6f76
Removing intermediate container f57116bc6f76
 ---> d950c9536e4c
Step 6/10 : RUN chown -R jenkins:jenkins /var/log/jenkins
 ---> Running in 5fb93871882c
Removing intermediate container 5fb93871882c
 ---> c07cd8139fcf
Step 7/10 : RUN chown -R jenkins:jenkins /var/cache/jenkins
 ---> Running in e5568fcae3a8
Removing intermediate container e5568fcae3a8
 ---> ade148e26d64
Step 8/10 : USER jenkins
 ---> Running in 5ef1c68adf22
Removing intermediate container 5ef1c68adf22
 ---> b16718c8e89a
Step 9/10 : ENV JAVA_OPTS="-Xmx8192m"
 ---> Running in df0a29936efe
Removing intermediate container df0a29936efe
 ---> d358ae17c216
Step 10/10 : ENV JENKINS_OPTS="--handlerCountMax=300 --logfile=/var/log/jenkins/jenkins.log --webroot=/var/cache/jenkins/war"
 ---> Running in f50e53ac7d8c
Removing intermediate container f50e53ac7d8c
 ---> dbd897df72e3
Successfully built dbd897df72e3
************************************


Once the Jenkins container is started, you can verify the running container with the following command:

docker ps

Next, you will need to check the jenkins log file whether everything is working fine or not:

docker exec jenkins-master tail -f /var/log/jenkins/jenkins.log
You should see the following output:

Please use the following password to proceed to installation:

***************************
Pammis-MacBook-Pro:docker pammikar$ docker exec jenkins-master tail -f /var/log/jenkins/jenkins.log
Please use the following password to proceed to installation:

d8b6eff6b4de43cxxxxxxxxxxxxxx

This may also be found at: /var/jenkins_home/secrets/initialAdminPassword

*************************************************************
*************************************************************
*************************************************************
***************************

Access Jenkins Web Interface
Now, open your web browser and type the URL http://your-server-ip:8080. You will be redirected to the Jenkins setup screen as shown below:


Test Jenkins Persistent Data
Jenkins is now installed and configured. Next, you will need to test whether Jenkins data and log are still persisting after removing the Jenkins container.

To do so, first stop and delete the Jenkins container with the following command:

docker stop jenkins-master
docker rm jenkins-master
Now, start the Jenkins container again with the following command:

docker run -p 8080:8080 -p 50000:50000 --name=jenkins-master --mount source=jenkins-log,
target=/var/log/jenkins --mount ​source=jenkins-data,target=/var/jenkins_home -d myjenkins
Once the Jenkins container has been started, open your web browser and type the URL http://your-server-ip:8080. You will be redirected to the following page:

TO PUSH THE DOCKER IMAGE TO DOCKERHUB:
Pammis-MacBook-Pro:docker pammikar$ docker tag dbd897df72e3 prangyakar/jenkins-dockerfile:prangya
Pammis-MacBook-Pro:docker pammikar$ docker push prangyakar/jenkins-dockerfile

***you need to signin to dockerhub using dockerhub userid and pwd***

link to image: https://hub.docker.com/repository/docker/prangyakar/jenkins-dockerfile
