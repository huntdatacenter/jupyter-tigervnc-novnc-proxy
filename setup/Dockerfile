FROM jupyter/minimal-notebook:lab-4.0.7

USER root

# -- dev - Installation of basic packages / utilities / apps
RUN set -x \
    && apt-get update -y \
    && apt-get install -y vim wget curl iputils-ping software-properties-common \
    && add-apt-repository ppa:mozillateam/ppa \
    && echo "Package: *" | tee /etc/apt/preferences.d/mozilla-firefox \
    && echo "Pin: release o=LP-PPA-mozillateam" | tee -a /etc/apt/preferences.d/mozilla-firefox \
    && echo "Pin-Priority: 1001" | tee -a /etc/apt/preferences.d/mozilla-firefox \
    && echo 'Unattended-Upgrade::Allowed-Origins:: "LP-PPA-mozillateam:${distro_codename}";' | sudo tee /etc/apt/apt.conf.d/51unattended-upgrades-firefox \
    && apt-get install -y firefox

# -- VNC DEPENDENCIES - installation of core VNC packages
RUN set -x \
    && apt-get update -y \
    # -- Install XFCE desktop and VNC packages
    && apt-get install -y --no-install-recommends xubuntu-desktop \
    && apt-get install -y --no-install-recommends xauth xinit dbus-x11 \
    && apt-get install -y --no-install-recommends tigervnc-standalone-server tigervnc-xorg-extension \
    # -- Cleanup - screensaver causes issues in VNC
    && apt-get purge -y pm-utils xscreensaver* \
    && apt-get purge -y update-manager

# -- Customize screen lock
COPY --chown=root:root --chmod=755 setup/bin/xflock4 /usr/bin/xflock4
COPY --chown=root:root --chmod=755 setup/bin/xfce4-session-logout /usr/bin/xfce4-session-logout
# -- VNC Customizations
COPY --chown=root:root setup/hunt_cloud_1440.png /usr/share/backgrounds
COPY --chown=root:root setup/hunt_cloud_2560.png /usr/share/backgrounds
COPY --chown=root:root --chmod=644 setup/etc/xdg/xfce4/xfconf/xfce-perchannel-xml/xfce4-desktop.xml /etc/xdg/xfce4/xfconf/xfce-perchannel-xml/xfce4-desktop.xml
COPY --chown=root:root --chmod=644 setup/etc/xdg/xfce4/xfconf/xfce-perchannel-xml/xsettings.xml /etc/xdg/xfce4/xfconf/xfce-perchannel-xml/xsettings.xml
# COPY --chown=root:root --chmod=644 setup/etc/xdg/xfce4/xfconf/xfce-perchannel-xml/xfce4-session.xml /etc/xdg/xfce4/xfconf/xfce-perchannel-xml/xfce4-session.xml
COPY --chown=root:root --chmod=644 setup/etc/xdg/xdg-xubuntu/xfce4/xfconf/xfce-perchannel-xml/xfce4-session.xml /etc/xdg/xdg-xubuntu/xfce4/xfconf/xfce-perchannel-xml/xfce4-session.xml
COPY --chown=root:root --chmod=644 setup/etc/xdg/xfce4/panel/default.xml /etc/xdg/xfce4/panel/default.xml
COPY --chown=root:root --chmod=644 setup/etc/xdg/menus/xfce-applications.menu /etc/xdg/menus/xfce-applications.menu
RUN set -x \
    && rm -vf /etc/xdg/autostart/xscreensaver.desktop \
    && rm -vf /usr/share/applications/xfce4-session-logout.desktop \
    # && sed -i -E 's/Exec=.*/Exec=novnc-supervisorctl restart vncserver/g' /usr/share/applications/xfce4-session-logout.desktop \
    && sed -i "s|tigervncconfig -iconic|#tigervncconfig -iconic|g" /etc/X11/Xtigervnc-session \
    && echo "done"

# -- dev - pre-install dependencies - only needed for local testing (faster rebuild)
RUN mamba install -q -c conda-forge -y websockify jupyter-server-proxy
RUN fix-permissions /opt/conda
RUN jupyter server extension enable --sys-prefix jupyter_server_proxy
RUN jupyter labextension disable "@jupyterlab/apputils-extension:announcements"
RUN fix-permissions /opt/conda

# -- dev - jupyter configuration
USER root

# RUN mv -v /etc/jupyter/jupyter_notebook_config.py /etc/jupyter/jupyter_notebook_config_base.py
# COPY setup/jupyter_notebook_config.py /etc/jupyter/jupyter_notebook_config.py
RUN mv -v /etc/jupyter/jupyter_server_config.py /etc/jupyter/jupyter_server_config_base.py
COPY setup/jupyter_server_config.py /etc/jupyter/jupyter_server_config.py
RUN fix-permissions /etc/jupyter

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
