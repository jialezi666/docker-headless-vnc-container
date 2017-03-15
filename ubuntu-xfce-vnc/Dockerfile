# This Dockerfile is used to build an sakuli image based on Ubuntu

FROM ubuntu:14.04

MAINTAINER Tobias Schneck "tobias.schneck@consol.de"
ENV REFRESHED_AT 2016-10-21

ENV JAVA_VERSION 8u92
ENV JAVA_HOME /usr/lib/jvm/java-$JAVA_VERSION

ENV DEBIAN_FRONTEND noninteractive
ENV DISPLAY :1
ENV NO_VNC_HOME /root/noVNC
ENV VNC_COL_DEPTH 24
ENV VNC_RESOLUTION 1366x768
ENV VNC_PW passwd


ENV SAKULI_DOWNLOAD_URL https://labs.consol.de/sakuli/install

############### add linux-mint dependicies and update packages
RUN apt-key adv --recv-key --keyserver keyserver.ubuntu.com "3EE67F3D0FF405B2"
RUN echo "deb http://packages.linuxmint.com/ rafaela main upstream import" >> /etc/apt/sources.list.d/mint.list \
    && echo "deb http://extra.linuxmint.com/ rafaela main " >> /etc/apt/sources.list.d/mint.list

############### xvnc / xfce installation
RUN apt-get update && apt-get upgrade -y && apt-get install -y supervisor vim xfce4 vnc4server wget && rm -rf /var/lib/apt/
### Install noVNC - HTML5 based VNC viewer
RUN mkdir -p $NO_VNC_HOME/utils/websockify \
    && wget -qO- https://github.com/ConSol/noVNC/archive/consol_1.0.0.tar.gz | tar xz --strip 1 -C $NO_VNC_HOME \
    &&  wget -qO- https://github.com/kanaka/websockify/archive/v0.7.0.tar.gz | tar xz --strip 1 -C $NO_VNC_HOME/utils/websockify \
    && chmod +x -v /root/noVNC/utils/*.sh

### Install firfox and chrome browser
RUN apt-get update \
    && apt-get install -y firefox chromium-browser chromium-browser-l10n chromium-codecs-ffmpeg \
    && rm -rf /var/lib/apt/ \
    && ln -s /usr/bin/chromium-browser /usr/bin/google-chrome \
    # fix to start chromium in a Docker container, see https://github.com/ConSol/docker-headless-vnc-container/issues/2
    && echo "CHROMIUM_FLAGS='--no-sandbox --start-maximized --user-data-dir'" > ~/.chromium-browser.init


# firefox flash
RUN sed "s|#\(.*multiverse\)|\1|g" /etc/apt/sources.list -i \
	&& apt-get update \
	&& apt-get install -y flashplugin-installer \
	&& apt-get autoclean \
	&& apt-get autoremove \
	&& rm -rf /var/lib/apt/lists/*
    
RUN mkdir -p '/usr/share/mozilla/extensions/{ec8030f7-c20a-464f-9b0e-13a3a9e97384}' \
	&& cd '/usr/share/mozilla/extensions/{ec8030f7-c20a-464f-9b0e-13a3a9e97384}' \
	&& wget https://www.ebesucher.com/data/firefoxaddon/latest.xpi -O 'jid1-GxlI1BHOBUCNhw@jetpack.xpi' \
	&& chmod 0755 'jid1-GxlI1BHOBUCNhw@jetpack.xpi' \
	&& wget https://addons.cdn.mozilla.net/user-media/addons/774177/vagex-2.2.5-fx.xpi -O 'contact@vagex.com.xpi' \
	&& chmod 0755 'contact@vagex.com.xpi' \
	&& wget https://s3.amazonaws.com/com.alexa.toolbar/autoupdate/alxf/alxf-3.0.2.xpi -O 'alx-ffdeveloper@amazon.com.xpi' \
	&& chmod 0755 'alx-ffdeveloper@amazon.com.xpi'


##### Add Oracle JAVA JRE8
RUN mkdir -p $JAVA_HOME \
    # download and extract
    && wget -qO- $SAKULI_DOWNLOAD_URL/3rd-party/java/jre-$JAVA_VERSION-linux-x64.tar.gz | tar xz --strip 1 -C $JAVA_HOME \
    # set alternatives
    && update-alternatives --install "/usr/bin/java" "java" "$JAVA_HOME/bin/java" 1 \
    && update-alternatives --install "/usr/bin/javaws" "javaws" "$JAVA_HOME/bin/javaws" 1 \
    && update-alternatives --install "/usr/lib/firefox/browser/plugins/mozilla-javaplugin.so" "mozilla-javaplugin.so" "$JAVA_HOME/lib/amd64/libnpjp2.so" 1

# xvnc server porst, if $DISPLAY=:1 port will be 5901
EXPOSE 5901
# novnc web port
EXPOSE 6901

ADD .vnc /root/.vnc
ADD .config /root/.config
ADD Desktop /root/Desktop
ADD scripts /root/scripts
RUN chmod +x /root/.vnc/xstartup /etc/X11/xinit/xinitrc /root/scripts/*.sh /root/Desktop/*.desktop

ENTRYPOINT ["/root/scripts/vnc_startup.sh"]
CMD ["--tail-log"]
