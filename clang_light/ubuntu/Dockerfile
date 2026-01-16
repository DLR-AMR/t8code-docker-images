FROM ubuntu:22.04
LABEL org.opencontainers.image.authors="Thomas Spenke <thomas.spenke@dlr.de>"
LABEL Description="Ubuntu 22.04 with clang and openmpi setup to be uses with t8code"
SHELL ["/bin/bash", "-c"]

ARG vtk_version=9.1.0
ARG vtk_library_version=9.1

ENV DEBIAN_FRONTEND=noninteractive
ENV LIBRARY_PATH=/usr/local/lib
ENV LD_LIBRARY_PATH=/usr/local/lib
ENV CUSTOM_CLANG_INSTALL=/custom_clang_install
ENV CUSTOM_CLANG_BUILD=/custom_clang_build

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
                        ca-certificates \
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
# # Added for OpenCASCADE
#                         libtbb-dev \
#                         libpng-dev libjpeg-dev libtiff-dev \
#                         libopenexr-dev \
# # Added for netcdf
                        autoconf automake libtool m4 \
                        zlib1g-dev libcurl4-openssl-dev

# Required for OpenCASCADE
RUN apt-get update && apt-get install -y --no-install-recommends \
    tcl-dev \
    tk-dev \
    libtbb-dev \
    libtbb2


# Prepare folders once
RUN rm -rf $CUSTOM_CLANG_BUILD && mkdir -p $CUSTOM_CLANG_BUILD && \
    rm -rf $CUSTOM_CLANG_INSTALL && mkdir -p $CUSTOM_CLANG_INSTALL

# Export openmpi path to later use it in the github workflow.
ENV OMPI_INSTALL_PATH=$CUSTOM_CLANG_INSTALL/openmpi_clang

# Install openmpi with clang compiler.
# TODO: Make version an ARG as in vtk?
RUN cd $CUSTOM_CLANG_BUILD; \
    wget https://download.open-mpi.org/release/open-mpi/v4.1/openmpi-4.1.6.tar.gz; \
    tar xf openmpi-4.1.6.tar.gz; \
    cd openmpi-4.1.6; \
    CC=clang CXX=clang++ ./configure --prefix=$OMPI_INSTALL_PATH; \
    make -j$(nproc); \
    make install

# Add OpenMPI to paths and environment variables
ENV PATH="$OMPI_INSTALL_PATH/bin:$PATH"
ENV LIBRARY_PATH="$OMPI_INSTALL_PATH/lib:$LIBRARY_PATH"
ENV LD_LIBRARY_PATH="$OMPI_INSTALL_PATH/lib:$LD_LIBRARY_PATH"
ENV CPATH="$OMPI_INSTALL_PATH/include:$CPATH"


# # Install OpenCASCADE
# ENV OCCT_INSTALL_PATH=$CUSTOM_CLANG_INSTALL/occt_clang
# RUN cd $CUSTOM_CLANG_BUILD; \
#     git clone https://github.com/Open-Cascade-SAS/OCCT.git; \
#     cd OCCT; \
#     git checkout V7_5_1; \
#     # configure build with cmake
#     rm -rf build && mkdir -p build && cd build ; \
#     cmake .. \
#         -DCMAKE_C_COMPILER=${OMPI_INSTALL_PATH}/bin/mpicc \
#         -DCMAKE_CXX_COMPILER=${OMPI_INSTALL_PATH}/bin/mpicxx \
#         -DCMAKE_INSTALL_PREFIX=${OCCT_INSTALL_PATH} \
#         -DBUILD_DRAW=OFF \
#         -DUSE_TCL=OFF \
#         -DUSE_TK=OFF \
#         -D3RDPARTY_TCL_INCLUDE_DIR:PATH=IGNORE \
#         -D3RDPARTY_TCL_LIBRARY:FILEPATH=IGNORE \
#         -D3RDPARTY_TCL_LIBRARY_DIR:PATH=IGNORE \
#         -D3RDPARTY_TK_INCLUDE_DIR:PATH=IGNORE \
#         -D3RDPARTY_TK_LIBRARY:FILEPATH=IGNORE \
#         -D3RDPARTY_TK_LIBRARY_DIR:PATH=IGNORE \
#         -D3RDPARTY_TBB_LIBRARY_DIR=/usr/lib/x86_64-linux-gnu \
#         -D3RDPARTY_TBBMALLOC_LIBRARY_DIR=/usr/lib/x86_64-linux-gnu \
#         -DUSE_TBB=OFF \
#         -DTBB_INCLUDE_DIRS=/usr/include \
#         -DTBB_LIBRARIES="/usr/lib/x86_64-linux-gnu/libtbb.so;/usr/lib/x86_64-linux-gnu/libtbbmalloc.so" \
#         -DBUILD_LIBRARY_TYPE=Shared \
#         -DUSE_FREETYPE=ON \
#         -DUSE_OPENGL=ON \
#         -GNinja; \
# # build and install OpenCASCADE
#     ninja -j4; \
#     ninja install

# # Add OpenCASCADE to paths and environment variables
# ENV PATH="$OCCT_INSTALL_PATH/bin:$PATH"
# ENV LIBRARY_PATH="$OCCT_INSTALL_PATH/lib:$LIBRARY_PATH"
# ENV LD_LIBRARY_PATH="$OCCT_INSTALL_PATH/lib:$LD_LIBRARY_PATH"
# ENV CPATH="$OCCT_INSTALL_PATH/include:$CPATH"


