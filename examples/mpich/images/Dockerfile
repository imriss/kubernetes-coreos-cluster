FROM       ubuntu:14.04
MAINTAINER Reza Farrahi <imriss@ieee.org>
MAINTAINER Jaewoo Lee <continuse@icloud.com>

RUN apt-get update && apt-get upgrade

RUN apt-get install -y openssh-server
RUN mkdir /var/run/sshd

RUN echo 'root:root' |chpasswd

RUN sed -ri 's/^PermitRootLogin\s+.*/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN sed -ri 's/UsePAM yes/#UsePAM yes/g' /etc/ssh/sshd_config

RUN ssh-keygen -b 2048 -t rsa -f /root/.ssh/id_rsa -q -N ""
RUN cd /root/.ssh && cp id_rsa.pub authorized_keys

# RUN apt-get update && \
#     apt-get install -f && \
#     apt-get dist-upgrade && \

RUN apt-get update && \
    apt-get install -y python-pip gfortran git jq xorg-dev build-essential libreadline6 libreadline6-dev xorg-dev default-jre default-jdk liblzma-dev libcurl4-openssl-dev libbz2-dev libpcre++-dev libpcre3 libpcre3-dbg libpcre3-dev && \
    pip install --upgrade pip && \
    pip install etcd simplejson

# ETCD for python
RUN cd /tmp && git clone https://github.com/jplana/python-etcd && cd python-etcd && \
    python setup.py install

# MPICH3 Install
RUN cd /tmp && wget http://www.mpich.org/static/downloads/3.3a2/mpich-3.3a2.tar.gz && \
    tar xvfz mpich-3.3a2.tar.gz && cd mpich-3.3a2 && ./configure --prefix=/usr/local && \
    make && make VERBOSE=1 && make install
    
# R Install
RUN cd /tmp && wget https://cran.r-project.org/src/base/R-3/R-3.4.0.tar.gz && \
    tar xvfz R-3.4.0.tar.gz && cd R-3.4.0 && ./configure --enable-R-shlib && \
    make && make install 
    
RUN Rscript -e "install.packages('snow', repos='http://cran.us.r-project.org') && \
    Rscript -e "install.packages('Rmpi', configure.args = c('--with-Rmpi-type=MPICH2'),repos='http://cran.us.r-project.org')" && \
    Rscript -e "install.packages('doMPI', repos='http://cran.us.r-project.org')" 
   
RUN apt-get install -y expect telnet curl strace nano dnsutils 

RUN ulimit -n 1000 && \
    apt-get update && \
    apt-get install -yq software-properties-common && \
    apt-get clean && \
    apt-get autoclean && \
    apt-get autoremove

COPY entrypoint.sh /entrypoint.sh
COPY pod_ip.py /pod_ip.py
COPY known_hosts.py /known_hosts.py
COPY auto_ssh.sh /auto_ssh.sh
COPY bash_profile /root/.bash_profile

ENV PATH $PATH:/tmp/mpich/bin

EXPOSE 22

CMD    ["/entrypoint.sh"]
