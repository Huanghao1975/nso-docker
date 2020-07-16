FROM ubuntu

ARG NSO_INSTALL_FILE
ENV http_proxy http://172.16.1.10:3128/
COPY $NSO_INSTALL_FILE /tmp/nso
RUN apt-get update -y

RUN apt-get install -qy \
  wget \
  && wget https://github.com/hawk/lux/releases/download/lux-2.1/lux_2.1-1.deb

RUN apt-get install -qy \
   python3.7 \
   python3-pip
RUN rm -rf /usr/bin/python3 \
  && ln -s /usr/bin/python3.8 /usr/bin/python3 \
  && python3 -m pip install pytz

RUN apt-get install -qy \
  openssh-client \
  libexpat1 \
  && sh /tmp/nso --system-install --non-interactive --run-dir /nso/run --log-dir /log \
  && sed -i -e 's/exec python -u/exec python3 -u/' /opt/ncs/current/bin/ncs-start-python-vm

RUN apt-get update \
  && apt-get install -qy \
  default-jre-headless \
  iputils-ping \
  procps \
  tcpdump \
  telnet \
  xmlstarlet \
  make \
  && apt-get -qy autoremove \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /root/.cache \
  && echo '. /opt/ncs/current/ncsrc' >> /root/.bashrc \
  # Add root to ncsadmin group for easier command-line tools usage
  && groupadd ncsadmin \
  && usermod -a -G ncsadmin root

COPY --from=0 /etc/profile.d /etc/profile.d
COPY --from=0 /etc/init.d/ncs /etc/init.d/.
COPY --from=0 /etc/ncs /etc/ncs/
COPY --from=0 /opt/ncs /opt/ncs/
COPY --from=0 /nso /nsoCOPY
COPY --from=0 /log /log


RUN apt-get clean \
  && rm -rf /var/lib/apt/lists/* /root/.cache

# default shell is ["/bin/sh", "-c"]. We add -l so we get a login shell which
# means the shell reads /etc/profile on startup. /etc/profile includes the files
# in /etc/profile.d where we have ncs.sh that sets the right paths so we can
# access ncsc and other NSO related tools. This makes it possible for
# Dockerfiles, using this image as a base, to directly invoke make for NSO
# package compilation.
SHELL ["/bin/sh", "-lc"]
COPY extra-files/ /
COPY /entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]