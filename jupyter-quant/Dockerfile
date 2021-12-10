# published as raisepartner/jupyter-quant:1.1.0
ARG BASE_CONTAINER=raisepartner/jupyter-minimal:1.1.0
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
RUN pip install -U pip \
    && pip install --no-cache-dir -U \
        numpy \
        scipy \
        pandas \
        pandas-datareader \
        scikit-learn \
        scipy \
        sympy \
        statsmodels \
        plotly \
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
        jupyterlab \
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
