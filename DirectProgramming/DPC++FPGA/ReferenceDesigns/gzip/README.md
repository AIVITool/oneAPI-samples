# GZIP Compression
This reference design demonstrates high-performance GZIP compression on FPGA.

| Optimized for                     | Description
---                                 |---
| OS                                | Linux* Ubuntu* 18.04/20.04 <br> RHEL*/CentOS* 8 <br> SUSE* 15 <br>Windows* 10
| Hardware                          | Intel&reg; Programmable Acceleration Card (PAC) with Intel Arria&reg; 10 GX FPGA; <br> Intel&reg; FPGA Programmable Acceleration Card (PAC) D5005 (with Intel Stratix&reg; 10 SX)
| Software                          | Intel&reg; oneAPI DPC++/C++ Compiler <br> Intel&reg; FPGA Add-On for oneAPI Base Toolkit
| What you will learn               | How to implement a high-performance multi-engine compression algorithm on FPGA
| Time to complete                  | 1 hr (not including compile time)


***Performance***: Please refer to the performance disclaimer at the end of this README.

| Device                                                | Throughput
|:---                                                   |:---
| Intel&reg; PAC with Intel Arria&reg; 10 GX FPGA               | 1 engine @ 3.4 GB/s
| Intel&reg; FPGA PAC D5005 (with Intel Stratix&reg; 10 SX)             | 2 engines @ 5.5 GB/s each = 11.0 GB/s total (High Bandwidth variant) using 120MB+ input, 2 engines @ 3.5 GB/s = 7.0 GB/s (Low Latency variant) using 80kB input


## Purpose

This reference design implements a compression algorithm. The implementation is optimized for the FPGA device. The compression result is GZIP-compatible and can be decompressed with GUNZIP. The GZIP output file format is compatible with GZIP's DEFLATE algorithm and follows a fixed subset [RFC 1951](https://www.ietf.org/rfc/rfc1951.txt). See the References section for more specific references.

The algorithm uses a GZIP-compatible Limpel-Ziv 77 (LZ77) algorithm for data de-duplication and a GZIP-compatible Static Huffman algorithm for bit reduction. The implementation includes three FPGA accelerated tasks (LZ77, Static Huffman and CRC).

The FPGA implementation of the algorithm enables either one or two independent GZIP compute engines to operate in parallel on the FPGA. The available FPGA resources constrain the number of engines. By default, the design is parameterized to create a single engine when the design is compiled to target Intel&reg; PAC with Intel Arria&reg; 10 GX FPGA. Two engines are created when compiling for Intel&reg; FPGA PAC D5005 (with Intel Stratix&reg; 10 SX), a larger device.

This reference design contains two variants: "High Bandwidth" and "Low-Latency."
The High Bandwidth variant maximizes system throughput without regard for latency. It transfers input/output SYCL Buffers to FPGA-attached DDR. The kernel then operates on these buffers.
The Low-Latency variant takes advantage of Universal Shared Memory (USM) to avoid these copy operations, allowing the GZIP engine to directly access input/output buffers in host-memory. This reduces latency, but throughput is also reduced. "Latency" in this context is defined as the duration of time between when the input buffer is available in host memory to when the output buffer (i.e., the compressed result) is available in host memory.
The Low-Latency variant is only supported on Stratix&reg; 10 SX.

