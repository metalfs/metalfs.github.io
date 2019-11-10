# Welcome

Metal FS is a near-storage compute framework that combines FPGA compute kernels ('operators') with a file system.

It currently supports CAPI-enabled FPGA cards in IBM POWER systems.

Development and simulation of operators is possible using only free tools (x86_64).

## Getting started

For operator development you can either use a [Dockerized development environment](docker_dev.html) (recommended), or set up the [prerequisites](prerequisites.html) manually.

The [tutorial](tutorial.html) will cover how to create a Metal FS FPGA image that consists of pre-defined operators as well as a custom image transformation kernel.

## Running on POWER

If you have access to a POWER8/POWER9 system equipped with a suitable FPGA, please follow the [deployment guide](deployment.html).

## Advanced Topics

Refer to these guides for advanced Metal FS use cases:

- Image Manifest Documentation
- Operator Manifest Documentation
- Operator Pipeline API (C++)
  - Basic usage
  - Filesystem Data Source / Data Sink
- Inspecting Simulation Waveforms
- Operator Profiling at Runtime
- Operator Hardware Interface
- Benchmarking Data Generator
- Operator Buildpacks
- FPGA Targets

## Feedback

We welcome your feedback to Metal FS!

If you have questions or remarks, please create an [issue](https://github.com/osmhpi/metal_fs/issues) on the main [GitHub](https://github.com/osmhpi/metal_fs) repo.
