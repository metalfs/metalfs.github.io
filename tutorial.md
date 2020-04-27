# Tutorial

This tutorial walks you through the process of creating a simple operator for Metal FS.

For your reference, the resulting project of this tutorial are available on GitHub: [https://github.com/metalfs/getting-started](https://github.com/metalfs/getting-started)

## Step 1: Set up the Development Environment

During this tutorial, we will develop a simple HLS operator that transforms a stream of ASCII characters into uppercase characters.

We first start with an empty folder for our 'uppercase' operator:

```bash
mkdir uppercase && cd uppercase
```

To get you started quickly, we recommend using a development environment that is backed by a Docker container.
It integrates nicely with Visual Studio Code and its 'Remote - Containers' extension.

If you'd rather like to use raw `docker-compose`, please refer to our [guide](docker_dev) for more details.

This downloads the container configuration files:

```bash
mkdir .devcontainer

wget -O .devcontainer/docker-compose.yml \
  https://raw.githubusercontent.com/metalfs/getting-started/master/.devcontainer/docker-compose.yml

wget -O .devcontainer/devcontainer.json \
  https://raw.githubusercontent.com/metalfs/getting-started/master/.devcontainer/devcontainer.json
```
Open the project directory in VS Code and select 'Reopen in Container' from the global menu (Ctrl/Cmd + Shift + P).

## Step 2: Bootstrap the Operator Project

To tie in the Metal FS [Operator buildpack](buildpacks) for HLS, we now create a `Makefile` with the following contents:

```Makefile
srcs           += uppercase.cpp

include $(METAL_ROOT)/buildpacks/hls/hls.mk
```

You can now run `make help` to see the available targets provided by the buildpack:

```
TODO
```

Before we get to the actual operator implementation, we need to provide an operator manifest in `operator.json`:

```json
{
  "main": "uppercase",
  "description": "Transform ASCII strings to uppercase."
}
```

The `main` attribute in the manifest refers to the entrypoint of our HLS code.

## Step 2: Implement the Operator in HLS

Now, the HLS implementation of our uppercase operator is very straightforward.
Here are the contents of the `uppercase.cpp` file:

```cpp
#include <metal/stream.h>

void uppercase(mtl_stream &in, mtl_stream &out) {
    #pragma HLS INTERFACE axis port=in name=axis_input
    #pragma HLS INTERFACE axis port=out name=axis_output
    #pragma HLS INTERFACE s_axilite port=return bundle=control

    mtl_stream_element element;
    do {
        element = in.read();

        for (int i = 0; i < sizeof(element.data); ++i)
        {
            // Select the ith byte from element
            auto current = element.data(i * 8 + 7, i * 8);

            // If current is lowercase, exchange it
            // by an uppercase letter
            if (current >= 'a' && current <= 'z') {
                element.data(i * 8 + 7, i * 8)
                  = current - ('a' - 'A');
            }
        }

        out.write(element);
    } while (!element.last);
}
```

Note how the `#pragma HLS INTERFACE` directives instruct the compiler to create the operator hardware interfaces.

Also note that in this code, we don't actually define how many bytes a stream element contains.
This is automatically inferred from the FPGA image configuration at build time.
Since we only perform bytewise processing and the streams always contain a full number of bytes, we don't care how many bytes a single stream element contains.

The HLS syntax for selecting a single byte from the data word by specifying the bit range (e.g. `lowest_byte = word(7, 0)`) might be familiar to you if you have experience with VHDL or Verilog.

## Step 3: Test the operator in a testbench

The benefit of HLS programming is that we can run our code as software to quickly see if it works.
Therefore, we add a testbench file reference to the top of our `Makefile`:

```Makefile
testbench_srcs += testbench.cpp
```

This is our `testbench.cpp`:
```cpp
#include <stdio.h>
#include <string.h>
#include <metal/stream.h>

// Forward-declere the operator entrypoint
void uppercase(mtl_stream &in, mtl_stream &out);

void copyBufferToStream(const char *buffer, size_t buffer_length, mtl_stream &stream) {
  size_t readBytes = 0;
  mtl_stream_element inputElement;
  do {
    memcpy(&inputElement.data, buffer + readBytes,
        std::min(buffer_length - readBytes, sizeof(inputElement.data)));
    inputElement.keep = 0xff;
    inputElement.last = readBytes + sizeof(inputElement.data) >= buffer_length;

    stream.write(inputElement);

    readBytes += sizeof(inputElement.data);
  } while (!inputElement.last);
}

void copyStreamToBuffer(mtl_stream &stream, char *buffer, size_t buffer_length) {
  size_t writtenBytes = 0;
  mtl_stream_element outputElement;
  do {
    outputElement = stream.read();
    memcpy(buffer + writtenBytes, &outputElement.data,
        std::min(buffer_length - writtenBytes, sizeof(outputElement.data)));

    writtenBytes += sizeof(outputElement.data);
  } while (!outputElement.last);
}

int main() {
  const char input[] = "This should become uppercase";

  // Transform our input data into a mtl_stream
  mtl_stream operatorInput;
  copyBufferToStream(input, sizeof(input), operatorInput);

  // Call the operator
  mtl_stream operatorOutput;
  uppercase(operatorInput, operatorOutput);

  // Read the output back into a buffer for comparison
  char outputData[sizeof(input)];
  copyStreamToBuffer(operatorOutput, outputData, sizeof(outputData));

  const char expected[] = "THIS SHOULD BECOME UPPERCASE";

  int result = memcmp(outputData, expected, sizeof(expected));
  if (result == 0) {
    printf("Success.\n");
  } else {
    printf("Failure: Output was different from expected value.\n");
  }


  return result;
}
```

Let's see if it works using `make test`. You will probably see lots of HLS compiler output, but towards the end you should find:
```
Success.
INFO: [SIM 211-1] CSim done with 0 errors.
INFO: [SIM 211-3] *************** CSIM finish ***************
INFO: [Common 17-206] Exiting vivado_hls at Mon Apr 27 17:39:19 2020...
```

It works!

## Step 3: Simulating the Hardware Description of the Operator

As the next step, we will create a simulation model of an entire FPGA image that contains our new operator.
On the first run, this takes some time (~10 min), since also the Metal FS HLS components need to be compiled to a hardware description.

Start the process with `make model`.

Once the model synthesis has finished, run `make sim` to start the simulation.
Afterwards, in a second terminal in the development container, you can try out the simulated operator:

```
$ echo "Hello World!" | /mtl/operators/uppercase
HELLO WORLD
```

Terminating the simulation is a bit cumbersome at this point:
```
killall metal-driver
```

## Next steps

 - Inspect the simulation waveform
 - [Create an image from multiple operators](image_manifest)
 - Add parameters to your operator
 - Integrate the operator into your C++ application
 - Learn more about operator profiling and benchmarking
 - [Write an operator using VHDL or Verilog](buildpacks)
