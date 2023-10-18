FROM jupyter/minimal-notebook:lab-3.6.1

USER root
RUN set -x \
    && apt-get update \
    && apt-get install -y --no-install-recommends xauth xinit dbus-x11

# setup package, enable classic extension, build lab extension
# USER "${NB_USER}"
RUN mamba install -q -c conda-forge -y uwsgi simple-websocket psycopg
# RUN python3 -m pip install pgadmin4
RUN fix-permissions /opt/conda
RUN python3 -m pip install git+https://github.com/huntdatacenter/workbench-app-example.git@main#egg=workbench-example-app

WORKDIR "${HOME}/jupyter-novnc-proxy"
COPY . .
RUN fix-permissions /opt/conda

WORKDIR "${HOME}"
RUN jupyter serverextension enable --sys-prefix jupyter_server_proxy
RUN jupyter labextension disable "@jupyterlab/apputils-extension:announcements"


# copy configs, update permissions as root
USER root
RUN rm -rf "${HOME}/jupyter-novnc-proxy"
RUN cp /etc/jupyter/jupyter_notebook_config.py /etc/jupyter/jupyter_notebook_config_base.py
COPY setup/jupyter_notebook_config.py /etc/jupyter/jupyter_notebook_config.py
RUN fix-permissions /etc/jupyter

WORKDIR /etc/novnc
COPY setup/config_system.py /etc/novnc/config_system.py

# Install VNC 
RUN \
    cd /etc/novnc && \
    # cd ${HOME}/jupyter-novnc-proxy && \
    wget -qO- https://sourceforge.net/projects/tigervnc/files/stable/1.11.0/tigervnc-1.11.0.x86_64.tar.gz/download && \
    mkdir -p ./novnc/utils/websockify && \
    wget -qO- https://github.com/novnc/noVNC/archive/v1.2.0.tar.gz | tar xz --strip 1 -C ./novnc && \
    wget -qO- https://github.com/novnc/websockify/archive/v0.9.0.tar.gz | tar xz --strip 1 -C ./novnc/utils/websockify && \
    chmod +x -v ./novnc/utils/*.sh && \
    mkdir -p ${HOME}/.vnc && \
    fix-permissions /etc/jupyter

# USER "${NB_USER}"
# WORKDIR "${HOME}"

# Basic VNC Settings - no password
ENV \
    VNC_PW=vncpassword \
    VNC_RESOLUTION=1600x900 \
    VNC_COL_DEPTH=24
