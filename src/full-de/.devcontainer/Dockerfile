FROM ${OS}:latest

ARG OS
ARG VIEWER
ARG DESKTOP_ENV
ENV DEBIAN_FRONTEND=noninteractive
ARG VNC_PASSWORD

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
RUN if [ "$OS" = "debian" ] || [ "$OS" = "ubuntu" ]; then \
        apt-get update && \
        apt-get install -y curl ca-certificates gnupg lsb-release && \
        curl -fsSL https://get.docker.com | sh; \
        apt-get install -y tigervnc-standalone-server tigervnc-common; \
    elif [ "$OS" = "alpine" ]; then \
        apk add --no-cache docker openrc && \
        rc-update add docker boot; \
        apk add --no-cache tigervnc; \
    elif [ "$OS" = "fedora" ]; then \
        dnf -y install dnf-plugins-core && \
        dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo && \
        dnf -y install docker-ce docker-ce-cli containerd.io; \
        dnf install tigervnc-server \
    elif [ "$OS" = "arch" ]; then \
        pacman -Sy --noconfirm --needed docker; \
        pacman -Sy --noconfirm --needed tigervnc; \
    else \
        echo "Unsupported OS: $OS"; exit 1; \
    fi

# Install desktop environment
RUN if [ "$DESKTOP_ENV" = "xfce" ]; then \
        if [ "$OS" = "debian" ] || [ "$OS" = "ubuntu" ]; then \
            apt-get install -y xfce4 xfce4-goodies; \
        elif [ "$OS" = "alpine" ]; then \
            apk add --no-cache xfce4 xfce4-terminal dbus dbus-x11; \
        elif [ "$OS" = "fedora" ]; then \
            dnf groupinstall -y "Xfce"; \
        elif [ "$OS" = "arch" ]; then \
            pacman -Sy --noconfirm --needed xfce4 xfce4-goodies; \
        fi; \
    elif [ "$DESKTOP_ENV" = "mate" ]; then \
        if [ "$OS" = "debian" ] || [ "$OS" = "ubuntu" ]; then \
            apt-get install -y mate-desktop-environment mate-desktop-en vironment-extras; \
        elif [ "$OS" = "alpine" ]; then \
            apk add mate-desktop-environment adwaita-icon-theme faenza-icon-theme font-dejavu \
            gvfs_pkgs=$(apk search gvfs -q | grep -v '\-dev' | grep -v '\-lang' | grep -v '\-doc') \
            apk add $gvfs_pkgs \
        elif [ "$OS" = "fedora" ]; then \
            dnf groupinstall -y "MATE Desktop"; \
        elif [ "$OS" = "arch" ]; then \
            pacman -Sy --noconfirm --needed mate mate-extra; \
        fi; \
    elif [ "$DESKTOP_ENV" = "gnome" ]; then \
        if [ "$OS" = "debian" ] || [ "$OS" = "ubuntu" ]; then \
            apt-get install -y gnome-session gnome-terminal gnome-tweaks gnome-control-center nautilus gnome-themes-standard fonts-dejavu; \
        elif [ "$OS" = "alpine" ]; then \
            apk add --no-cache gnome gnome-terminal gnome-tweaks nautilus adwaita-icon-theme font-dejavu dbus dbus-x11; \
        elif [ "$OS" = "fedora" ]; then \
            dnf groupinstall -y "GNOME Desktop Environment"; \
        elif [ "$OS" = "arch" ]; then \
            pacman -Sy --noconfirm --needed gnome gnome-extra; \
        fi; \
    elif [ "$DESKTOP_ENV" = "lxqt" ]; then \
        if [ "$OS" = "debian" ] || [ "$OS" = "ubuntu" ]; then \
            apt-get install -y lxqt; \
        elif [ "$OS" = "alpine" ]; then \
            apk add --no-cache lxqt dbus dbus-x11; \
        elif [ "$OS" = "fedora" ]; then \
            dnf groupinstall -y "LXQt Desktop"; \
        elif [ "$OS" = "arch" ]; then \
            pacman -Sy --noconfirm --needed lxqt; \
        fi; \
    elif [ "$DESKTOP_ENV" = "lxde" ]; then \
        if [ "$OS" = "debian" ] || [ "$OS" = "ubuntu" ]; then \
            apt-get install -y lxde; \
        elif [ "$OS" = "alpine" ]; then \
            apk add --no-cache lxde dbus dbus-x11; \
        elif [ "$OS" = "fedora" ]; then \
            dnf groupinstall -y "LXDE Desktop"; \
        elif [ "$OS" = "arch" ]; then \
            pacman -Sy --noconfirm --needed lxde; \
        fi; \
    else \
        echo "Unsupported desktop environment: $DESKTOP_ENV"; exit 1; \
    fi

# Add VNC pwd and startup script
RUN mkdir ~/.vnc
RUN echo "${VNC_PASSWORD}" | vncpasswd -f >> ~/.vnc/passwd
RUN echo "${DE_STARTUP}" > ~/.vnc/xstartup
RUN chmod 600 ~/.vnc/passwd

# Add DinD script
ADD https://raw.githubusercontent.com/moby/moby/master/hack/dind /usr/local/share/dind
RUN chmod +x /usr/local/share/dind

# configure web viewer
RUN if [ "$VIEWER" = "guacamole" ]; then \
        curl -o /usr/local/share/guacamole-docker.yml https://raw.githubusercontent.com/oofybruh9/devcontainers-templates/main/src/full-de/guac-compose.yml \
        echo "docker compose -f /usr/local/share/guacamole-docker.yml up -d & /usr/local/share/dind" >> /usr/local/share/start-vnc.sh \
    elif [ "$VIEWER" = "noVNC" ]; then \
        git clone https://github.com/noVNC/noVNC /opt/noVNC \
        chmod +x /opt/noVNC/utils/novnc_proxy\
        echo "/opt/noVNC/utils/novnc_proxy --listen 6080 --vnc localhost:5900 & /usr/local/share/dind" >> /usr/local/share/start-vnc.sh \
    else \
        echo "Unsupported viewer: $VIEWER"; exit 1; \
    fi
RUN chmod +x /usr/local/share/start-vnc.sh
ENTRYPOINT ["/usr/local/share/start-vnc.sh"]
CMD []