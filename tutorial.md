# Tutorial 

This tutorial walks you through the process of creating a simple operator for Metal FS.

For your reference, the resulting project of this tutorial are available on GitHub: [https://github.com/metalfs/getting-started](https://github.com/metalfs/getting-started)

## Step 1: Set up the Development Environment

During this tutorial, we will develop a simple HLS operator that transforms a stream of ASCII characters into uppercase characters.

We first start with an empty folder for our 'uppercase' operator: 
```
mkdir uppercase && cd uppercase
```

To get you started quickly, we recommend using a development environment that is backed by a Docker container.
It integrates nicely with Visual Studio Code and its 'Remote - Containers' extension.

If you'd rather like to use raw `docker-compose`, please refer to our [guide](docker_dev) for more details.

This downloads the container configuration files:

```bash
mkdir .devcontainer

wget -o .devcontainer/docker-compose.yml \
  https://raw.githubusercontent.com/metalfs/getting-started/master/.devcontainer/docker-compose.yml 

wget -o .devcontainer/devcontainer.json \
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

```
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
Therefore, we add a testbench file reference to our `Makefile`:

```Makefile
testbench_srcs += testbench.cpp
```

This is our `testbench.cpp`:
```cpp
void uppercase(mtl_stream &in, mtl_stream &out);

int main() {
  const char input[] = "This should become uppercase";

  // Transform our input data into a mtl_stream
  int readBytes = 0;
  mtl_stream operatorInput; 
  mtl_stream_element inputElement;
  do {
    memcpy(&inputElement.data, input + readBytes, 
        std::min(sizeof(input) - readBytes, sizeof(inputElement.data)));
    inputElement.keep = 0xff;
    inputElement.last = readBytes + sizeof(inputElement.data) >= sizeof(input);

    operatorInput.write(inputElement);

    readBytes += sizeof(inputElement.data);
  } while (!inputElement.last);

  // Call the operator
  mtl_stream operatorOutput;
  colorfilter(operatorInput, operatorOutput);

  // Read the output back into a buffer for comparison
  char outputData[sizeof(input)];
  int writtenBytes = 0;
  mtl_stream_element outputElement;
  do {
    outputElement = operatorOutput.read();
    memcpy(outputData + writtenBytes, &outputElement.data, 
        std::min(sizeof(outputData) - writtenBytes, sizeof(outputElement.data)));

    writtenBytes += sizeof(outputElement.data);
  } while (!outputElement.last);

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
TODO
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