# This can be used to create a Docker container with a prebuilt 3D Slicer

# Run this to create a docker image:
# docker build -t slicer-build .

# Give created image a new name. Only needed to be run once for each name/tag
# docker tag slicer-build aliveresearch/slicer-build

# Upload new image
# docker push aliveresearch/slicer-build

FROM slicer/slicer-base:latest
RUN \
	#ls	  
	# Checkout latest
		cd /usr/src && \
		# Currently main Slicer fork won't compile in Docker
		#git clone git://github.com/Slicer/Slicer && \
		git clone https://github.com/ALive-research/Slicer.git && \
		cd Slicer && \
		
	# Build
		cd /usr/src/Slicer-build && \/usr/src/Slicer-build/BuildSlicer.sh 
		
WORKDIR /usr/src/Slicer-build
CMD bash