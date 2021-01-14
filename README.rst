ALive-Docker
************

Dockerfiles used for CI setup for the Liver Analysis Plugin for 3D Slicer:
https://github.com/ALive-research/Slicer-LiverAnalysis

.. |slicer-build-images| image:: https://images.microbadger.com/badges/image/slicer/slicer-build.svg
  :target: https://microbadger.com/images/slicer/slicer-build
  
slicer/slicer-build
  |slicer-build-images| Docker image containing Slicer build files, so that the CI system don't have to rebuild Slicer for each commit. 

Usage
=====

Run this to create an updated docker image. This need to be done to test with new versions of Slicer
    docker build -t slicer-build .
Upload new image:
    docker push aliveresearch/slicer-build
	
Useful commands
===============

Give created image a new name. Only needed to be run once for each name/tag
    docker tag slicer-build aliveresearch/slicer-build

Explore Docker image in command line
    docker run --rm -it --entrypoint=/bin/bash aliveresearch/slicer-build