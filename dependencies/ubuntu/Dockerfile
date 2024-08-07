FROM ubuntu:20.04
LABEL org.opencontainers.image.authors="Sandro Elsweijer <sandro.elsweijer@dlr.de>"
LABEL Description="Ubuntu 20.04 with VTK, NetCDF and OpenCASCADE setup for use with t8code"
SHELL ["/bin/bash", "-c"]

ARG vtk_version=9.1.0
ARG vtk_library_version=9.1

ENV DEBIAN_FRONTEND noninteractive
ENV LIBRARY_PATH=/usr/local/lib:$LIBRARY_PATH
ENV LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH

#######################################################
# Install software requirements and build tools       #
#######################################################

RUN apt-get update -y && apt-get upgrade -y; \
# Prepare apt-get build-dep (https://askubuntu.com/questions/496549/error-you-must-put-some-source-uris-in-your-sources-list)
    cp /etc/apt/sources.list /etc/apt/sources.list~; \
    sed -Ei 's/^# deb-src /deb-src /' /etc/apt/sources.list; \
    apt-get update; \
# install compiler cmake and other useful stuff
    apt-get install -y sudo \
                       build-essential \
                       cmake \
                       wget \
                       curl \
                       git \
                       libmpich-dev \
                       zlib1g-dev \
                       libtool \
                       doxygen-latex \
                       ninja-build \
# install opengl libs
                       xorg-dev \
                       libglu1-mesa-dev \
                       freeglut3-dev \
                       libglew-dev \
                       libglu1-mesa-dev \
                       libgl1-mesa-dev \
# install OpenCASCADE
                       libocct-foundation-dev \
                       libocct-modeling-algorithms-dev\
                       libocct-modeling-data-dev \
# install NetCDF
                       libnetcdf-dev

# clone vtk
RUN git clone https://github.com/Kitware/VTK.git; \
    cd VTK; \
    git checkout v${vtk_version}; \
# configure build with cmake
    mkdir -p build && cd build ; \
    cmake .. \
    -DBUILD_TESTING=OFF \
    -DCMAKE_BUILD_TYPE=Release \
    -DVTK_USE_MPI=ON \
    -DVTK_GROUP_ENABLE_MPI=YES \
    -GNinja; \
# build and install vtk
    ninja -j4; \
    ninja install -j4; \
# cleanup
    cd ../.. && rm -rf VTK
