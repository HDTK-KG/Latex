FROM ubuntu:24.04

ARG DEBIAN_FRONTEND=noninteractive
ARG UDEV_GOTHIC_VERSION=2.2.0

ENV LANG=ja_JP.UTF-8 \
    LANGUAGE=ja_JP:ja:en_US:en \
    LC_ALL=ja_JP.UTF-8

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# texlive-full includes LuaLaTeX, LuaTeX-ja, latexmk, bib tools, and the
# standard LaTeX packages. fonts-lmodern is listed explicitly so that the
# system-font version is also visible to fontspec.
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        fontconfig \
        fonts-lmodern \
        fonts-noto-cjk \
        git \
        locales \
        perl \
        texlive-full \
        unzip \
    && locale-gen ja_JP.UTF-8 en_US.UTF-8 \
    && rm -rf /var/lib/apt/lists/*

# Source Han Sans/Serif variable OpenType collections. The files are pinned
# to release tags, and work on both amd64 (Windows/Intel Mac) and arm64
# (Apple Silicon) because fonts are architecture-independent.
RUN install -d /usr/local/share/fonts/source-han \
    && curl -fL --retry 3 \
        https://raw.githubusercontent.com/adobe-fonts/source-han-sans/2.005R/Variable/OTC/SourceHanSans-VF.otf.ttc \
        -o /usr/local/share/fonts/source-han/SourceHanSans-VF.otf.ttc \
    && curl -fL --retry 3 \
        https://raw.githubusercontent.com/adobe-fonts/source-han-serif/2.003R/Variable/OTC/SourceHanSerif-VF.otf.ttc \
        -o /usr/local/share/fonts/source-han/SourceHanSerif-VF.otf.ttc

# UDEV Gothic (normal edition). Verify the archive before installing it so a
# corrupted or unexpectedly replaced download cannot enter the image.
RUN curl -fL --retry 3 \
        "https://github.com/yuru7/udev-gothic/releases/download/v${UDEV_GOTHIC_VERSION}/UDEVGothic_v${UDEV_GOTHIC_VERSION}.zip" \
        -o /tmp/udev-gothic.zip \
    && echo "c104c171f6ed8922ca52d74cd915a271e427f1e884e51431aae71d99e8b3b47b  /tmp/udev-gothic.zip" | sha256sum -c - \
    && unzip -q /tmp/udev-gothic.zip -d /tmp/udev-gothic \
    && install -d /usr/local/share/fonts/udev-gothic \
    && find /tmp/udev-gothic -type f -name '*.ttf' \
        -exec install -m 0644 '{}' /usr/local/share/fonts/udev-gothic/ \; \
    && rm -rf /tmp/udev-gothic /tmp/udev-gothic.zip \
    && fc-cache -f -v >/dev/null \
    && luaotfload-tool --update --force >/dev/null

# VS Code Dev Containers convention: do daily work as an unprivileged user.
# Ubuntu 24.04 images may already reserve UID/GID 1000 for an `ubuntu` user.
RUN if id ubuntu >/dev/null 2>&1; then \
        usermod --login vscode --home /home/vscode --move-home ubuntu \
        && groupmod --new-name vscode ubuntu; \
    else \
        groupadd --gid 1000 vscode \
        && useradd --create-home --shell /bin/bash --uid 1000 --gid 1000 vscode; \
    fi

RUN apt-get update \
    && apt-get install -y --no-install-recommends openssh-client \
    && rm -rf /var/lib/apt/lists/*
USER vscode
WORKDIR /workspace

CMD ["sleep", "infinity"]
