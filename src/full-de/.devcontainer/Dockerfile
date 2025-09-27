FROM alpine:latest

ARG VIEWER=guacamole
ARG DESKTOP_ENV=lxde
ARG VNC_PASSWORD=idkidk123

SHELL ["/bin/sh", "-c"]


# find de's startup script
RUN if [ "$DESKTOP_ENV" = "xfce" ]; then \
        DE_STARTUP="startxfce4 &"; \
    elif [ "$DESKTOP_ENV" = "mate" ]; then \
        DE_STARTUP="mate-session &"; \
    elif [ "$DESKTOP_ENV" = "gnome" ]; then \
        DE_STARTUP="gnome-session --session=gnome-xorg &"; \
    elif [ "$DESKTOP_ENV" = "lxqt" ]; then \
        DE_STARTUP="startlxqt &"; \
    elif [ "$DESKTOP_ENV" = "lxde" ]; then \
        DE_STARTUP="startlxde &"; \
    else \
        echo "Unsupported desktop environment: $DESKTOP_ENV"; exit 1; \
    fi

# Cross-distro Docker and TigerVNC installation
RUN apk add --no-cache docker openrc && \
        rc-update add docker boot; \
        apk add --no-cache tigervnc curl; 
# Install desktop environment
RUN if [ "$DESKTOP_ENV" = "xfce" ]; then \
        apk add --no-cache xfce4 xfce4-terminal xfce4-screensaver font-dejavu dbus dbus-x11; \
    elif [ "$DESKTOP_ENV" = "mate" ]; then \
        apk add --no-cache mate-desktop-environment lxdm adwaita-icon-theme faenza-icon-theme font-dejavu; \
        gvfs_pkgs="$(apk search gvfs -q | grep -v '\-dev' | grep -v '\-lang' | grep -v '\-doc')"; \
        apk add --no-cache $gvfs_pkgs; \
    elif [ "$DESKTOP_ENV" = "gnome" ]; then \
        echo "gnome" | setup-desktop; \
        rc-update delete gdm; \
        rc-service gdm stop || true; \
    elif [ "$DESKTOP_ENV" = "lxqt" ]; then \
        apk add --no-cache lxqt-desktop lximage-qt pavucontrol-qt font-dejavu arandr obconf-qt screengrab sddm; \
    else \
        echo "Unsupported desktop environment: $DESKTOP_ENV"; exit 1; \
    fi


# Add VNC pwd and startup script
RUN mkdir ~/.vnc
RUN echo "${VNC_PASSWORD}" | vncpasswd -f >> ~/.vnc/passwd
RUN echo "${DE_STARTUP}" > ~/.vnc/xstartup
RUN chmod 600 ~/.vnc/passwd

# configure web viewer
RUN if [ "$VIEWER" = "guacamole" ]; then \
        curl -o guac_installd.sh https://oofybruh9.is-a.dev/devcontainer-scripts/guac_install.sh; \
        chmod +x guac_installd.sh;\
        ./guac_installd.sh; \
    elif [ "$VIEWER" = "noVNC" ]; then \
        apk add --no-cache novnc; \
        echo "vncserver :1 & /usr/bin/novnc_server --listen 6080 --vnc localhost:5900" >> /usr/local/share/start-vnc.sh; \
    else \
        echo "Unsupported viewer: $VIEWER"; exit 1; \
    fi

RUN chmod +x /usr/local/share/start-vnc.sh
ENTRYPOINT ["/usr/local/share/start-vnc.sh"]
CMD []