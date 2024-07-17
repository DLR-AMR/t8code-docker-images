FROM dlramr/t8code-ubuntu:t8-dependencies
LABEL org.opencontainers.image.authors="Sandro Elsweijer <sandro.elsweijer@dlr.de>"
LABEL Description="Ubuntu 20.04 with VTK, NetCDF and OpenCASCADE setup and t8code"
SHELL ["/bin/bash", "-c"]

ARG t8code_version=1.0.0
ARG vtk_version=9.1.0
ARG vtk_library_version=9.1
ARG occt_version=7_8_1

ENV DEBIAN_FRONTEND noninteractive
ENV LIBRARY_PATH=/usr/local/lib:$LIBRARY_PATH
ENV LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH


RUN cd ~; \
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
                        LDFLAGS="-L/usr/lib/ -L/usr/local/lib" \
                        CPPFLAGS="-isystem/usr/local/include/opencascade -isystem/usr/local/include/vtk-${vtk_library_version}"; \
make -j4 V=0; \
make install -j4; \
fi

