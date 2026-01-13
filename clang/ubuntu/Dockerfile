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
                        ca-certificates \
# Added for vtk library
                        libgl1-mesa-dev libx11-dev libxmu-dev libxi-dev \
                        libxt-dev libxrender-dev libxrandr-dev \
                        libwayland-dev wayland-protocols \
                        libfontconfig1-dev libfreetype6-dev \
                        libxkbcommon-dev libdrm-dev \
# Added for OpenCASCADE
                        libtbb-dev \
                        libpng-dev libjpeg-dev libtiff-dev \
                        libopenexr-dev \
# Added for netcdf
                        autoconf automake libtool m4 \
                        zlib1g-dev libcurl4-openssl-dev

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

# Install HDF5
ENV HDF5_INSTALL_PATH=/opt/install_hdf5_clang
RUN wget https://github.com/HDFGroup/hdf5/releases/download/hdf5_1.14.6/hdf5-1.14.6.tar.gz \
    tar xf hdf5-1.14.6.tar.gz; \
    cd hdf5-1.14.6; \
    ./configure \
    --prefix=${HDF5_INSTALL_PATH} \
    --enable-parallel \
    --enable-shared \
    --enable-hl \
    CC=${OMPI_INSTALL_PATH}/bin/mpicc \
    CXX=${OMPI_INSTALL_PATH}/bin/mpicxx; \
    make -j4 ; \
    make install; \
    cd ..

# Install netcdf-c
ENV NETCDF_INSTALL_PATH=/opt/install_netcdf_clang
RUN wget https://github.com/Unidata/netcdf-c/archive/refs/tags/v4.9.2.tar.gz; \
    tar xf v4.9.2.tar.gz; \
    cd netcdf-c-4.9.2; \
    export CPPFLAGS="-I${HDF5_INSTALL_PATH}/include"; \
    export LDFLAGS="-L${HDF5_INSTALL_PATH}/lib"; \
    ./configure \
    --prefix=${NETCDF_INSTALL_PATH} \
    --enable-parallel-tests \
    --enable-shared \
    CC=${OMPI_INSTALL_PATH}/bin/mpicc; \
    make -j4; \
    sudo make install; \
    cd ..

ENV VTK_INSTALL_PATH=/opt/install_vtk_clang

# Install vtk library with clang compiler
RUN git clone https://github.com/Kitware/VTK.git; \
    cd VTK; \
    git checkout v${vtk_version}; \
# configure build with cmake
    mkdir -p build && cd build ; \
    cmake .. \
        -DCMAKE_C_COMPILER=${OMPI_INSTALL_PATH}/bin/mpicc \
        -DCMAKE_CXX_COMPILER=${OMPI_INSTALL_PATH}/bin/mpicxx \
        -DCMAKE_INSTALL_PREFIX=${VTK_INSTALL_PATH} \
        -DBUILD_TESTING=OFF \
        -DCMAKE_BUILD_TYPE=Release \
        -DVTK_USE_MPI=ON \
        -DVTK_GROUP_ENABLE_MPI=YES; \
        -DNetCDF_ROOT=${NETCDF_INSTALL_PATH} \
        -DHDF5_ROOT=${HDF5_INSTALL_PATH} \
# build and install vtk
    ninja -j4; \
    ninja install -j4; \
# cleanup
    cd ../.. && rm -rf VTK

# Install OpenCASCADE
ENV OCCT_INSTALL_PATH=/opt/install_opencascade_clang
RUN git clone https://github.com/Open-Cascade-SAS/OCCT.git; \
    cd OCCT; \
    git checkout V7_8_0; \
    # configure build with cmake
    mkdir -p build && cd build ; \
    cmake .. \
        -DCMAKE_C_COMPILER=${OMPI_INSTALL_PATH}/bin/mpicc \
        -DCMAKE_CXX_COMPILER=${OMPI_INSTALL_PATH}/bin/mpicxx \
        -DCMAKE_INSTALL_PREFIX=${OCCT_INSTALL_PATH} \
        -DCMAKE_BUILD_TYPE=Release \
        -DBUILD_LIBRARY_TYPE=Shared \
        -DUSE_TBB=ON \
        -DUSE_FREETYPE=ON \
        -DUSE_OPENGL=ON \
        -DBUILD_DRAW=OFF; \
# build and install OpenCASCADE
    ninja -j4; \
    ninja install -j4; \
# cleanup
    cd ../.. && rm -rf OCCT



