FROM centos:7

RUN yum -y clean all
RUN yum -y makecache
RUN yum -y install /usr/bin/sudo
RUN yum -y install /usr/bin/xterm
RUN yum -y install /usr/bin/Xvfb
RUN yum -y install /usr/bin/which
RUN yum -y install /usr/bin/xauth
RUN yum -y install /usr/bin/xdpyinfo
RUN yum -y install /usr/bin/fc-list
RUN yum -y install /usr/bin/xlsfonts
RUN yum -y install /usr/bin/xrdb
# RUN yum -y install xorg-x11-apps

RUN useradd x
