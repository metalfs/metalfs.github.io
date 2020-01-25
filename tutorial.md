# Tutorial 

This tutorial guides you through the process of creating a simple operator for Metal FS.

Please make sure that you have prepared your system either by pulling the development Docker Container or by manually setting up the [prerequisites](prerequisites.html).

Afterwards, clone the repository accompanying this tutorial from GitHub: [https://github.com/metalfs/getting-started](https://github.com/metalfs/getting-started)

### Project Structure

The project is structured as follows:
```
- Makefile
- image.json
- src
  `-- hls_operator_colorfilter
      |-- Makefile
      |-- hls_operator_colorfilter.cpp
      |-- operator.json
      `-- testbench.cpp
```

The [`image.json`](https://github.com/metalfs/getting-started/tree/master/image.json) file contains the project configuration: It specifies which FPGA to target and which operators to include in the image.

> Note: The default configuration does not enable the DRAM and NVMe capabilities of the N250S FPGA card.

The root [Makefile](https://github.com/metalfs/getting-started/tree/master/Makefile) offers access to builtin SNAP targets (e.g. `make model`, `make sim`, `make image`).

Under [`src/hls_operator_colorfilter`](https://github.com/metalfs/getting-started/tree/master/src/hls_operator_colorfilter), an example operator is defined that converts a bitmap image to grayscale.

Its [`operator.json`](https://github.com/metalfs/getting-started/tree/master/src/hls_operator_colorfilter/operator.json) file contains the operator manifest which is used at build time to describe the operator's configuration interface. Furthermore it specifies the command line options to be used at runtime.

[`hls_operator_colorfilter.cpp`](https://github.com/metalfs/getting-started/tree/master/src/hls_operator_colorfilter/hls_operator_colorfilter.cpp) contains the operator implementation in Vivado HLS. [`testbench.cpp`](https://github.com/metalfs/getting-started/tree/master/src/hls_operator_colorfilter/testbench.cpp) can be used to test the implementation in software by running `make test` from the operator directory.

### How to try it out

Once you have started the development Docker container, run `make model` to generate a simulation model. Afterwards, start a simulation environment with `make sim`.

In the simulation shell, first initialize the environment with `snap_maint`, then start the file system driver: `metal_fs /mnt`.

Start a second shell in the container and try out the operator with the provided bitmap file:
```
cat src/hls_operator_colorfilter/apples_simulation.bmp \
  | /mnt/operators/colorfilter \
  > out.bmp
```
