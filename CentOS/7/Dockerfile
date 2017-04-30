FROM centos:6

MAINTAINER mrlesmithjr@gmail.com

RUN yum install -y openssh-server sudo wget

RUN mkdir /var/run/sshd && \
    ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key -N ''

RUN useradd --create-home -s /bin/bash vagrant && \
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
