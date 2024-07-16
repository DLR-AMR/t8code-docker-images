FROM ubuntu:24.04
LABEL org.opencontainers.image.authors="Sandro Elsweijer <sandro.elsweijer@dlr.de>"
LABEL Description="Ubuntu 24.04 with VTK, NetCDF and OpenCASCADE setup for use with t8code"
SHELL ["/bin/bash", "-c"]

ARG t8code_version=1.0.0
ARG vtk_version=9.1.0
ARG vtk_library_version=9.1
ARG occt_version=7_8_1

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
    apt-get install -y build-essential \
                       cmake \
                       wget \
                       curl \
                       git \
                       libmpich-dev \
                       zlib1g-dev \
                       libtool \
                       doxygen-latex \
# install opengl libs
                       xorg-dev \
                       libglu1-mesa-dev \
                       freeglut3-dev \
                       libglew-dev \
                       libglu1-mesa \
                       libglu1-mesa-dev \
                       libgl1-mesa-glx \
                       libgl1-mesa-dev \
# install NetCDF
                       libnetcdf-dev

#######################################
# build vtk with VTK_WRAP_PYTHON=ON   #
#######################################

# clone occt
RUN git clone https://github.com/Open-Cascade-SAS/OCCT.git; \
    cd OCCT; \
    git checkout V${occt_version}; \
# configure build with cmake
    mkdir -p build && cd build; \
    cmake .. \
    -DCMAKE_BUILD_TYPE=Release; \
# build and install vtk
    make install -j; \
# cleanup
    cd ../.. && rm -rf OCCT


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
    -DVTK_GROUP_ENABLE_MPI=YES; \
# build and install vtk
    make install -j; \
# cleanup
    cd ../.. && rm -rf VTK

RUN if [ "${t8code_version}" != "0" ]; then \
cd ~; \
git clone https://github.com/DLR-AMR/t8code.git; \
cd t8code; \
git checkout v${t8code_version}; \
git submodule init; \
git submodule update; \
./bootstrap; \
cd ..; \
mkdir t8code_build; \
cd t8code_build; \
./../t8code/configure CFLAGS="-O3" CXXFLAGS="-O3" \
                        --enable-mpi CC=mpicc CXX=mpicxx \
                        --with-occ --with-netcdf --enable-less-tests\
                        --with-vtk --with-vtk_version_number=${vtk_library_version} \
                        LDFLAGS="-L/usr/lib/" \
                        CPPFLAGS="-I/usr/include/opencascade -I/usr/local/include/vtk-${vtk_library_version}"; \
make -j V=0; \
make install -j; \
fi

