
# published as raisepartner/jupyter-minimal:1.1.0
ARG py_version="3.9"
ARG ROOT_CONTAINER=python:${py_version}

ARG BASE_CONTAINER=$ROOT_CONTAINER
FROM $BASE_CONTAINER

LABEL maintainer="Vincent Pfister <vincent.pfister@raisepartner.com>"
ARG NB_USER="jovyan"
ARG NB_UID="1000"
ARG NB_GID="100"

# Fix DL4006
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

USER root

# Install all OS dependencies for notebook server that starts but lacks all
# features (e.g., download as all possible file formats)
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get -q update \
  && apt-get install -yq --no-install-recommends \
    wget \
    ca-certificates \
    sudo \
    locales \
    fonts-liberation \
  && apt-get clean && rm -rf /var/lib/apt/lists/*

RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen

# manual installs
WORKDIR /tmp
ENV RUN_ONE_VERSION=1.17 \
    TINI_VERSION=v0.19.0
RUN \
    # install run-one
    wget --quiet https://launchpad.net/run-one/trunk/${RUN_ONE_VERSION}/+download/run-one_${RUN_ONE_VERSION}.orig.tar.gz \
    && tar -xzf run-one_${RUN_ONE_VERSION}.orig.tar.gz \
    && mv run-one-${RUN_ONE_VERSION}/run-one* /usr/local/bin/ \
    && chmod 0755 /usr/local/bin/run-one* \
    && rm -rf /tmp/run-one* \
    # install node
    && curl -fsSL https://deb.nodesource.com/setup_16.x | bash - \
    && apt-get install -y nodejs \
    && apt-get clean && rm -rf /var/lib/apt/lists/* \
    # install tini
    && wget --quiet https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini \
    && mv tini /usr/bin/tini  \
    && chmod +x /usr/bin/tini

# Configure environment
ENV SHELL=/bin/bash \
    NB_USER=$NB_USER \
    NB_UID=$NB_UID \
    NB_GID=$NB_GID \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8 \
    PY_VERSION="$py_version"

# install jupyter
RUN pip install -U pip \
    && pip install --no-cache-dir \
        notebook \
        jupyterlab \
        jupyterhub \
    && npm cache clean --force \
    && jupyter notebook --generate-config \
    && jupyter lab clean 

ENV HOME=/home/$NB_USER \
    PATH="$PATH:/home/$NB_USER/.local/bin" 

# Copy a script that we will use to correct permissions after running certain commands
COPY fix-permissions /usr/local/bin/fix-permissions
RUN chmod a+rx /usr/local/bin/fix-permissions

# Enable prompt color in the skeleton .bashrc before creating the default NB_USER
# hadolint ignore=SC2016
RUN sed -i 's/^#force_color_prompt=yes/force_color_prompt=yes/' /etc/skel/.bashrc

# Create NB_USER with name jovyan user with UID=1000 and in the 'users' group
# and make sure these dirs are writable by the `users` group.
RUN echo "auth requisite pam_deny.so" >> /etc/pam.d/su \
    && sed -i.bak -e 's/^%admin/#%admin/' /etc/sudoers \
    && sed -i.bak -e 's/^%sudo/#%sudo/' /etc/sudoers \
    && useradd -l -m -s /bin/bash -N -u $NB_UID $NB_USER \
    && chmod g+w /etc/passwd 

EXPOSE 8888

# Configure container startup
ENTRYPOINT ["tini", "-g", "--"]
CMD ["start-notebook.sh"]

# Copy local files as late as possible to avoid cache busting
COPY start.sh start-notebook.sh start-singleuser.sh /usr/local/bin/
# Currently need to have both jupyter_notebook_config and jupyter_server_config to support classic and lab
COPY jupyter_notebook_config.py /etc/jupyter/

# Fix permissions on /etc/jupyter as root
USER root

# Prepare upgrade to JupyterLab V3.0 #1205
RUN sed -re "s/c.NotebookApp/c.ServerApp/g" \
    /etc/jupyter/jupyter_notebook_config.py > /etc/jupyter/jupyter_server_config.py

RUN fix-permissions /etc/jupyter/ \
    && fix-permissions $HOME 

# Switch back to jovyan to avoid accidental container runs as root
USER $NB_UID

WORKDIR $HOME
