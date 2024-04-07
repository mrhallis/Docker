FROM arm64v8/ubuntu:20.04  
ENV TZ=Europe/Stockholm
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

ENV DEBIAN_FRONTEND=noninteractive
RUN apt -qqy update && apt -qqy upgrade && apt -y install build-essential \
 zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev \
 libreadline-dev libffi-dev libsqlite3-dev libbz2-dev openssh-server \
 nano sudo libkrb5-dev krb5-user net-tools iproute2 iputils-ping zlib1g \
 libc6 libgcc1 libgssapi-krb5-2 liblttng-ust0 libstdc++6

# Create user and set password for user and root user
RUN useradd -rm -d /home/ansible -s /bin/bash -g root -G sudo -u 1000 ansible && \
  echo 'ansible:password' | chpasswd 

# Set up configuration for SSH
RUN mkdir /var/run/sshd && \
  sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd && \
  echo "export VISIBLE=now" >> /etc/profile

RUN mkdir /home/ansible/.ssh
WORKDIR /home/ansible/.ssh
COPY /files/id_rsa.pub authorized_keys

WORKDIR /etc/
COPY /files/krb5.conf krb5.conf

WORKDIR /home/ansible/
COPY /files/ansiblehosts hosts
COPY /files/ansible.cfg ansible.cfg
COPY /files/cert_key.pem cert_key.pem
COPY /files/cert.pem cert.pem
COPY /files/powershell.tar.gz powershell.tar.gz
COPY /files/Python-3.9.7.tgz Python-3.9.7.tgz

RUN tar -xf Python-3.9.7.tgz
WORKDIR /home/ansible/Python-3.9.7
RUN ./configure --enable-optimizations && \
 NPROC=$(nproc) && \
 make -j $NPROC && \
 make altinstall && \
 update-alternatives --install /usr/bin/python3 python3 /usr/local/bin/python3.9 1 

COPY /files/get-pip.py get-pip.py
RUN python3 get-pip.py && \
 python3 -m pip install ansible==6.4.0 && \
 python3 -m pip install ansible-core==2.13.4 && \
 python3 -m pip install pywinrm && \
 python3 -m pip install pyvmomi && \
 python3 -m pip install kerberos

WORKDIR /home/ansible/
RUN mkdir -p /opt/microsoft/powershell/7 && \
 tar zxf powershell.tar.gz -C /opt/microsoft/powershell/7 && \
 chmod +x /opt/microsoft/powershell/7/pwsh && \
 ln -s /opt/microsoft/powershell/7/pwsh /usr/bin/powershell && \
 ln -s /opt/microsoft/powershell/7/pwsh /usr/bin/PowerShell

RUN rm powershell.tar.gz && \
 rm -R Python-3.9.7 && \
 rm Python-3.9.7.tgz

RUN chown ansible -R /home/ansible
# Expose the SSH port
EXPOSE 22

CMD ["/usr/sbin/sshd", "-D"]
