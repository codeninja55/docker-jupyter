# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.
# Modified by Andrew Che (andrew@codeninja55.me)

# Installation Flow
# base-notebook: https://github.com/jupyter/docker-stacks/tree/master/base-notebook
# minimal-notebook: https://github.com/jupyter/docker-stacks/blob/master/minimal-notebook/Dockerfile
# scipy-notebook: https://github.com/jupyter/docker-stacks/blob/master/scipy-notebook/Dockerfile
# tensorflow-notebook: https://github.com/jupyter/docker-stacks/tree/master/tensorflow-notebook
# datascience-notebook: https://github.com/jupyter/docker-stacks/tree/master/datascience-notebook
# Install xeus-cling: https://github.com/QuantStack/xeus-cling
# Install jupyter_c_kernel: https://github.com/brendan-rius/jupyter-c-kernel/blob/master/Dockerfile
# Additional configurations: https://jupyter-docker-stacks.readthedocs.io/en/latest/using/common.html

# Ubuntu 18.04 (bionic) from 2018-05-26
# https://github.com/docker-library/official-images/commit/aac6a45b9eb2bffb8102353c350d341a410fb169
# FROM ubuntu:bionic-20180526@sha256:c8c275751219dadad8fa56b3ac41ca6cb22219ff117ca98fe82b42f24e1ba64e
FROM ubuntu:latest@sha256:c8c275751219dadad8fa56b3ac41ca6cb22219ff117ca98fe82b42f24e1ba64e

RUN echo # ------------------------------| DOCKER BUILD - START |------------------------------ #

LABEL maintainer="Andrew Che <andrew@codeninja55.me>"

RUN echo # ==============================| BASE NOTEBOOK - START |============================== #

USER root

# Install all OS dependencies for notebook server that starts but lacks all
# features (e.g., download as all possible file formats)
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update && apt-get -yq dist-upgrade \
 && apt-get install -yq --no-install-recommends \
    wget \
    bzip2 \
    ca-certificates \
    sudo \
    locales \
    fonts-liberation \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

