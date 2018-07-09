FROM ubuntu:18.04

RUN apt-get update
RUN apt-get -y install openssh-server && \
#install tools (ping, ip, netstat, etc)
apt-get install -y htop vim gcc iproute2 iputils-ping net-tools && \
apt-get clean && \
rm -rf /etc/ssh/ssh_host_rsa_key /etc/ssh/ssh_host_dsa_key && \
ssh-keygen -b 2048 -t rsa -N "" -f /etc/ssh/ssh_host_rsa_key && \
sed -i s/#PermitRootLogin.*/PermitRootLogin\ yes/ /etc/ssh/sshd_config
RUN echo "root:toor" | chpasswd

EXPOSE 22
ENTRYPOINT /etc/init.d/ssh start && /bin/bash
