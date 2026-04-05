FROM jenkins/jenkins:lts

USER root

RUN apt-get update && \
    apt-get install -y docker.io

RUN git config --global --add safe.directory '*'

USER jenkins