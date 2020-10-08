########################################################

# Emscripten Environment 

FROM emscripten/emsdk

########################################################

# Install Dependencies 
RUN     apt-get update
RUN     apt-get install -y make
RUN     apt-get install -y \
          apt-utils \
          curl \
          build-essential \
          cmake \
          pkg-config \
          git \
          ncurses-dev libreadline-dev libedit-dev \
          libgoogle-perftools-dev \
          libunwind-dev \
          libgmp-dev \
          libssl-dev \
          unixodbc-dev \
          zlib1g-dev libarchive-dev \
          libossp-uuid-dev \
          libxext-dev libice-dev libjpeg-dev libxinerama-dev libxft-dev \
          libxpm-dev libxt-dev \
          libdb-dev \
          libpcre3-dev \
          libyaml-dev \
          python3 \
          git 

########################################################       

# Configure Emsdk

RUN     emsdk install latest-upstream
RUN     emsdk activate latest-upstream

########################################################

# Build ZLib   

WORKDIR    /
RUN        wget https://zlib.net/zlib-1.2.11.tar.gz
RUN        tar -xf zlib-1.2.11.tar.gz
RUN        ls -l
WORKDIR    /zlib-1.2.11
RUN        emconfigure ./configure
RUN        emmake make

########################################################

# Build SWIPL       
WORKDIR   /
RUN       git clone https://github.com/SWI-Prolog/swipl-devel.git
WORKDIR   /swipl-devel
RUN       git submodule update --init

## Development Build...

RUN       mkdir build
WORKDIR   /swipl-devel/build
RUN       cmake -DSWIPL_PACKAGES_JAVA=OFF ..
RUN       make && make install

##  WASM Build...

WORKDIR   /swipl-devel
RUN       mkdir build.wasm
WORKDIR   /swipl-devel/build.wasm 

WORKDIR   /emsdk

ARG       CMAKE_INSTALL_PREFIX=/usr/local
ARG       EMSDK=/emsdk_portable
ARG       ZLIB='/zlib-1.2.11'

RUN       cmake -DCMAKE_TOOLCHAIN_FILE=${EMSDK}/upstream/emscripten/cmake/Modules/Platform/Emscripten.cmake \
          -DCMAKE_BUILD_TYPE=Release \
          -DZLIB_LIBRARY=${ZLIB}/libz.a \
          -DZLIB_INCLUDE_DIR=${ZLIB} \
          -DMULTI_THREADED=OFF \
          -DUSE_SIGNALS=OFF \
          -DUSE_GMP=OFF \
          -DBUILD_SWIPL_LD=OFF \
          -DSWIPL_PACKAGES=OFF \
          -DINSTALL_DOCUMENTATION=OFF \
          -DSWIPL_NATIVE_FRIEND=build \
          -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX} \
          .. 

RUN       emmake make

