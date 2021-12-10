# published as raisepartner/jupyter-minimal:1.1.0
ARG BASE_CONTAINER=raisepartner/jupyter-base:1.1.0
FROM $BASE_CONTAINER

LABEL maintainer="Vincent Pfister <vincent.pfister@raisepartner.com>"

USER root

# Install additional OS dependencies
RUN apt-get update \
    && apt-get install -yq --no-install-recommends \
        build-essential \
        vim-tiny \
        git \
        inkscape \
        libsm6 \
        libxext-dev \
        libxrender1 \
        lmodern \
        netcat \
        # ---- nbconvert dependencies ----
        texlive-xetex \
        texlive-fonts-recommended \
        texlive-plain-generic \
        # ----
        tzdata \
        unzip \
        nano-tiny \
        jed \
        openssh-client \
        pandoc \
        jq \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# add and configure jupyter extensions
RUN pip install --no-cache-dir -U \
        pipenv \
        ipywidgets \
        ipyauth-keycloak \
        jupyterlab_widgets \
        jupyterlab-lsp \
        'python-language-server[all]' \
        jupyter-dash \
        python-language-server \
        jupyterlab-git \
    && jupyter nbextension enable --py widgetsnbextension --sys-prefix \
#     # Activate ipywidgets extension in the environment that runs the notebook server
#     # Also activate ipywidgets extension for JupyterLab
#     # Check this URL for most recent compatibilities
#     # https://github.com/jupyter-widgets/ipywidgets/tree/master/packages/jupyterlab-manager
    && jupyter labextension install @jupyter-widgets/jupyterlab-manager --no-build \
#     # install jupyterlab-git extension
    && jupyter labextension install @jupyterlab/git --no-build \
#     #&& jupyter serverextension enable --py jupyterlab_git \
#     # install jupyterlab gitlab extension
#     #&& jupyter labextension install jupyterlab-gitlab --no-build \
#     #&& jupyter serverextension enable --sys-prefix jupyterlab_gitlab \
    && jupyter serverextension enable --py --sys-prefix ipyauth.ipyauth_callback \
    && jupyter labextension install @raisepartner/ipyauth --no-build \
    && jupyter labextension install jupyterlab_onedarkpro --no-build \
    # rebuild jupyter lab
    && jupyter lab build \
    # clean
    && npm cache clean --force \
    && rm -rf /home/$NB_USER/.cache/yarn \
    && rm -rf /home/$NB_USER/.node-gyp \
    && fix-permissions /home/$NB_USER

COPY extra_config.py /tmp/extra_config.py
RUN cat /tmp/extra_config.py >> /etc/jupyter/jupyter_server_config.py

# Switch back to jovyan to avoid accidental container runs as root
USER $NB_UID
