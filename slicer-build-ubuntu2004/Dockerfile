# Create:
# docker build -t slicer-build-ubuntu2004 .

FROM ubuntu:20.04

RUN apt-get upgrade && apt-get update 

# Ubuntu 20.04, from https://slicer.readthedocs.io/en/latest/developer_guide/build_instructions/linux.html
RUN DEBIAN_FRONTEND=noninteractive apt install -y git subversion build-essential cmake cmake-curses-gui cmake-qt-gui qt5-default qtmultimedia5-dev qttools5-dev libqt5xmlpatterns5-dev libqt5svg5-dev qtwebengine5-dev qtscript5-dev qtbase5-private-dev libqt5x11extras5-dev libxt-dev 

RUN apt install -y ninja-build gawk

# Install Xvfb - to be run from CircleCI
RUN apt-get install xvfb
ENV DISPLAY :99

# Copied, with some modifications, from slicer/slicer-base:latest

RUN \
  #
  # Download sources
  #
  cd /usr/src && \
  git clone https://github.com/Slicer/Slicer.git && \
  cd Slicer
 
RUN \
  cd /usr/src/Slicer && \
  git reset --hard ${SLICER_VERSION} && \
  #
  # Configure
  #
  mkdir /usr/src/Slicer-build && \
  cd /usr/src/Slicer-build && \
  cmake \
    -G Ninja \
    -DCMAKE_BUILD_TYPE:STRING=Release \
    -DQt5_DIR:PATH=${Qt5_DIR} \
    -DSlicer_BUILD_ITKPython:BOOL=OFF \
    -DSlicer_INSTALL_ITKPython:BOOL=OFF \
      /usr/src/Slicer && \
  #
  # Split the superbuild commands (what is executed by ninja) into building Slicer's
  # dependencies, and building Slicer itself.
  #   NOTE: xx00, xx01... files are created by `csplit` from the `ninja ...` output.
  #
  cd /usr/src/Slicer-build && \
  ninja -t commands Slicer | csplit - '/Slicer-mkdir/' && \
  echo '#!/bin/bash' > BuildSlicerDependencies.sh && \
    echo "set -e" >> BuildSlicerDependencies.sh && \
    echo "set -x" >> BuildSlicerDependencies.sh && \
    echo "set -o pipefail" >> BuildSlicerDependencies.sh && \
    echo "set -o" >> BuildSlicerDependencies.sh && \
    while IFS='' read -r line || [[ -n "$line" ]]; do \
      echo "$line || exit \$?" >> BuildSlicerDependencies.sh; \
    done < xx00 && \
    chmod +x BuildSlicerDependencies.sh && \
    rm xx00 && \
  echo '#!/bin/bash' > BuildSlicer.sh && \
    echo "set -e" >> BuildSlicer.sh && \
    echo "set -x" >> BuildSlicer.sh && \
    echo "set -o pipefail" >> BuildSlicer.sh && \
    echo "set -o" >> BuildSlicer.sh && \
    head -n 5 xx01 > xx01-no-slicer-build && \
    while IFS='' read -r line || [[ -n "$line" ]]; do \
      echo "$line || exit \$?" >> BuildSlicer.sh; \
    done < xx01-no-slicer-build && \
    echo "cmake --build /usr/src/Slicer-build/Slicer-build -- \$BUILD_TOOL_FLAGS" >> BuildSlicer.sh && \
    echo "cmake --build /usr/src/Slicer-build/Slicer-build --target package -- \$BUILD_TOOL_FLAGS | tee /usr/src/Slicer-build/Slicer-build/PACKAGES.txt" >> BuildSlicer.sh && \
    echo "cat /usr/src/Slicer-build/Slicer-build/PACKAGES.txt | gawk 'match(\$0, /CPack: - package: (.*) generated/, a) {print a[1]}' > /usr/src/Slicer-build/Slicer-build/PACKAGE_FILE.txt" >> BuildSlicer.sh && \
    chmod +x BuildSlicer.sh && \
    rm xx01 xx01-no-slicer-build && \
  #
  # Build dependencies
  #
  ./BuildSlicerDependencies.sh 


# Build Slicer
RUN cd /usr/src/Slicer-build && \/usr/src/Slicer-build/BuildSlicer.sh 

WORKDIR /usr/src/Slicer-build

# Build-time metadata as defined at http://label-schema.org
# BUILD_DATE is omitted to avoid unneeded rebuild
ARG IMAGE
ARG VCS_REF
ARG VCS_URL
LABEL org.label-schema.name=$IMAGE \
      org.label-schema.vcs-ref=$VCS_REF \
      org.label-schema.vcs-url=$VCS_URL \
      org.label-schema.schema-version="1.0" \
	  
	 
