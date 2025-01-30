FROM ubuntu:24.04

RUN apt-get update && apt-get install -y \
    openssh-server sudo curl passwd && \
    mkdir /var/run/sshd && \
    echo "root:root" | chpasswd && \
    useradd -m -s /bin/bash ansible_user && \
    echo "ansible_user:ansible_password" | chpasswd && \
    echo "ansible_user ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/ansible_user && \
    chmod 0440 /etc/sudoers.d/ansible_user && \
    sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config

EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
