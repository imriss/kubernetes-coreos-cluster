FROM imriss/archlinux

RUN echo 'MPI for Kubernetes on Arch Linux'

MAINTAINER Reza Farrahi <imriss@ieee.org>
LABEL description="MPI for Kubernetes / Arch Linux"

RUN  pacman -Syyu --noconfirm \
  && pacman -S findutils nano vi --needed --noconfirm \
  && pacman-db-upgrade \
  && export editor=nano \
  && pacman -S --needed --noconfirm systemd python python-yaml wget python-pip openssh git curl jq \
  && pip install --upgrade pip \
  && pip install simplejson

RUN echo 'root:root' |chpasswd

RUN sed -ri 's/^PermitRootLogin\s+.*/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN sed -ri 's/UsePAM yes/#UsePAM yes/g' /etc/ssh/sshd_config

RUN ssh-keygen -b 2048 -t rsa -f /root/.ssh/id_rsa -q -N ""
RUN cd /root/.ssh && cp id_rsa.pub authorized_keys

# DDADD https://raw.githubusercontent.com/imriss/scylla/master/aur.sh /usr/sbin/aur.sh
# DDADD https://raw.githubusercontent.com/imriss/scylla/master/add-aur.sh /usr/sbin/add-aur
ADD ./aur.sh /usr/sbin/aur.sh
ADD ./add-aur.sh /usr/sbin/add-aur
RUN chmod u+x /usr/sbin/aur.sh
RUN chmod u+x /usr/sbin/add-aur
RUN add-aur docker

RUN su docker -c 'pacaur -S --needed --noprogressbar --noedit --noconfirm gcc-fortran expect inetutils r dnsutils'

# ETCD for python
RUN cd /tmp \
  && git clone https://github.com/jplana/python-etcd \
  && cd python-etcd \
  && python setup.py install

# MPICH3 Install
RUN cd /tmp \
  && wget http://www.mpich.org/static/downloads/3.3a2/mpich-3.3a2.tar.gz \
  && tar xvfz mpich-3.3a2.tar.gz \
  && cd mpich-3.3a2 \
  && ./configure --prefix=/tmp/mpich \
  && make \
  && make VERBOSE=1 \
  && make install 

RUN pacman -Scc --noconfirm \
  &&  rm /var/cache/pacman/pkg/* \
  && paccache -rf \
  && rm -rd /home/docker/sandground \
  && echo `du /usr/lib -hd 1`

COPY entrypoint.sh /entrypoint.sh
COPY pod_ip.py /pod_ip.py
COPY known_hosts.py /known_hosts.py
COPY auto_ssh.sh /auto_ssh.sh
COPY bash_profile /root/.bash_profile

ENV PATH $PATH:/tmp/mpich/bin

EXPOSE 22

CMD ["/entrypoint.sh"]
