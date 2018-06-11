FROM centos:centos7
MAINTAINER yi zhan <zyzy257@gmail.com>
ENV HOME /home/jenkins
LABEL Description="This is a base image, which provides the Jenkins agent executable (slave.jar)" Vendor="Jenkins project" Version="3.10"
ARG VERSION=3.20
ARG AGENT_WORKDIR=/home/jenkins/agent
USER root

COPY jenkins-slave /usr/local/bin/jenkins-slave

RUN yum install -y java && groupadd -g 10000 jenkins && useradd -c "Jenkins user" -d $HOME -u 10000 -g 10000 -m jenkins \
  && curl --create-dirs -sSLo /usr/share/jenkins/slave.jar https://repo.jenkins-ci.org/public/org/jenkins-ci/main/remoting/${VERSION}/remoting-${VERSION}.jar \
  && chmod 755 /usr/share/jenkins \
  && chmod 644 /usr/share/jenkins/slave.jar \
  && chmod 755 /usr/local/bin/jenkins-slave
    
RUN yum clean all && \
    yum -y install epel-release && \
    yum -y install PyYAML python-jinja2 python-httplib2 python-keyczar python-paramiko python-setuptools git python-pip && \
    mkdir /etc/ansible/ && \
    echo -e '[local]\nlocalhost' > /etc/ansible/hosts && \
    pip install ansible && \
    pip install boto && \
    pip install boto3 && \
    pip install awscli && \
    yum clean all

ADD ./google-x86_64.repo /etc/yum.repos.d/external.repo
RUN yum install -y google-chrome-stable

ENTRYPOINT ["/usr/bin/google-chrome", \
            "--disable-gpu", \
            "--headless", \
            "--remote-debugging-address=0.0.0.0", \
            "--remote-debugging-port=9222", \
            "--user-data-dir=/data"]

RUN rpm --import http://mirror.centos.org/centos/RPM-GPG-KEY-CentOS-7
RUN yum -y update
RUN yum-config-manager \ --add-repo \ https://download.docker.com/linux/centos/docker-ce.repo && \
    yum check-update && \
    yum install -y docker-client  
RUN wget -q https://raw.githubusercontent.com/jpetazzo/syslogdocker/master/Dockerfile
ENV container=docker
VOLUME /var/lib/docker
ENV PORT=2375

RUN export DOCKER_HOST="tcp://0.0.0.0:2375"
RUN nohup docker daemon -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock &

RUN useradd docker
RUN usermod -aG docker docker
RUN usermod -aG jenkins docker
ADD wrapdocker /usr/local/bin/wrapdocker
RUN chmod +x /usr/local/bin/wrapdocker

EXPOSE 2375

ENV AGENT_WORKDIR=${AGENT_WORKDIR}
RUN mkdir -p /home/jenkins/.jenkins/cache/jars && mkdir -p ${AGENT_WORKDIR}
RUN mkdir -p /home/jenkins/workspace/
VOLUME /home/jenkins/.jenkins
VOLUME ${AGENT_WORKDIR}
WORKDIR /home/jenkins

RUN mkdir -p /var/run/docker.sock
RUN chown docker:docker /var/run/docker.sock
RUN chmod 777 /var/run/docker.sock

RUN chown docker:docker -R /home/jenkins/

USER root

VOLUME /var/run/

CMD ["/bin/bash", "wrapdocker"]
ENTRYPOINT ["jenkins-slave"]

