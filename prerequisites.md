# Metal FS Prerequisites

These are the current prerequisites for developing Metal FS applications (on an `amd64` host):

 - Xilinx Vivado 2018.1
   - Requires specific OS versions, e.g. Ubuntu 16.04
   - As prescribed by the underlying SNAP framework
   - The WebPACK edition is sufficient for simulating operators
 - The following packages, assuming Ubuntu 16.04
   - `build-essential cmake libfuse-dev liblmdb-dev libprotobuf-dev protobuf-compiler`
   - For SNAP: `git xterm tmux libncurses-dev`
   - libjq: [https://github.com/stedolan/jq](https://github.com/stedolan/jq)
   - GCC 7, must be installed from an unofficial package source
   - [node.js and npm](https://github.com/nodesource/distributions/blob/master/README.md#deb) for using third party operator packages
