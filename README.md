# SWI-Prolog ported to WebAssembly

This repository contains instructions to compile a Prolog
implementation SWI-Prolog (<http://swi-prolog.org>) to
WebAssembly.

## Compilation

These compilation instructions assume Linux-based
host machine. The resulting WebAssembly binary is
platform-independent. 

_Note: This build will NOT work on a Raspberry PI due to the non-64-bit architecture._

### Preparation

You need to download the Emscripten compiler. Follow
the instruction on its [homepage][em-install].

[em-install]:http://kripken.github.io/emscripten-site/docs/getting_started/downloads.html

__this build will place certain source files in your $HOME.__


```sh
########################################################

# Install Dependencies 

apt update
apt upgrade

apt-get install -y \
	apt-utils \
	build-essential cmake pkg-config \
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
	python3

########################################################

# Build ZLib   
wget https://zlib.net/zlib-1.2.11.tar.gz -O "$HOME/zlib-1.2.11.tar.gz"
tar -xf "$HOME/zlib-1.2.11.tar.gz" -C "$HOME"
cd "$HOME/zlib-1.2.11"
emconfigure ./configure
emmake make

########################################################

# Build Emscripten   
cd $HOME
git clone https://github.com/emscripten-core/emsdk.git
cd emsdk
./emsdk install latest
./emsdk activate latest

# After the successful installation load the Emscripten
#environment into the current terminal session (adjust path):
source ./emsdk_env.sh

########################################################

# Build SWIPL       
cd $HOME
git clone https://github.com/SWI-Prolog/swipl-devel.git
cd swipl-devel
git submodule update --init

## 'Development' Build...
mkdir build
cd build
cmake -DSWIPL_PACKAGES_JAVA=OFF ..
make
make install

##  'WASM' 'Development' Build...
cd $HOME/swipl-devel
mkdir build.wasm
cd build.wasm
cmake -DCMAKE_TOOLCHAIN_FILE=$HOME/emsdk/upstream/emscripten/cmake/Modules/Platform/Emscripten.cmake \
      -DCMAKE_BUILD_TYPE=Release \
		  -DZLIB_LIBRARY=$HOME/zlib-1.2.11/libz.a \
		  -DZLIB_INCLUDE_DIR=$HOME/zlib-1.2.11 \
      -DMULTI_THREADED=OFF \
      -DUSE_SIGNALS=OFF \
      -DUSE_GMP=OFF \
      -DBUILD_SWIPL_LD=OFF \
      -DSWIPL_PACKAGES=OFF \
      -DINSTALL_DOCUMENTATION=OFF \
      -DSWIPL_NATIVE_FRIEND=build \
      -G "Unix Makefiles" ..
      
emmake make


########################################################

# Move WASM Source Files to Build (to allow friend-build dependencies)
#


# ... and all of the other WASMs?
```


## Usage

Please see "Foreign Language Interface" (FLI) in the SWI-Prolog manual. A very limited
set of function findings into JavaScript can be seen in the demo.
The bindings use [cwrap][cwrap] from Emscripten.

[cwrap]:https://kripken.github.io/emscripten-site/docs/api_reference/preamble.js.html#cwrap

General workflow:

 * Set up a stub [Module object][module] with `noInitialRun: true` and other options.
 * Set up FLI bindings in `Module.onRuntimeInitialized`.
 * Use the bindings to call `PL_initialise`.
 * Set up the location for standard library.
 * Use the [FS API][fs] to write code files into the virtual filesystem.
 * Load the code files from SWI-Prolog side using `consult/1` or a similar way.
 * Interact with SWI-Prolog through its FLI.

[module]:https://kripken.github.io/emscripten-site/docs/api_reference/module.html
[fs]:https://kripken.github.io/emscripten-site/docs/api_reference/Filesystem-API.html

### Demo

See `example/index.html` as a simple example. It can be found online at
<http://demos.rlaanemets.com/swi-prolog-wasm/example/>. The commented code
inside the demo provides the documentation.

To test it out locally, you need to serve the files through an HTTP server.

## TODO

 * Provide full set of bindings?
 * A way to call JavaScript from SWI-Prolog.
 * Compile and add useful packages.
 * Provide a mechanism to load packs?
 * Easier way to turn Prolog terms into JS objects?
 * See where WebAssembly goes and what interfaces
   could be added (direct DOM access?).

## License

SWI-Prolog is covered with the Simplified BSD license. See <http://www.swi-prolog.org/license.html>

zlib is covered with the zlib license. See <https://zlib.net/zlib_license.html>

