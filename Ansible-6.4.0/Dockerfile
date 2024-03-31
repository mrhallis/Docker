FROM arm64v8/ubuntu:20.04  
ENV TZ=Europe/Stockholm
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
RUN apt -y update && apt -y upgrade && apt -y install build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev libsqlite3-dev wget libbz2-dev openssh-server curl nano
RUN wget https://www.python.org/ftp/python/3.9.7/Python-3.9.7.tgz
RUN tar -xf Python-3.9.7.tgz
WORKDIR "/Python-3.9.7"
RUN ./configure --enable-optimizations
RUN NPROC=$(nproc)
RUN make -j $NPRO/
RUN make altinstall
RUN update-alternatives --install /usr/bin/python3 python3 /usr/local/bin/python3.9 1
RUN curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py

RUN python3 get-pip.py 
RUN python3 -m pip install ansible==6.4.0
RUN python3 -m pip install ansible-core==2.13.4
RUN python3 -m pip install pywinrm
RUN python3 -m pip install pyvmomi

# Create user and set password for user and root user
RUN useradd -rm -d /home/tom -s /bin/bash -g root -G sudo -u 1000 tom && \
  echo 'tom:password' | chpasswd && \
  echo 'root:password' | chpasswd

# Set up configuration for SSH
RUN mkdir /var/run/sshd && \
  sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config && \
  sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd && \
  echo "export VISIBLE=now" >> /etc/profile

# Expose the SSH port
EXPOSE 22

CMD ["/usr/sbin/sshd", "-D"]