### Additional Documentation
- [Explore SYCL* Through Intel&reg; FPGA Code Samples](https://software.intel.com/content/www/us/en/develop/articles/explore-dpcpp-through-intel-fpga-code-samples.html) helps you to navigate the samples and build your knowledge of FPGAs and SYCL.
- [FPGA Optimization Guide for Intel&reg; oneAPI Toolkits](https://software.intel.com/content/www/us/en/develop/documentation/oneapi-fpga-optimization-guide) helps you understand how to target FPGAs using SYCL and Intel&reg; oneAPI Toolkits.
- [Intel&reg; oneAPI Programming Guide](https://software.intel.com/en-us/oneapi-programming-guide) helps you understand target-independent, SYCL-compliant programming using Intel&reg; oneAPI Toolkits.

## Key Implementation Details

 | Kernel                     | Description
---                          |---
| LZ Reduction               | Implements an LZ77 algorithm for data de-duplication. The algorithm produces distance and length information that is compatible with GZIP's DEFLATE implementation.
| Static Huffman             | Uses the same Static Huffman codes used by GZIP's DEFLATE algorithm when it chooses a Static Huffman coding scheme for bit reduction. This choice maintains compatibility with GUNZIP.
| CRC                        | Adds a CRC checksum based on the input file; the gzip file format requires this

To optimize performance, GZIP leverages techniques discussed in the following FPGA tutorials:
* **Double Buffering to Overlap Kernel Execution with Buffer Transfers and Host Processing** (double_buffering)
* **On-Chip Memory Attributes** (mem_config)



## Building the `gzip` Reference Design

> **Note**: If you have not already done so, set up your CLI
> environment by sourcing  the `setvars` script located in
> the root of your oneAPI installation.
>
> Linux*:
> - For system wide installations: `. /opt/intel/oneapi/setvars.sh`
> - For private installations: `. ~/intel/oneapi/setvars.sh`
>
> Windows*:
> - `C:\Program Files(x86)\Intel\oneAPI\setvars.bat`
>
>For more information on environment variables, see Use the setvars Script for [Linux or macOS](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-programming-guide/top/oneapi-development-environment-setup/use-the-setvars-script-with-linux-or-macos.html), or [Windows](https://www.intel.com/content/www/us/en/develop/documentation/oneapi-programming-guide/top/oneapi-development-environment-setup/use-the-setvars-script-with-windows.html).


### Running Samples in Intel&reg; DevCloud
If running a sample in the Intel&reg; DevCloud, remember that you must specify the type of compute node and whether to run in batch or interactive mode. Compiles to FPGA are only supported on fpga_compile nodes. Executing programs on FPGA hardware is only supported on fpga_runtime nodes of the appropriate type, such as fpga_runtime:arria10 or fpga_runtime:stratix10.  Neither compiling nor executing programs on FPGA hardware are supported on the login nodes. For more information, see the Intel&reg; oneAPI Base Toolkit Get Started Guide ([https://devcloud.intel.com/oneapi/documentation/base-toolkit/](https://devcloud.intel.com/oneapi/documentation/base-toolkit/)).

When compiling for FPGA hardware, it is recommended to increase the job timeout to 24h.


### Using Visual Studio Code*  (Optional)

You can use Visual Studio Code (VS Code) extensions to set your environment, create launch configurations,
and browse and download samples.

The basic steps to build and run a sample using VS Code include:
 - Download a sample using the extension **Code Sample Browser for Intel oneAPI Toolkits**.
 - Configure the oneAPI environment with the extension **Environment Configurator for Intel oneAPI Toolkits**.
 - Open a Terminal in VS Code (**Terminal>New Terminal**).
 - Run the sample in the VS Code terminal using the instructions below.

To learn more about the extensions and how to configure the oneAPI environment, see
[Using Visual Studio Code with Intel&reg; oneAPI Toolkits](https://software.intel.com/content/www/us/en/develop/documentation/using-vs-code-with-intel-oneapi/top.html).

After learning how to use the extensions for Intel oneAPI Toolkits, return to this readme for instructions on how to build and run a sample.

### On a Linux* System

1. Generate the `Makefile` by running `cmake`.
     ```
   mkdir build
   cd build
   ```
   To compile for the Intel&reg; PAC with Intel Arria&reg; 10 GX FPGA, run `cmake` using the command:
    ```
    cmake ..
   ```
   Alternatively, to compile for the Intel&reg; FPGA PAC D5005 (with Intel Stratix&reg; 10 SX), run `cmake` using the command:

   ```
   cmake .. -DFPGA_DEVICE=intel_s10sx_pac:pac_s10_usm
   ```
   To compile for the Low Latency version of the design use the command

   ```
   cmake .. -DLOW_LATENCY=1 -DFPGA_DEVICE=intel_s10sx_pac:pac_s10_usm
   ```
2. Compile the design through the generated `Makefile`. The following build targets are provided, matching the recommended development flow:

   * Compile for emulation (fast compile time, targets emulated FPGA device):
      ```
      make fpga_emu
      ```

   * Generate the optimization report:
     ```
     make report
     ```

   * Compile for FPGA hardware (longer compile time, targets FPGA device):
     ```
     make fpga
     ```
3. (Optional) As the above hardware compile may take several hours to complete, FPGA precompiled binaries (compatible with Linux* Ubuntu* 18.04) can be downloaded <a href="https://iotdk.intel.com/fpga-precompiled-binaries/latest/gzip.fpga.tar.gz" download>here</a>.

### On a Windows* System

1. Generate the `Makefile` by running `cmake`.
     ```
   mkdir build
   cd build
   ```
   To compile for the Intel&reg; PAC with Intel Arria&reg; 10 GX FPGA, run `cmake` using the command:
    ```
    cmake -G "NMake Makefiles" ..
   ```
   Alternatively, to compile for the Intel&reg; FPGA PAC D5005 (with Intel Stratix&reg; 10 SX), run `cmake` using the command:

   ```
   cmake -G "NMake Makefiles" .. -DFPGA_DEVICE=intel_s10sx_pac:pac_s10_usm
   ```
   To compile for the Low Latency version of the design use the command:

   ```
   cmake -G "Nmake Makefiles" .. -DLOW_LATENCY=1 -DFPGA_DEVICE=intel_s10sx_pac:pac_s10_usm
   ```

2. Compile the design through the generated `Makefile`. The following build targets are provided, matching the recommended development flow:

   * Compile for emulation (fast compile time, targets emulated FPGA device):
     ```
     nmake fpga_emu
     ```
   * Generate the optimization report:
     ```
     nmake report
     ```
   * Compile for FPGA hardware (longer compile time, targets FPGA device):
     ```
     nmake fpga
     ```

*Note:* The Intel&reg; PAC with Intel Arria&reg; 10 GX FPGA and Intel&reg; FPGA PAC D5005 (with Intel Stratix&reg; 10 SX) do not yet support Windows*. Compiling to FPGA hardware on Windows* requires a third-party or custom Board Support Package (BSP) with Windows* support.<br>
*Note:* If you encounter any issues with long paths when compiling under Windows*, you may have to create your ‘build’ directory in a shorter path, for example c:\samples\build.  You can then run cmake from that directory, and provide cmake with the full path to your sample directory.

If an error occurs, you can get more details by running `make` with
the `VERBOSE=1` argument:
``make VERBOSE=1``
For more comprehensive troubleshooting, use the Diagnostics Utility for
Intel&reg; oneAPI Toolkits, which provides system checks to find missing
dependencies and permissions errors.
[Learn more](https://software.intel.com/content/www/us/en/develop/documentation/diagnostic-utility-user-guide/top.html).


 ### In Third-Party Integrated Development Environments (IDEs)

You can compile and run this tutorial in the Eclipse* IDE (in Linux*) and the Visual Studio* IDE (in Windows*). For instructions, refer to the following link: [FPGA Workflows on Third-Party IDEs for Intel&reg; oneAPI Toolkits](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-oneapi-dpcpp-fpga-workflow-on-ide.html)


## Running the Reference Design

 1. Run the sample on the FPGA emulator (the kernel executes on the CPU):
     ```
     ./gzip.fpga_emu <input_file> [-o=<output_file>]     (Linux)
     gzip.fpga_emu.exe <input_file> [-o=<output_file>]   (Windows)
     ```
2. Run the sample on the FPGA device:
     ```
     aocl initialize acl0 pac_s10_usm
     ./gzip.fpga <input_file> [-o=<output_file>]         (Linux)
     gzip.fpga.exe <input_file> [-o=<output_file>]       (Windows)
     ```
 ### Application Parameters

| Argument | Description
---        |---
| `<input_file>` | Mandatory argument that specifies the file to be compressed. Use a 120+ MB file to achieve peak performance (80kB for Low Latency variant).
| `-o=<output_file>` | Optional argument that specifies the name of the output file. The default name of the output file is `<input_file>.gz`. When targeting Intel&reg; FPGA PAC D5005 (with Intel Stratix&reg; 10 SX), the single `<input_file>` is fed to both engines, yielding two identical output files, using `<output_file>` as the basis for the filenames.

### Example of Output

```
Running on device:  pac_a10 : Intel PAC Platform (pac_ee00000)
Throughput: 3.4321 GB/s
Compression Ratio 33.2737%
PASSED
```
## Additional Design Information
### Source Code Explanation

| File                         | Description
---                            |---
| `gzip.cpp`                   | Contains the `main()` function and the top-level interfaces to the SYCL* GZIP functions.
| `gzip_ll.cpp`                | Low latency variant of the top level file.
| `gzipkernel.cpp`             | Contains the SYCL* kernels used to implement GZIP.
| `gzipkernel_ll.cpp`          | Low-latency variant of kernels.
| `CompareGzip.cpp`            | Contains code to compare a GZIP-compatible file with the original input.
| `WriteGzip.cpp`              | Contains code to write a GZIP compatible file.
| `crc32.cpp`                  | Contains code to calculate a 32-bit CRC compatible with the GZIP file format and to combine multiple 32-bit CRC values. It is only used to account for the CRC of the last few bytes in the file, which are not processed by the accelerated CRC kernel.
| `kernels.hpp`                  | Contains miscellaneous defines and structure definitions required by the LZReduction and Static Huffman kernels.
| `crc32.hpp`                    | Header file for `crc32.cpp`.
| `gzipkernel.hpp`              | Header file for `gzipkernels.cpp`.
| `gzipkernel)ll.hpp`              | Header file for `gzipkernels_ll.cpp`.
| `CompareGzip.hpp`              | Header file for `CompareGzip.cpp`.
| `pipe_utils.hpp`             | Header file containing the definition of an array of pipes. This header can be found in the DirectProgramming/DPC++FPGA/include/ directory of this repository.
| `WriteGzip.hpp`                | Header file for `WriteGzip.cpp`.

### Compiler Flags Used

| Flag | Description
---    |---
`-Xshardware` | Target FPGA hardware (as opposed to FPGA emulator)
`-Xsparallel=2` | Uses two cores when compiling the bitstream through Quartus&reg;
`-Xsseed=<seed_num>` | Uses a particular seed while running Quartus&reg;, selected to yield the best Fmax for this desgin
`-Xsnum-reorder=6` | On Intel Stratix&reg; 10 SX only, specify a wider data path for read data from global memory
`-Xsopt-arg="-nocaching"` | Specifies that cached LSUs should not be used.
`-DNUM_ENGINES=<1|2>` | Specifies that 1 GZIP engine should be compiled when targeting Intel Arria&reg; 10 GX and two engines when targeting Intel Stratix&reg; 10 SX


### Performance disclaimers

Tests document the performance of components on a particular test on a specific system. Differences in hardware, software, or configuration will affect actual performance. Consult other sources of information to evaluate performance as you consider your purchase.  For complete information about performance and benchmark results, visit [this page](https://edc.intel.com/content/www/us/en/products/performance/benchmarks/overview).

Performance results are based on testing as of October 27, 2020 (using tool version 2021.1), and may not reflect all publicly available security updates.  See configuration disclosure for details.  No product or component can be absolutely secure.

Intel technologies’ features and benefits depend on system configuration and may require enabled hardware, software or service activation. Performance varies depending on system configuration. Check with your system manufacturer or retailer or learn more at [intel.com](www.intel.com).

Intel measured the performance on October 27, 2020 (using tool version 2021.1).

Intel and the Intel logo are trademarks of Intel Corporation or its subsidiaries in the U.S. and/or other countries.

&copy; Intel Corporation.

###  Additional References
- [Khronos SYCL Resources](https://www.khronos.org/sycl/resources)
- [Intel GZIP OpenCL Design Example](https://www.intel.com/content/www/us/en/programmable/support/support-resources/design-examples/design-software/opencl/gzip-compression.html)
- [RFC 1951 - DEFLATE Data Format](https://www.ietf.org/rfc/rfc1951.txt)
- [RFC 1952 - GZIP Specification 4.3](https://www.ietf.org/rfc/rfc1952.txt)
- [OpenCL Intercept Layer](https://github.com/intel/opencl-intercept-layer)

## License
Code samples are licensed under the MIT license. See
[License.txt](https://github.com/oneapi-src/oneAPI-samples/blob/master/License.txt) for details.

Third party program Licenses can be found here: [third-party-programs.txt](https://github.com/oneapi-src/oneAPI-samples/blob/master/third-party-programs.txt).
