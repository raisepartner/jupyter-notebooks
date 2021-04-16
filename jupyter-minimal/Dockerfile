ARG BASE_CONTAINER=raisepartner/jupyter-base
FROM $BASE_CONTAINER

LABEL maintainer="Vincent Pfister <vincent.pfister@raisepartner.com>"

USER root

# Install all OS dependencies for fully functional notebook server
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
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Create alternative for nano -> nano-tiny
RUN update-alternatives --install /usr/bin/nano nano /bin/nano-tiny 10

# Switch back to jovyan to avoid accidental container runs as root
USER $NB_UID
