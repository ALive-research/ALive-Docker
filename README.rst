ALive-Docker
************

Dockerfiles used for CI setup for the ALive project's Slicer extensions
(Slicer-Liver and friends).

Images
======

slicer-build-ubuntu2404 (current — published to GHCR)
-----------------------------------------------------

Ubuntu 24.04 + Qt 6.9 (via aqtinstall) + 3D Slicer ``main``,
pre-built.

Published to ``ghcr.io/alive-research/slicer-build-ubuntu2404`` by
``.github/workflows/build-image.yml`` on:

- pushes to ``master`` that touch the Dockerfile,
- manual ``workflow_dispatch`` (with optional Slicer-revision override),
- a weekly cron (Monday 03:00 UTC) that catches Slicer ``main`` drift.

Tags published per build:

- ``:latest`` — the rolling pointer; what consumers normally use.
- ``:main-<7-char-slicer-sha>`` — reproducible pin to a Slicer revision.
- ``:dockerfile-<full-alive-docker-sha>`` — pin to an exact recipe commit.

The pre-built Slicer tree lives at
``/usr/src/Slicer-build/Slicer-build/`` (where ``SlicerConfig.cmake``
resolves; consumers pass this path as ``Slicer_DIR``).

Legacy images (Docker Hub, unmaintained)
----------------------------------------

aliveresearch/slicer-build
  Based on ``slicer/slicer-base`` (CentOS 7).  Last built 2021.

aliveresearch/slicer-build-ubuntu2004
  Based on ``ubuntu:20.04`` + Qt 5 + Slicer 5.2.1.  Last built 2023-02-08.
  Used by ``.circleci/config.yml`` in ``Slicer-Liver`` until the GH
  Actions migration lands; retained here for rollback.

Usage
=====

Pull the published image (from a downstream CI workflow or locally)::

    docker pull ghcr.io/alive-research/slicer-build-ubuntu2404:latest

Local rebuild (e.g. to test a recipe change before pushing)::

    cd slicer-build-ubuntu2404
    docker build -t slicer-build-ubuntu2404:local .

Useful commands
===============

Explore the image interactively::

    docker run --rm -it --entrypoint=/bin/bash \
      ghcr.io/alive-research/slicer-build-ubuntu2404:latest

Check the pinned Slicer revision inside the image::

    docker run --rm ghcr.io/alive-research/slicer-build-ubuntu2404:latest \
      sh -c 'cat /usr/src/Slicer-build/Slicer-build/PACKAGE_FILE.txt 2>/dev/null || \
             ls /opt/qt-current && Qt6_DIR=/opt/qt-current/lib/cmake/Qt6 && echo Slicer_DIR=/usr/src/Slicer-build/Slicer-build'
