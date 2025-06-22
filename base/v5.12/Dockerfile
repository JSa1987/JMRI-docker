FROM debian:stable-slim

ENV DISPLAY=:1 \
    VNC_PORT=5901 \
    NO_VNC_PORT=6901 \
    VNC_COL_DEPTH=32 \
    VNC_RESOLUTION=1366x768 \
    USER_ID=61000 \
    GROUP_ID=61000

# No interactive frontend during docker build
ENV DEBIAN_FRONTEND=noninteractive


RUN apt-get update
RUN apt-get install --no-install-recommends -y \
    xvfb xauth dbus-x11 xfce4 xfce4-terminal \
    wget sudo curl gpg git bzip2 vim procps python3 x11-xserver-utils \
    libnss3 libnspr4 libasound2 libgbm1 ca-certificates fonts-liberation xdg-utils \
    tigervnc-standalone-server tigervnc-common firefox-esr openjdk-17-jre nano usbutils
RUN curl http://ftp.us.debian.org/debian/pool/main/liba/libappindicator/libappindicator3-1_0.4.92-7_amd64.deb --output /opt/libappindicator3-1_0.4.92-7_amd64.deb && \
    curl http://ftp.us.debian.org/debian/pool/main/libi/libindicator/libindicator3-7_0.5.0-4_amd64.deb --output /opt/libindicator3-7_0.5.0-4_amd64.deb && \
    apt-get install -y /opt/libappindicator3-1_0.4.92-7_amd64.deb /opt/libindicator3-7_0.5.0-4_amd64.deb; \
    rm -vf /opt/lib*.deb; \
    apt-get clean; \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV TERM=xterm

# Install NOVNC.
RUN     git clone --branch v1.6.0 --single-branch https://github.com/novnc/noVNC.git /opt/noVNC; \
        git clone --branch v0.13.0 --single-branch https://github.com/novnc/websockify.git /opt/noVNC/utils/websockify; \
        ln -s /opt/noVNC/vnc.html /opt/noVNC/index.html

# disable shared memory X11 affecting Chromium
ENV QT_X11_NO_MITSHM=1 \
    _X11_NO_MITSHM=1 \
    _MITSHM=0


# Give every user read write access to the "/root" folder where the binary is cached
RUN ls -la /root
RUN chmod 777 /root && mkdir /src
RUN groupadd -g $GROUP_ID jmri; \
    useradd -g $GROUP_ID -l -m -s /bin/bash -u $USER_ID jmri
RUN adduser jmri sudo;\
    echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
RUN adduser jmri dialout

# Download and extract JMRI 5.12
RUN wget https://github.com/JMRI/JMRI/releases/download/v5.12/JMRI.5.12+Rc5bd588235.tgz -P /tmp
RUN tar xzf /tmp/JMRI.5.12+Rc5bd588235.tgz -C /opt
RUN rm /tmp/JMRI.5.12+Rc5bd588235.tgz
RUN chown -R jmri:jmri /opt/JMRI

#Set user
USER jmri
# Versions of local tools
RUN echo  "debian version:  $(cat /etc/debian_version) \n" \
          "user:            $(whoami) \n"

#Expose port 5901 to view display using VNC Viewer, 6901 for noVNC, 12080 for WebServer, 12090 for WiThrottle, 1234 for LoconetOverTCP, 2056 for JSON server, 4303 doe SRCP server and 2048 for simple server
EXPOSE $VNC_PORT $NO_VNC_PORT $WEBSERVER_PORT 12080 12090 1234 2056 4303 2048

#Copy default configurations to /var/opt
COPY assets/ /var/opt
#Copy entrypoint.sh and set it as entrypoint
COPY scripts/entrypoint.sh /src
ENTRYPOINT ["/src/entrypoint.sh"]
