ALive-Docker
************

Dockerfiles used for CI setup for the Liver Analysis Plugin for 3D Slicer:
https://github.com/ALive-research/Slicer-LiverAnalysis

.. |slicer-build-images| image:: https://images.microbadger.com/badges/image/aliveresearch/slicer-build.svg
  :target: https://microbadger.com/images/aliveresearch/slicer-build

.. |slicer-build-ubuntu2004-images| image:: https://images.microbadger.com/badges/image/aliveresearch/slicer-build-ubuntu2004.svg
  :target: https://microbadger.com/images/aliveresearch/slicer-build-ubuntu2004


Docker images containing Slicer build files, so that the CI system don't have to rebuild Slicer for each commit:

aliveresearch/slicer-build
  |slicer-build-images| Based on slicer/slicer-base (centos7)
  
aliveresearch/slicer-build-ubuntu2004
  |slicer-build-ubuntu2004-images| Based on ubuntu:20.04, using code from slicer/slicer-base

Usage
=====

Run this to create an updated docker image. This need to be done to test with new versions of Slicer::

    docker build -t slicer-build-ubuntu2004 .

Give created image a name/tag::

    docker tag slicer-build-ubuntu2004 aliveresearch/slicer-build-ubuntu2004

Upload new image::

    docker push aliveresearch/slicer-build-ubuntu2004
	
Useful commands
===============

Explore Docker image in command line::

    docker run --rm -it --entrypoint=/bin/bash aliveresearch/slicer-build-ubuntu2004
