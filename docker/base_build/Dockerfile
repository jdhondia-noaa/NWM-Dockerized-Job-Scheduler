FROM alpine:3.10
MAINTAINER aaraney@crimson.ua.edu

# This container contains the requirements to compile
# wrf-hydro/nwm. It was designed in such a way so it may
# be pulled as used not to lock users into a single version
# of the nwm.

USER root

# Environment install vars
ENV HYDRA_HOST_FILE /etc/opt/hosts
ENV H5DIR=/usr/lib/x86_64-linux-gnu/hdf5/serial
ENV HDF5_DIR=/usr/lib/x86_64-linux-gnu/hdf5/serial
ENV NETCDF=/usr/local
ENV NCDIR=/usr/local
ENV NETCDF_LIB=/usr/local/lib
ENV NETCDF_INC=/usr/local/include
ENV NFDIR=/usr/local
ENV LD_LIBRARY_PATH=${NCDIR}/lib

## Install dependency packages
## could probably remove some of these
RUN apk update --no-cache \
    && apk add --no-cache\
    file wget \
    gfortran g++ \
    m4 make \
    # Install HDF5
    && apk add --no-cache --repository http://dl-3.alpinelinux.org/alpine/edge/testing hdf5-dev hdf5 \
    # Download, build, and install MPICH
    && MPICH_VERSION="3.2" \
    && MPICH_CONFIGURE_OPTIONS="" \
    && MPICH_MAKE_OPTIONS='-j 2' \
    && mkdir /tmp/mpich-src \
    && cd /tmp/mpich-src \
    && wget http://www.mpich.org/static/downloads/${MPICH_VERSION}/mpich-${MPICH_VERSION}.tar.gz \
    && tar xfz mpich-${MPICH_VERSION}.tar.gz  \
    && cd mpich-${MPICH_VERSION}  \
    && ./configure ${MPICH_CONFIGURE_OPTIONS}  \
    && make ${MPICH_MAKE_OPTIONS} && make install \
    && rm -rf /tmp/mpich-src \
    # Install netCDF-C
    && NETCDF_C_VERSION="4.4.1.1" \
    && wget ftp://ftp.unidata.ucar.edu/pub/netcdf/netcdf-${NETCDF_C_VERSION}.tar.gz -P /tmp \
    && tar -xf /tmp/netcdf-${NETCDF_C_VERSION}.tar.gz -C /tmp \
    && cd /tmp/netcdf-${NETCDF_C_VERSION} \
    && CPPFLAGS=-I${H5DIR}/include LDFLAGS=-L${H5DIR}/lib ./configure --prefix=/usr/local \
    && cd /tmp/netcdf-${NETCDF_C_VERSION} \
    && make -j 2\
    && cd /tmp/netcdf-${NETCDF_C_VERSION} \
    && make install \
    && rm -rf /tmp/netcdf-${NETCDF_C_VERSION} \
    # Echo number of cpus for mpi to use
    && echo "localhost:$(grep processor /proc/cpuinfo | wc -l)" > ${HYDRA_HOST_FILE} \
    && chown root:wheel ${HYDRA_HOST_FILE} \
    # Install netcdf fortran
    && NETCDF_F_VERSION="4.4.4" \
    && cd /tmp \
    && wget ftp://ftp.unidata.ucar.edu/pub/netcdf/netcdf-fortran-${NETCDF_F_VERSION}.tar.gz \
    && tar -xf netcdf-fortran-${NETCDF_F_VERSION}.tar.gz \
    && cd /tmp/netcdf-fortran-${NETCDF_F_VERSION} \
    && CPPFLAGS=-I${NCDIR}/include LDFLAGS=-L${NCDIR}/lib ./configure --prefix=${NFDIR} \
    && make -j 2\
    && make install \
    && cd / \
    && rm -rf /tmp/netcdf-fortran-${NETCDF_F_VERSION} \
    # Remove packages
    && apk del -r \
    wget make \
    gfortran m4 \
    file g++

WORKDIR /

ENTRYPOINT ["sh"]

