FROM mcr.microsoft.com/vscode/devcontainers/base:0-hirsute

ARG MINIFORGE_NAME=Miniforge3
ARG MINIFORGE_VERSION=4.11.0-0
ARG USERNAME=vscode

# Setup default python tools in a venv via pipx to avoid conflicts
ENV CONDA_DIR=/opt/conda \
    LANG=C.UTF-8 \
    LC_ALL=C.UTF-8 \
    PIPX_HOME=/usr/local/py-utils \
    PIPX_BIN_DIR=/usr/local/py-utils/bin

ENV PATH=${CONDA_DIR}/bin:${PATH}:${PIPX_BIN_DIR}

# Copy library scripts to execute
COPY .devcontainer/library-scripts/*.sh .devcontainer/library-scripts/*.env /tmp/library-scripts/

# 1. Install just enough for conda to work
# 2. Keep $HOME clean (no .wget-hsts file), since HSTS isn't useful in this context
# 3. Install miniforge from GitHub releases
# 4. Apply some cleanup tips from https://jcrist.github.io/conda-docker-tips.html
#    Particularly, we remove pyc and a files. The default install has no js, we can skip that
# 5. Activate base by default when running as any *non-root* user as well
#    Good security practice requires running most workloads as non-root
#    This makes sure any non-root users created also have base activated
#    for their interactive shells.
# 6. Activate base by default when running as root as well
#    The root user is already created, so won't pick up changes to /etc/skel
RUN apt-get update \
    && export DEBIAN_FRONTEND=noninteractive \
    && apt-get install --no-install-recommends --yes \
        wget bzip2 ca-certificates libglib2.0-0 libsm6 libxext6 libxrender1 \
        mercurial subversion build-essential git \
    && apt-get upgrade -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/* \
    && mv -f "/tmp/library-scripts/meta.env" /usr/local/etc/vscode-dev-containers/meta.env \
    && wget --no-hsts --quiet https://github.com/conda-forge/miniforge/releases/download/${MINIFORGE_VERSION}/${MINIFORGE_NAME}-${MINIFORGE_VERSION}-Linux-$(uname -m).sh -O /tmp/miniforge.sh \
    && /bin/bash /tmp/miniforge.sh -b -p ${CONDA_DIR} \
    && rm /tmp/miniforge.sh \
    && conda clean -tipsy \
    && find ${CONDA_DIR} -follow -type f -name '*.a' -delete \
    && find ${CONDA_DIR} -follow -type f -name '*.pyc' -delete \
    && conda clean -afy \
    && ln -s ${CONDA_DIR}/etc/profile.d/conda.sh /etc/profile.d/conda.sh \
    && echo ". ${CONDA_DIR}/etc/profile.d/conda.sh" >> ~/.bashrc \
    && echo "conda activate base" >> ~/.bashrc \
    && groupadd -r conda --gid 900 \
    && usermod -aG conda ${USERNAME} \
    && chown -R :conda ${CONDA_DIR} \
    && chmod -R g+w ${CONDA_DIR} \
    && find /opt -type d | xargs -n 1 chmod g+s

RUN bash /tmp/library-scripts/python-debian.sh "none" "${CONDA_DIR}" "${PIPX_HOME}" "${USERNAME}" "true" \
    && apt-get clean -y && rm -rf /var/lib/apt/lists/* .devcontainer/library-scripts

# Copy environment.yml (if found) to a temp location so we update the environment.
COPY .devcontainer/environment.yml /tmp/conda-tmp/
RUN umask 0002 \
    && ${CONDA_DIR}/bin/mamba env update -n base -f /tmp/conda-tmp/environment.yml \
    && rm -rf /tmp/conda-tmp \
    && ${CONDA_DIR}/bin/conda clean -afy \
    && find ${CONDA_DIR} -follow -type f -name '*.a' -delete \
    && find ${CONDA_DIR} -follow -type f -name '*.pyc' -delete \
    && jupyter labextension install dask-labextension \
    && find ${CONDA_DIR} -follow -type f -name '*.js.map' -delete \
    && find ${CONDA_DIR}/lib/python*/site-packages/bokeh/server/static -follow -type f -name '*.js' ! -name '*.min.js' -delete \
    && STAN_BACKEND=CMDSTANPY CMDSTAN=${CONDA_DIR}/bin/cmdstan ${CONDA_DIR}/bin/python -m pip install --no-cache-dir --no-deps -v prophet \
    && ${CONDA_DIR}/bin/python -m pip cache purge