RUN echo # SCIPY NOTEBOOK DEPENDENCIES
# ffmpeg for matplotlib anim
RUN apt-get update && \
   apt-get install -y --no-install-recommends ffmpeg && \
   apt-get clean && \
   rm -rf /var/lib/apt/lists/*

RUN echo # DATASCIENCE NOTEBOOK DEPENDENCIES
# R pre-requisites
RUN apt-get update && \
   apt-get install -y --no-install-recommends \
   fonts-dejavu \
   tzdata \
   gfortran \
   gcc && apt-get clean && \
   rm -rf /var/lib/apt/lists/*

RUN echo "en_AU.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen

# Configure environment
ENV CONDA_DIR=/opt/conda \
    SHELL=/bin/bash \
    NB_USER=codeninja \
    NB_UID=1000 \
    NB_GID=100 \
    LC_ALL=en_AU.UTF-8 \
    LANG=en_AU.UTF-8 \
    LANGUAGE=en_AU.UTF-8
ENV PATH=$CONDA_DIR/bin:$PATH \
    HOME=/home/$NB_USER

COPY fix-permissions /usr/local/bin/fix-permissions

# Create codeninja user with UID=1000 and in the 'users' group
# and make sure these dirs are writable by the `users` group.
RUN groupadd wheel -g 11 && \
    echo "auth required pam_wheel.so use_uid" >> /etc/pam.d/su && \
    useradd -m -s /bin/bash -N -u $NB_UID $NB_USER && \
    mkdir -p $CONDA_DIR && \
    chown $NB_USER:$NB_GID $CONDA_DIR && \
    chmod g+w /etc/passwd && \
    fix-permissions $HOME && \
    fix-permissions $CONDA_DIR

# Change back to $NB_USER from ROOT
USER $NB_UID

# Setup work directory for backward-compatibility
RUN mkdir /home/$NB_USER/work && \
    fix-permissions /home/$NB_USER

RUN mkdir /home/$NB_USER/Dropbox && \
    fix-permissions /home/$NB_USER

# Install conda as codeninja and check the md5 sum provided on the download site
ENV MINICONDA_VERSION 4.5.4
RUN cd /tmp && \
    wget --quiet https://repo.continuum.io/miniconda/Miniconda3-${MINICONDA_VERSION}-Linux-x86_64.sh && \
    echo "a946ea1d0c4a642ddf0c3a26a18bb16d *Miniconda3-${MINICONDA_VERSION}-Linux-x86_64.sh" | md5sum -c - && \
    /bin/bash Miniconda3-${MINICONDA_VERSION}-Linux-x86_64.sh -f -b -p $CONDA_DIR && \
    rm Miniconda3-${MINICONDA_VERSION}-Linux-x86_64.sh && \
    $CONDA_DIR/bin/conda config --system --prepend channels conda-forge && \
    $CONDA_DIR/bin/conda config --system --set auto_update_conda false && \
    $CONDA_DIR/bin/conda config --system --set show_channel_urls true && \
    $CONDA_DIR/bin/conda install --quiet --yes conda="${MINICONDA_VERSION%.*}.*" && \
    $CONDA_DIR/bin/conda update --all --quiet --yes && \
    conda clean -tipsy && \
    rm -rf /home/$NB_USER/.cache/yarn && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

# Install Tini
RUN conda install --yes 'tini=0.18.0' && \
    conda list tini | grep tini | tr -s ' ' | cut -d ' ' -f 1,2 >> $CONDA_DIR/conda-meta/pinned && \
    conda clean -tipsy && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

# Install Jupyter Notebook, Lab, and Hub
# Generate a notebook server config
# Cleanup temporary files
# Correct permissions
# Do all this in a single RUN command to avoid duplicating all of the
# files across image layers when the permissions change
RUN conda install --yes \
    'notebook' \
    'jupyterhub' \
    'jupyterlab' && \
    conda clean -tipsy && \
    jupyter labextension install @jupyterlab/hub-extension@^0.10.0 && \
    npm cache clean --force && \
    jupyter notebook --generate-config && \
    rm -rf $CONDA_DIR/share/jupyter/lab/staging && \
    rm -rf /home/$NB_USER/.cache/yarn && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

USER root

EXPOSE 8888
WORKDIR $HOME

COPY jupyter-notebook.key /home/$NB_USER/.local/share/jupyter/jupyter-notebook.key
COPY jupyter-notebook.crt /home/$NB_USER/.local/share/jupyter/jupyter-notebook.crt

# Configure container startup
ENTRYPOINT ["tini", "-g", "--"]
CMD ["start-notebook.sh"]

# Add local files as late as possible to avoid cache busting
COPY start.sh /usr/local/bin/
COPY start-notebook.sh /usr/local/bin/
COPY start-singleuser.sh /usr/local/bin/
COPY jupyter_notebook_config.py /etc/jupyter/
RUN fix-permissions /etc/jupyter/

USER $NB_UID

RUN echo # ==============================| BASE NOTEBOOK - END |============================== #
# ---------------------------------------------------------------------------------------------------
RUN echo # ==============================| MINIMAL NOTEBOOK - START |============================== #

USER root

# Install all OS dependencies for fully functional notebook server
RUN apt-get update && apt-get install -yq --no-install-recommends \
    build-essential \
    emacs \
    git \
    inkscape \
    jed \
    libsm6 \
    libxext-dev \
    libxrender1 \
    lmodern \
    netcat \
    pandoc \
    python-dev \
    texlive-fonts-extra \
    texlive-fonts-recommended \
    texlive-generic-recommended \
    texlive-latex-base \
    texlive-latex-extra \
    texlive-xetex \
    unzip \
    nano \
    && apt-get clean && \
    rm -rf /var/lib/apt/lists/*

USER $NB_UID

RUN echo # ==============================| MINIMAL NOTEBOOK - END |============================== #
# -------------------------------------------------------------------------------------------------
RUN echo # ==============================| SCIPY NOTEBOOK - START |============================== #

# Install Python 3 packages
# Remove pyqt and qt pulled in for matplotlib since we're only ever going to
# use notebook-friendly backends in these images
RUN conda install --yes \
    'conda-forge::blas=*=openblas' \
    'ipywidgets=7.2*' \
    'pandas=0.23*' \
    'numexpr=2.6*' \
    'matplotlib=2.2*' \
    'scipy=1.1*' \
    'seaborn=0.9*' \
    'scikit-learn=0.19*' \
    'scikit-image=0.14*' \
    'sympy=1.1*' \
    'cython=0.28*' \
    'patsy=0.5*' \
    'statsmodels=0.9*' \
    'cloudpickle=0.5*' \
    'dill=0.2*' \
    'numba=0.38*' \
    'bokeh=0.12*' \
    'sqlalchemy=1.2*' \
    'hdf5=1.10*' \
    'h5py=2.7*' \
    'vincent=0.4.*' \
    'beautifulsoup4=4.6.*' \
    'protobuf=3.*' \
    'xlrd'  && \
    conda remove --quiet --yes --force qt pyqt && \
    conda clean -tipsy && \
    # Activate ipywidgets extension in the environment that runs the notebook server
    jupyter nbextension enable --py widgetsnbextension --sys-prefix && \
    # Also activate ipywidgets extension for JupyterLab
    jupyter labextension install @jupyter-widgets/jupyterlab-manager@^0.36.0 && \
    jupyter labextension install jupyterlab_bokeh@^0.6.0 && \
    npm cache clean --force && \
    rm -rf $CONDA_DIR/share/jupyter/lab/staging && \
    rm -rf /home/$NB_USER/.cache/yarn && \
    rm -rf /home/$NB_USER/.node-gyp && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

# Install facets which does not have a pip or conda package at the moment
RUN cd /tmp && \
    git clone https://github.com/PAIR-code/facets.git && \
    cd facets && \
    jupyter nbextension install facets-dist/ --sys-prefix && \
    cd && \
    rm -rf /tmp/facets && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

# Import matplotlib the first time to build the font cache.
ENV XDG_CACHE_HOME /home/$NB_USER/.cache/
RUN MPLBACKEND=Agg python -c "import matplotlib.pyplot" && \
    fix-permissions /home/$NB_USER

USER $NB_UID

RUN echo # ==============================| SCIPY NOTEBOOK - END |============================== #
# ------------------------------------------------------------------------------------------------------
RUN echo # ==============================| TENSORFLOW NOTEBOOK - START |============================== #

# Install Tensorflow
RUN conda install --yes \
    'tensorflow' \
    'keras' && \
    conda clean -tipsy && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

RUN echo # ==============================| TENSORFLOW NOTEBOOK - END |============================== #
# -------------------------------------------------------------------------------------------------------
RUN echo # ==============================| DATASCIENCE NOTEBOOK - START |============================== #

USER $NB_UID

# R packages including IRKernel which gets installed globally.
RUN conda install --yes \
    'rpy2=2.8*' \
    'r-base=3.4.1' \
    'r-irkernel=0.8*' \
    'r-plyr=1.8*' \
    'r-devtools=1.13*' \
    'r-tidyverse=1.1*' \
    'r-shiny=1.0*' \
    'r-rmarkdown=1.8*' \
    'r-forecast=8.2*' \
    'r-rsqlite=2.0*' \
    'r-reshape2=1.4*' \
    'r-nycflights13=0.2*' \
    'r-caret=6.0*' \
    'r-rcurl=1.95*' \
    'r-crayon=1.3*' \
    'r-randomforest=4.6*' \
    'r-htmltools=0.3*' \
    'r-sparklyr=0.7*' \
    'r-htmlwidgets=1.0*' \
    'r-hexbin=1.27*' && \
    conda clean -tipsy && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

RUN echo # ==============================| DATASCIENCE NOTEBOOK - END |============================== #
# ---------------------------------------------------------------------------------------------
RUN echo # ==============================| XEUS-CLING - START |============================== #

# Install xeus-cling C++ kernel
# https://github.com/QuantStack/xeus-cling
RUN conda install --yes \
    xeus-cling \
    -c QuantStack \
    -c conda-forge && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

RUN echo # ==============================| XEUS-CLING - END |============================== #
# ---------------------------------------------------------------------------------------------------
RUN echo # ==============================| JUPYTER-C-KERNEL - START |============================== #

USER root

# Download jupyter-c-kernel
# https://github.com/brendan-rius/jupyter-c-kernel

WORKDIR /tmp

COPY ./jupyter_c_kernel/ jupyter_c_kernel/
RUN fix-permissions /tmp/jupyter_c_kernel

RUN pip install --no-cache-dir jupyter_c_kernel/
RUN cd jupyter_c_kernel && install_c_kernel --user
RUN fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

USER $NB_UID

RUN echo # ==============================| JUPYTER-C-KERNEL - END |============================== #
# --------------------------------------------------------------------------------------------------
RUN echo # ==============================| JUPYTER THEMES - START |============================== #

WORKDIR $HOME

# Install jupyterthemes
RUN pip install jupyterthemes && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER
RUN conda install --yes \
    'jupyterthemes' && \
    conda clean -tipsy && \
    #jt -t onedork -kl -T -N \
    #-lineh 200 -cellw 80% \
    #-nfs 11 -fs 11 \
    #-ofs 10 -cursc r && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

# Switch back to codeninja to avoid accidental container runs as root
USER $NB_UID

RUN echo # ==============================| JUPYTER THEMES - END |============================== #

RUN echo # ------------------------------| DOCKER BUILD - START |------------------------------ #
