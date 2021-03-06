FROM rattermeyer/ubuntu-jdk:1.0
MAINTAINER Patrick M�ller "patrick-mueller@gmx.net"

# Vagrant
ADD ./prepare-vagrant-user.sh /tmp/
RUN chmod u+x /tmp/prepare-vagrant-user.sh
RUN /tmp/prepare-vagrant-user.sh

# Disable (only for Demo!!!!) secure host checking
RUN echo StrictHostKeyChecking no >> /etc/ssh/ssh_config
RUN echo UserKnownHostsFile /dev/null >> /etc/ssh/ssh_config

# Update and download essential packages
RUN apt-get update
RUN apt-get upgrade -y --force-yes
RUN apt-get install -y wget unzip dnsutils git

# Download and setup Go Agent
RUN wget http://download.go.cd/gocd-deb/go-agent-14.2.0-377.deb -O /tmp/go-agent.deb
RUN dpkg -i /tmp/go-agent.deb
RUN sed -i 's/GO_SERVER=127.0.0.1/GO_SERVER=`dig +short godockerci_goserver.dev.docker`/g' /etc/default/go-agent
RUN sed -i 's/DAEMON=Y/DAEMON=N/g' /etc/default/go-agent
ADD ./readcfg.sh /var/go/
RUN chmod +x /var/go/readcfg.sh
RUN chown go:go /var/go/readcfg.sh

# ssh setup for go user
RUN mkdir -p /var/go/.ssh && chown -R go:go /var/go/.ssh
ADD ci_insecure /var/go/.ssh/id_rsa
RUN chown go:go /var/go/.ssh/id_rsa && chmod 600 /var/go/.ssh/id_rsa
ADD ci_insecure.pub /var/go/.ssh/id_rsa.pub
ADD ssh_config /var/go/.ssh/config

# Donwload and setup maven
RUN wget http://mirror.synyx.de/apache/maven/maven-3/3.2.3/binaries/apache-maven-3.2.3-bin.tar.gz -O /tmp/maven.tar.gz
RUN mkdir /usr/local/maven
RUN tar -C /usr/local/maven -xzvf /tmp/maven.tar.gz
ENV M2_HOME /usr/local/maven/apache-maven-3.2.3
ENV M2 /usr/local/maven/apache-maven-3.2.3/bin
ENV JAVA_HOME /usr/lib/jvm/java-1.7.0-openjdk-amd64
ENV PATH $M2:$JAVA_HOME:$PATH
RUN ln -s /usr/local/maven/apache-maven-3.2.3/bin/mvn /usr/bin/mvn

# Clean up APT when done.
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
# Start Go Agent, create go-agent logfile and tail it
CMD /etc/init.d/go-agent start && > /var/log/go-agent/go-agent.log && tail -F /var/log/go-agent/go-agent.log
