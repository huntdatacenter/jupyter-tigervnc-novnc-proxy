FROM jupyter/minimal-notebook:lab-3.6.1

USER root

# -- dev - Installation of basic packages / utilities / apps
RUN set -x \
    && apt-get update -y \
    && apt-get install -y vim wget curl iputils-ping software-properties-common \
    && add-apt-repository ppa:mozillateam/ppa \
    && apt-get install -y firefox

# -- dev - pre-install dependencies - only needed for local testing (faster rebuild)
RUN mamba install -q -c conda-forge -y websockify jupyter-server-proxy
RUN fix-permissions /opt/conda
RUN jupyter serverextension enable --sys-prefix jupyter_server_proxy
RUN jupyter labextension disable "@jupyterlab/apputils-extension:announcements"
RUN fix-permissions /opt/conda

# -- dev - jupyter configuration
USER root
RUN mv -v /etc/jupyter/jupyter_notebook_config.py /etc/jupyter/jupyter_notebook_config_base.py
COPY setup/jupyter_notebook_config.py /etc/jupyter/jupyter_notebook_config.py
RUN fix-permissions /etc/jupyter

# -- VNC DEPENDENCIES - installation of core VNC packages
RUN set -x \
    && apt-get update -y \
    # -- Install XFCE desktop and VNC packages
    && apt-get install -y --no-install-recommends xubuntu-desktop \
    && apt-get install -y --no-install-recommends xauth xinit dbus-x11 \
    && apt-get install -y --no-install-recommends tigervnc-standalone-server tigervnc-xorg-extension \
    # -- Cleanup - screensaver causes issues in VNC
    && apt-get purge -y pm-utils xscreensaver*

# -- VNC DEPENDENCIES - Customizations
COPY --chown=root:root setup/hunt_cloud_1440.png /usr/share/backgrounds
COPY --chown=root:root setup/hunt_cloud_2560.png /usr/share/backgrounds
COPY --chown=root:root setup/xfce4-desktop.xml /etc/xdg/xfce4/xfconf/xfce-perchannel-xml/xfce4-desktop.xml

# -- VNC DEPENDENCIES - install jupyter-tigervnc-novnc-proxy package
#  - use installation from git repository in production setup
# RUN python3 -m pip install git+https://github.com/huntdatacenter/jupyter-tigervnc-novnc-proxy.git@main#egg=jupyter-novnc-prox
# RUN fix-permissions /opt/conda

# -- dev - local setup
WORKDIR "${HOME}/jupyter-tigervnc-novnc-proxy"
COPY . .
RUN python3 -m pip install "${HOME}/jupyter-tigervnc-novnc-proxy"
RUN fix-permissions /opt/conda

# -- dev - jupyter user configuration
USER "${NB_USER}"
WORKDIR "${HOME}"

# Package related ENV variables should be defined inside python package
