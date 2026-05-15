# Ubuntu 24.04 + Qt 6.9 (installed via aqtinstall) + 3D Slicer main, pre-built.
#
# Pre-built Slicer tree is at:
#   /usr/src/Slicer-build/Slicer-build/        <- SlicerConfig.cmake (Slicer_DIR)
#
# Image is intended for downstream Slicer extension CI (Slicer-Liver and
# friends).  Built and published by .github/workflows/build-image.yml.
#
# Ubuntu 24.04 ships Qt 6.4.2, below Slicer's required Qt 6.8.  We
# therefore install Qt 6.9 from Qt's official installer via aqtinstall
# rather than from apt.

FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive \
    LANG=C.UTF-8

# Slicer revision pinned at build time.  Defaults to the tip of main
# but the CI workflow always passes an explicit SHA so images are
# reproducible.
ARG SLICER_REVISION=main

# ---------------------------------------------------------------------------
# 1. System packages: toolchain + X11/GL + Python + Xvfb.
#    The Qt6 dev packages are NOT installed from apt (Ubuntu 24.04 only
#    ships Qt 6.4.2); aqt provides Qt 6.9 below.
# ---------------------------------------------------------------------------
RUN apt-get update && apt-get install -y --no-install-recommends \
    git git-lfs build-essential cmake ninja-build pkg-config \
    curl wget ca-certificates gnupg \
    perl gawk \
    python3 python3-pip python3-venv python3-dev python3-pytest \
    libssl-dev libffi-dev zlib1g-dev libsqlite3-dev libreadline-dev \
    libxt-dev libxext-dev libxrender-dev libxrandr-dev libxfixes-dev \
    libxss-dev libxslt1-dev libxinerama-dev libxcursor-dev libxi-dev \
    libxcomposite-dev libxdamage-dev libxtst-dev \
    libgl1-mesa-dev libglu1-mesa-dev libxkbcommon-dev libxkbcommon-x11-0 \
    libfontconfig1-dev libfreetype6-dev \
    libxcb-cursor0 libxcb-icccm4 libxcb-image0 libxcb-keysyms1 \
    libxcb-render-util0 libxcb-shape0 libxcb-xinerama0 libxcb-xkb1 \
    libdbus-1-3 libnss3 libxcb1-dev \
    libglib2.0-dev \
    # --- Qt 6.9 runtime deps not pulled by aqt itself ---
    # aqt installs Qt binaries from Qt's installer but does NOT install
    # the OS-level libraries those binaries link against at run time.
    # On Ubuntu, Slicer's apt install line uses `qt6-multimedia-dev`
    # which pulls these transitively; we skip qt6-multimedia-dev (it's
    # Qt 6.4 on Ubuntu 24.04, conflicting with aqt's Qt 6.9), so we
    # have to enumerate them here.
    #
    # Qt6Multimedia: libpulse (PulseAudio), libasound2 (ALSA),
    # libgstreamer1.0-0 + libgstreamer-plugins-base1.0-0 (GStreamer
    # for media playback).  Slicer's tests link Qt6Multimedia even
    # when the extension code does not use it, so absence triggers
    # `error while loading shared libraries: libpulse.so.0` at test
    # time and CTest fails 14/14.
    #
    # Qt6WebEngine: libnss3 (already), libnspr4, libxss-conf
    # (already), libdrm2 (transitively).  libnspr4 is the only one
    # not pulled in by what we already have.
    libpulse-dev libasound2-dev \
    libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev \
    libnspr4-dev \
    xvfb \
 && rm -rf /var/lib/apt/lists/*

# ---------------------------------------------------------------------------
# 2. Qt 6.9 via aqtinstall.
#    Note on the module list: Qt 6.9 bundles `qtsvg` and `qttools` into
#    the base install (they're no longer separate aqt modules; verify
#    with `aqt list-qt linux desktop --modules 6.9.0 linux_gcc_64`).
#    The modules we explicitly request cover what Slicer's Ubuntu 24.04
#    build instructions enumerate plus the WebEngine deps.
# ---------------------------------------------------------------------------
ARG QT_VERSION=6.9.0
ENV QT_BASE=/opt/qt
RUN pip3 install --break-system-packages aqtinstall \
 && aqt install-qt linux desktop ${QT_VERSION} linux_gcc_64 \
        --outputdir ${QT_BASE} \
        --modules qtmultimedia qtwebengine qtwebchannel \
                  qtpositioning qt5compat qtscxml \
 && ln -sf ${QT_BASE}/${QT_VERSION}/gcc_64 /opt/qt-current \
 && pip3 uninstall --break-system-packages -y aqtinstall
ENV Qt6_DIR=/opt/qt-current/lib/cmake/Qt6
ENV PATH=/opt/qt-current/bin:${PATH}
ENV LD_LIBRARY_PATH=/opt/qt-current/lib

# ---------------------------------------------------------------------------
# 3. Clone Slicer source at the requested revision.
# ---------------------------------------------------------------------------
RUN mkdir -p /usr/src && cd /usr/src && \
    git clone https://github.com/Slicer/Slicer.git Slicer && \
    cd Slicer && \
    git reset --hard ${SLICER_REVISION} && \
    git log -1 --format='Slicer pinned at %h %s (%ci)'

# ---------------------------------------------------------------------------
# 4. Configure and build Slicer.
#    The build tree ends up at /usr/src/Slicer-build/Slicer-build/ with
#    SlicerConfig.cmake — that's what downstream extension builds point
#    Slicer_DIR at.
# ---------------------------------------------------------------------------
RUN mkdir -p /usr/src/Slicer-build && cd /usr/src/Slicer-build && \
    cmake -G Ninja \
        -DCMAKE_BUILD_TYPE:STRING=Release \
        -DQt6_DIR:PATH=${Qt6_DIR} \
        -DSlicer_REQUIRED_QT_VERSION:STRING=6.8 \
        /usr/src/Slicer && \
    ninja

# ---------------------------------------------------------------------------
# 5. Convenience: Xvfb display for tests that need an X server.
# ---------------------------------------------------------------------------
ENV DISPLAY=:99

WORKDIR /usr/src/Slicer-build

# Build-time metadata.
ARG IMAGE
ARG VCS_REF
ARG VCS_URL
LABEL org.opencontainers.image.title=$IMAGE \
      org.opencontainers.image.source=$VCS_URL \
      org.opencontainers.image.revision=$VCS_REF \
      org.opencontainers.image.licenses="BSD-3-Clause" \
      org.opencontainers.image.description="Ubuntu 24.04 + Qt 6.9 + 3D Slicer (main) pre-built; intended for Slicer extension CI."
