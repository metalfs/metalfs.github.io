# Metal FS

Metal FS is a near-storage compute framework that combines FPGA compute kernels ('operators') with a file system.
It is open-source on [GitHub](https://github.com/osmhpi/metalfs).

It currently supports CAPI-enabled FPGA cards in IBM POWER systems.

Development and simulation of operators is possible using only free tools (x86_64).

## Getting started

The [tutorial](tutorial.html) will cover how to create a Metal FS FPGA image that consists of pre-defined operators as well as a custom image transformation kernel.

For operator development you can either use a [Dockerized development environment](docker_dev.html) (recommended), or set up the [prerequisites](prerequisites.html) manually.


## Running on POWER

If you have access to a POWER8/POWER9 system equipped with a suitable FPGA, please follow the [deployment guide](deployment.html).

## Advanced Topics

Refer to these guides for advanced Metal FS use cases:

- Operator Parameters
- [Image Manifest Documentation](image_manifest)
- [Operator Manifest Documentation](operator_manifest)
- Operator Pipeline API (C++)
  - Basic usage
  - Filesystem Data Source / Data Sink
- Inspecting Simulation Waveforms
- Operator Profiling at Runtime
- Operator Hardware Interface
- Benchmarking Data Generator
- [Operator Buildpacks](buildpacks)
- FPGA Targets

## Feedback

We welcome your feedback to Metal FS!

If you have questions or remarks, please create an [issue](https://github.com/osmhpi/metalfs/issues) on the main [GitHub](https://github.com/osmhpi/metalfs) repo.
