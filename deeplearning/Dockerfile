FROM sinzlab/pytorch:bethgeclusterbase
MAINTAINER bethgelab (it@bethgelab.org)

USER root

# Set the time zone correctly
ENV TZ=Europe/Berlin
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

ENV SHELL /bin/bash

# Install SSH, sudo and gfortran, etc.
RUN apt-get update -qq \
 && DEBIAN_FRONTEND=noninteractive apt-get install -yq -qq --no-install-recommends \
    ca-certificates \
    gfortran \
    openssh-server \
    pwgen \
    screen \
    sudo \
    tmux \
    xterm \
    zsh \
    libcurl3-dev \
    libfreetype6-dev \
    libzmq3-dev \
    rsync \
    libjs-mathjax \ 
	libgoogle-perftools-dev \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Enable passwordless sudo for all users
RUN echo '%sudo ALL=(ALL:ALL) NOPASSWD:ALL' >> /etc/sudoers

# Setup gosu (https://github.com/tianon/gosu)
# gosu is an improved version of su which behaves better inside docker
# we use it to dynamically switch to the desired user in the entrypoint
# (see below)
ENV GOSU_VERSION 1.10
# Use unsecure HTTP via Port 80 to fetch key due to firewall in CIN.
RUN set -x \
 && dpkgArch="$(dpkg --print-architecture | awk -F- '{ print $NF }')" \
 && wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch" \
 && chmod +x /usr/local/bin/gosu \
 && gosu nobody true

COPY entrypoint.sh /usr/local/bin/
RUN chmod a+x /usr/local/bin/entrypoint.sh

ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]

RUN mkdir /usr/.jupyter
ENV JUPYTER_CONFIG_DIR /usr/.jupyter
COPY jupyter_notebook_config.py /usr/.jupyter/

# Python packages
RUN pip3 --no-cache-dir install \
    ipykernel \
    click \
    mypy \
    boltons \
 && python3 -m ipykernel.kernelspec

# enable jupyter extensions
RUN pip install jupyter_contrib_nbextensions \
 && jupyter contrib nbextension install --system \
 && jupyter nbextension enable codefolding/main \
 && jupyter nbextension enable hinterland/hinterland \
 && jupyter nbextension enable varInspector/main \
 && jupyter nbextension enable comment-uncomment/main

# Jupyter has issues with being run directly:
#   https://github.com/ipython/ipython/issues/7062
# We just add a little wrapper script.
COPY run_jupyter.sh /usr/local/bin
COPY run_jupyterlab.sh /usr/local/bin
RUN chmod +x /usr/local/bin/run_jupyter.sh \
 && chmod -R a+rwx /usr/.jupyter \
 && chmod +x /usr/local/bin/run_jupyterlab.sh

USER $NB_USER

CMD ["/usr/local/bin/run_jupyter.sh"]