# # Install HDF5
# ENV HDF5_INSTALL_PATH=$CUSTOM_CLANG_INSTALL/hdf5_clang
# RUN cd $CUSTOM_CLANG_BUILD; \
#     wget https://support.hdfgroup.org/ftp/HDF5/releases/hdf5-1.10/hdf5-1.10.7/src/hdf5-1.10.7.tar.gz; \
#     tar xf hdf5-1.10.7.tar.gz; \
#     cd hdf5-1.10.7; \
#     rm -rf build && mkdir -p build && cd build; \
#     cmake .. \
#         -DCMAKE_INSTALL_PREFIX=${HDF5_INSTALL_PATH} \
#         -DCMAKE_C_COMPILER=${OMPI_INSTALL_PATH}/bin/mpicc \
#         -DCMAKE_CXX_COMPILER=${OMPI_INSTALL_PATH}/bin/mpicxx \
#         -DBUILD_SHARED_LIBS=ON \
#         -DHDF5_ENABLE_Z_LIB_SUPPORT=ON \
#         -DZLIB_INCLUDE_DIR=/usr/include \
#         -DZLIB_LIBRARY=/usr/lib/x86_64-linux-gnu/libz.so \
#         -GNinja; \
#     ninja -j4 ; \
#     ninja install

# # Add HDF5 to paths and environment variables
# ENV PATH="$HDF5_INSTALL_PATH/bin:$PATH"
# ENV LIBRARY_PATH="$HDF5_INSTALL_PATH/lib:$LIBRARY_PATH"
# ENV LD_LIBRARY_PATH="$HDF5_INSTALL_PATH/lib:$LD_LIBRARY_PATH"
# ENV CPATH="$HDF5_INSTALL_PATH/include:$CPATH"


# # Install netcdf-c
# ENV NETCDF_INSTALL_PATH=$CUSTOM_CLANG_INSTALL/netcdf_clang
# # RUN wget https://github.com/Unidata/netcdf-c/archive/refs/tags/v4.9.2.tar.gz; \
# #     tar xf v4.9.2.tar.gz; \
# #     cd netcdf-c-4.9.2; \
# RUN cd $CUSTOM_CLANG_BUILD; \
#     wget https://github.com/Unidata/netcdf-c/archive/refs/tags/v4.8.1.tar.gz -O netcdf-c-4.8.1.tar.gz; \
#     tar xf netcdf-c-4.8.1.tar.gz; \
#     cd netcdf-c-4.8.1; \
#     rm -rf build && mkdir -p build && cd build; \
#     cmake .. \
#         -DCMAKE_INSTALL_PREFIX=${NETCDF_INSTALL_PATH} \
#         -DCMAKE_C_COMPILER=${OMPI_INSTALL_PATH}/bin/mpicc \
#         -DCMAKE_CXX_COMPILER=${OMPI_INSTALL_PATH}/bin/mpicxx \
#         -DBUILD_SHARED_LIBS=ON \
#         -DENABLE_TESTS=OFF \
#         -GNinja; \
#     ninja -j4; \
#     ninja install

# # Add NetCDF to paths and environment variables
# ENV PATH="$NETCDF_INSTALL_PATH/bin:$PATH"
# ENV LIBRARY_PATH="$NETCDF_INSTALL_PATH/lib:$LIBRARY_PATH"
# ENV LD_LIBRARY_PATH="$NETCDF_INSTALL_PATH/lib:$LD_LIBRARY_PATH"
# ENV CPATH="$NETCDF_INSTALL_PATH/include:$CPATH"

# ENV VTK_INSTALL_PATH=$CUSTOM_CLANG_INSTALL/vtk_clang

# # Install vtk library with clang compiler
# RUN cd $CUSTOM_CLANG_BUILD; \
#     git clone https://github.com/Kitware/VTK.git; \
#     cd VTK; \
#     git checkout v${vtk_version}; \
# # configure build with cmake
#     rm -rf build && mkdir -p build && cd build ; \
#     cmake .. \
#         -DCMAKE_C_COMPILER=${OMPI_INSTALL_PATH}/bin/mpicc \
#         -DCMAKE_CXX_COMPILER=${OMPI_INSTALL_PATH}/bin/mpicxx \
#         -DCMAKE_INSTALL_PREFIX=${VTK_INSTALL_PATH} \
#         -DBUILD_TESTING=OFF \
#         -DCMAKE_BUILD_TYPE=Release \
#         -DVTK_USE_MPI=ON \
#         -DVTK_GROUP_ENABLE_MPI=YES \
#         -GNinja; \
# # build and install vtk
#     ninja -j4; \
#     ninja install -j4

# # Add NetCDF to paths and environment variables
# ENV PATH="$VTK_INSTALL_PATH/bin:$PATH"
# ENV LIBRARY_PATH="$VTK_INSTALL_PATH/lib:$LIBRARY_PATH"
# ENV LD_LIBRARY_PATH="$VTK_INSTALL_PATH/lib:$LD_LIBRARY_PATH"
# ENV CPATH="$VTK_INSTALL_PATH/include:$CPATH"

# Clean up build directories
RUN rm -rf $CUSTOM_CLANG_BUILD

# # BACKUP lines
#         # -DHDF5_ROOT=${HDF5_INSTALL_PATH} \
#         # -DNetCDF_ROOT=${NETCDF_INSTALL_PATH} \
#         # -DHDF5_ROOT=${HDF5_INSTALL_PATH} \