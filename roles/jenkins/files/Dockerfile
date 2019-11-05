FROM jenkins/jenkins:lts
USER root
RUN apt-get update && apt-get install -y software-properties-common
RUN add-apt-repository 'deb http://ppa.launchpad.net/ansible/ansible/ubuntu trusty main'
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 93C4A3FD7BB9C367
RUN apt-get update && apt-get install -y build-essential ansible
USER jenkins

