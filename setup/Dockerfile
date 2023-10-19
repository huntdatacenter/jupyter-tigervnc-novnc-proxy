FROM jupyter/minimal-notebook:lab-3.6.1

USER root
RUN set -x \
    && apt-get update \
    # TODO -- Install desktop env https://docs.hdc.ntnu.no/do-science/tools/technical/vnc/#_2-install-a-desktop-environment
    && apt-get install -y --no-install-recommends xauth xinit dbus-x11 \
    # TODO -- Install tigervnc packages (apt search tigervnc)
    && apt-get install -y --no-install-recommends <name-of-tigervnc-pkg> \
    # Cleanup -- screensaver causes issues in VNC
    && apt-get purge -y pm-utils xscreensaver* \
    # Installation of utilities
    && apt-get install -y vim file


# setup package, enable classic extension, build lab extension
RUN mamba install -q -c conda-forge -y websockify
RUN fix-permissions /opt/conda

# Install this proxy package
WORKDIR "${HOME}/jupyter-tigervnc-novnc-proxy"
COPY . .
RUN python3 -m pip install "${HOME}/jupyter-tigervnc-novnc-proxy"
# RUN python3 -m pip install git+https://github.com/huntdatacenter/workbench-app-example.git@main#egg=workbench-example-app
RUN fix-permissions /opt/conda

# copy configs, update permissions as root
USER root
RUN rm -rf "${HOME}/jupyter-novnc-proxy"
RUN mv -v /etc/jupyter/jupyter_notebook_config.py /etc/jupyter/jupyter_notebook_config_base.py
COPY setup/jupyter_notebook_config.py /etc/jupyter/jupyter_notebook_config.py
RUN fix-permissions /etc/jupyter

WORKDIR "${HOME}"
RUN jupyter serverextension enable --sys-prefix jupyter_server_proxy
RUN jupyter labextension disable "@jupyterlab/apputils-extension:announcements"

# TODO -- later on we will add some customizations here

USER "${NB_USER}"
WORKDIR "${HOME}"

# Package related ENV variables should be defined inside python package
