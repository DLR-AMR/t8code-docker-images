FROM ubuntu:22.04
LABEL org.opencontainers.image.authors="Thomas Spenke <thomas.spenke@dlr.de>"
LABEL Description="Ubuntu 22.04 with clang and openmpi setup to be uses with t8code"
SHELL ["/bin/bash", "-c"]

ARG vtk_version=9.1.0
ARG vtk_library_version=9.1

ENV DEBIAN_FRONTEND=noninteractive
ENV LIBRARY_PATH=/usr/local/lib
ENV LD_LIBRARY_PATH=/usr/local/lib

#######################################################
# Install software requirements and build tools       #
#######################################################

# Install required packages (without recommends to avoid gcc).
RUN apt-get update -y && apt-get upgrade -y; \
    apt-get install -y  wget \
                        tar \
                        lld \
                        make \
                        cmake \
                        ninja-build \
                        libc6-dev \
                        libc++-dev \
                        libc++abi-dev \
                        pkg-config \
                        git \
                        clang \
                        ca-certificates
# Export openmpi path to later use it in the github workflow.
ENV OMPI_INSTALL_PATH=/opt/install_openmpi_clang
ENV PATH="$OMPI_INSTALL_PATH/bin:$PATH"

# Install openmpi with clang compiler.
# TODO: Make version an ARG as in vtk?
RUN mkdir install_ompi_clang; \
    wget https://download.open-mpi.org/release/open-mpi/v4.1/openmpi-4.1.6.tar.gz; \
    tar xf openmpi-4.1.6.tar.gz; \
    cd openmpi-4.1.6; \
    CC=clang CXX=clang++ ./configure --prefix=$OMPI_INSTALL_PATH; \
    make -j$(nproc); \
    make install

# Install vtk library with clang compiler
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
    make -j4; \
    make install -j4; \
# cleanup
    cd ../.. && rm -rf VTK



