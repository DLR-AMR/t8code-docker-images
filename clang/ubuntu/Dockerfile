FROM ubuntu:22.04
LABEL org.opencontainers.image.authors="Thomas Spenke <thomas.spenke@dlr.de>"
LABEL Description="Ubuntu 22.04 with clang and openmpi setup to be uses with t8code"
SHELL ["/bin/bash", "-c"]

# Set vtk version
ARG vtk_version=9.1.0
ARG vtk_library_version=9.1

# Set openmpi version
ARG ompi_version=4.1.6
ARG ompi_library_version=4.1

# Set environment variables
ENV DEBIAN_FRONTEND=noninteractive
ENV LIBRARY_PATH=/usr/local/lib
ENV LD_LIBRARY_PATH=/usr/local/lib
ENV CUSTOM_CLANG_INSTALL=/custom_clang_install
ENV CUSTOM_CLANG_BUILD=/custom_clang_build

# Set install paths for openmpi and vtk
ENV OMPI_INSTALL_PATH=$CUSTOM_CLANG_INSTALL/openmpi_clang
ENV VTK_INSTALL_PATH=$CUSTOM_CLANG_INSTALL/vtk_clang

#######################################################
# Install software requirements and build tools       #
#######################################################

# Install required packages (without recommends to avoid gcc).
RUN apt-get update -y && apt-get upgrade -y; \
    apt-get install -y  wget \
                        sudo \
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
                        gfortran \
                        ca-certificates \
                        autoconf automake libtool m4 \
                        zlib1g-dev libcurl4-openssl-dev \
# Added for vtk library
                        libgl1-mesa-dev libx11-dev libxmu-dev libxi-dev \
                        libxt-dev libxrender-dev libxrandr-dev \
                        libwayland-dev wayland-protocols \
                        libfontconfig1-dev libfreetype6-dev \
                        libxkbcommon-dev libdrm-dev \
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

# Prepare folders once
RUN rm -rf $CUSTOM_CLANG_BUILD && mkdir -p $CUSTOM_CLANG_BUILD && \
    rm -rf $CUSTOM_CLANG_INSTALL && mkdir -p $CUSTOM_CLANG_INSTALL

# Install openmpi with clang compiler.
RUN cd $CUSTOM_CLANG_BUILD; \
    wget https://download.open-mpi.org/release/open-mpi/v${ompi_library_version}/openmpi-${ompi_version}.tar.gz; \
    tar xf openmpi-${ompi_version}.tar.gz; \
    cd openmpi-${ompi_version}; \
    CC=clang CXX=clang++ FC=gfortran ./configure --prefix=$OMPI_INSTALL_PATH; \
    make -j$(nproc); \
    make install

# Add OpenMPI to paths and environment variables
ENV PATH="$OMPI_INSTALL_PATH/bin:$PATH"
ENV LIBRARY_PATH="$OMPI_INSTALL_PATH/lib:$LIBRARY_PATH"
ENV LD_LIBRARY_PATH="$OMPI_INSTALL_PATH/lib:$LD_LIBRARY_PATH"
ENV CPATH="$OMPI_INSTALL_PATH/include:$CPATH"

# Install vtk library with clang compiler
RUN cd $CUSTOM_CLANG_BUILD; \
    git clone https://github.com/Kitware/VTK.git; \
    cd VTK; \
    git checkout v${vtk_version}; \
# configure build with cmake
    rm -rf build && mkdir -p build && cd build ; \
    cmake .. \
        -DCMAKE_C_COMPILER=${OMPI_INSTALL_PATH}/bin/mpicc \
        -DCMAKE_CXX_COMPILER=${OMPI_INSTALL_PATH}/bin/mpicxx \
        -DCMAKE_INSTALL_PREFIX=${VTK_INSTALL_PATH} \
        -DBUILD_TESTING=OFF \
        -DCMAKE_BUILD_TYPE=Release \
        -DVTK_USE_MPI=ON \
        -DVTK_GROUP_ENABLE_MPI=YES \
        -GNinja; \
# build and install vtk
    ninja -j4; \
    ninja install -j4

# Add NetCDF to paths and environment variables
ENV PATH="$VTK_INSTALL_PATH/bin:$PATH"
ENV LIBRARY_PATH="$VTK_INSTALL_PATH/lib:$LIBRARY_PATH"
ENV LD_LIBRARY_PATH="$VTK_INSTALL_PATH/lib:$LD_LIBRARY_PATH"
ENV CPATH="$VTK_INSTALL_PATH/include:$CPATH"

# Clean up build directories
RUN rm -rf $CUSTOM_CLANG_BUILD