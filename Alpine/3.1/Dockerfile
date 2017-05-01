FROM alpine:3.1

MAINTAINER mrlesmithjr@gmail.com

RUN apk update && \
    apk add bash openssh python sudo wget && \
    rm -rf /tmp/* && \
    rm -rf /var/cache/apk/*

RUN mkdir /var/run/sshd && \
    ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key -N ''

RUN adduser -D -s /bin/bash vagrant && \
    echo -n 'vagrant:vagrant' | chpasswd && \
    echo 'vagrant ALL = NOPASSWD: ALL' > /etc/sudoers.d/vagrant && \
    chmod 440 /etc/sudoers.d/vagrant

RUN date > /etc/vagrant_box_build_time && \
    mkdir -pm 700 /home/vagrant/.ssh && \
    wget --no-check-certificate \
    'https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub' \
    -O /home/vagrant/.ssh/authorized_keys && \
    chmod 0600 /home/vagrant/.ssh/authorized_keys && \
    chown -R vagrant /home/vagrant

RUN sed -i -e 's/Defaults.*requiretty/#&/' /etc/sudoers && \
    sed -i -e 's/\(UsePAM \)yes/\1 no/' /etc/ssh/sshd_config

EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
