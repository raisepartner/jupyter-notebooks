ARG BASE_CONTAINER=raisepartner/jupyter-minimal
FROM $BASE_CONTAINER

LABEL maintainer="Vincent Pfister <vincent.pfister@raisepartner.com>"

USER root

# Install additional OS dependencies
RUN apt-get update \
    && apt-get install -yq --no-install-recommends \
        cm-super \
        libgfortran5 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# add and configure libraries / jupyter extensions
RUN pip install --no-cache-dir -U \
        numpy \
        scipy \
        pandas \
        scikit-learn \
        scipy \
        sympy \
        statsmodels \
        cvxopt \
        matplotlib \
        seaborn \
        ipympl \
        numexpr \
        tables \
        cython \
        fortran-magic \
        protobuf \
        grpcio \
        grpcio-tools \
        jupyterthemes \
        black \
        jupyterlab_code_formatter \
    # [VP 2021-04-17: update jupyterlab because we need >=3.1.0 (still in alpha) for overriding keyboard shortcut.
    #   Can remove when released. see https://discourse.jupyter.org/t/keyboard-shortcuts-with-overrides-json/5751]
    && pip install -U --pre jupyterlab \
    # rebuild jupyter lab
    && jupyter lab build \
    # clean
    && npm cache clean --force \
    && rm -rf /home/$NB_USER/.cache/yarn \
    && rm -rf /home/$NB_USER/.node-gyp \
    && fix-permissions /home/$NB_USER

# Import matplotlib the first time to build the font cache.
ENV XDG_CACHE_HOME="/home/${NB_USER}/.cache/"
RUN MPLBACKEND=agg python -c "import matplotlib.pyplot" && \
    fix-permissions "/home/${NB_USER}"

# settings overrides
COPY overrides.json /usr/local/share/jupyter/lab/settings/

# Switch back to jovyan to avoid accidental container runs as root
USER $NB_UID
WORKDIR $HOME
