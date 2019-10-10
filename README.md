# NVBit (NVidia Binary Instrumentation Tool)
NVIDIA Corporation

NVBit is covered by the same End User License Agreement as that of the 
NVIDIA CUDA Toolkit. By using NVBit you agree to End User License Agreement
described in the EULA.txt file.

NVBit is not part of the official CUDA toolkit, but instead is a research prototype from the Architecture Research Group at NVIDIA and as such is provided as-is with no guarantee of support.

You can downlaod NVBit at https://github.com/NVlabs/NVBit/releases 

You can find NVBit's paper, published at the IEEE MICRO 2019 conference, at https://github.com/NVlabs/NVBit/releases/download/v1.0/MICRO_19_NVBit.pdf 

## Introduction
NVBit (NVidia Binary Instrumentation Tool) is a research prototype of a dynamic 
binary instrumentation library for NVIDIA GPUs.

NVBit provides a set of simple APIs that enable writing a variety of 
instrumentation tools. Example of instrumentation tools are: dynamic 
instruction counters, instruction tracers, memory reference tracers, 
profiling tools, etc.

NVBit allows writing instrumentation tools (which we call **NVBit tools**) 
that can inspect and modify the assembly code (SASS) of a GPU application 
without requiring recompilation, thus dynamic. NVBit allows instrumentation 
tools to inspect the SASS instructions of each function (\_\_global\_\_ or 
\_\_device\_\_) as it is loaded for the first time in the GPU. During this 
phase is possible to inject one or more instrumentation calls to arbitrary 
device functions before (or after) a SASS instruction. It is also possible to 
remove SASS instructions, although in this case NVBit does not guarantee that 
the application will continue to work correctly.

NVBit tries to be as low overhead as possible, although any injection of 
instrumentation function has an associated cost due to saving and restoring 
application state before and after jumping to/from the instrumentation 
function.

Because NVBit does not require application source code, any pre-compiled GPU 
application should work regardless of which compiler (or version) has been 
used (i.e. nvcc, pgicc, etc). NVBit supports Kepler, Maxwell, Pascal and 
Volta architectures (specifically SM >= 3.5 && SM <= 7.0) and it is developed 
for compute (tested with CUDA and OpenACC) under x86-64 and PPC64 Linux. 

## Getting Started with NVBit

NVBit is provided in a .tgz file containing this README file and three folders:
1. A ```core``` folder, which contains the main static library 
```libnvbit.a``` and various headers files (among which the ```nvbit.h``` 
file which contains all the main NVBit APIs declarations).
2. A ```tools``` folder, which contains various source code examples of NVBit 
tools. A new user of NVBit, after familiarizing with these pre-existing tools 
will typically make a copy of one of them and modify appropriately.
3. A ```test-apps``` folder, which contains a simple application that can be 
used to test NVBit tools. There is nothing special about this application, it 
is a simple vector addition program.

NVBit tools must be compiled with nvcc version >= 8.0 , using GCC 
version >= 5.3.0. 

To compile the NVBit tools simply type ```make``` from  inside the ```tools``` 
folder (make sure ```nvcc``` is in your PATH).
Compile the test application by typing ```make``` inside the ```test-apps``` 
folder.

## Using an NVBit tool

Before running an NVBit tool, make sure ```nvdisasm``` is in your PATH. In 
Ubuntu distributions this is typically done by adding /usr/local/cuda/bin or 
/usr/local/cuda-"version"/bin to the PATH environment variable.

To use an NVBit tool we simply LD_PRELOAD the tool before the application 
execution command. 

For instance if the application vector add runs natively as: 

```
./test-apps/vectoradd/vectoradd
``` 

and produces the following output: 

```
Final sum = 100000.000000; sum/n = 1.000000 (should be ~1)
```

we would use the NVBit tool which performs instruction count as follow:

```
LD_PRELOAD=./tools/instr_count/instr_count.so ./test-apps/vectoradd/vectoradd
```

The output for this command should be the following:

```no-highlight
------------- NVBit (NVidia Binary Instrumentation Tool) Loaded --------------
NVBit core environment variables (mostly for nvbit-devs):
             VERBOSE = 0 - if set, enables verbose messages level  (1,2,3,...)
           DUMP_SASS = 0 - if set, dumps SASS of original and patched kernel
           NOINSPECT = 0 - if set, skips function inspection and instrumentation
            WARNINGS = 0 - if set, enable warning prints
            NVDISASM = nvdisasm - override default nvdisasm found in PATH
            NOBANNER = 0 - if set, does not print this banner
-----------------------------------------------------------------------------
         INSTR_BEGIN = 0 - Beginning of the instruction interval where to apply instrumentation
           INSTR_END = 4294967295 - End of the instruction interval where to apply instrumentation
        KERNEL_BEGIN = 0 - Beginning of the kernel launch interval where to apply instrumentation
          KERNEL_END = 4294967295 - End of the kernel launch interval where to apply instrumentation
    COUNT_WARP_LEVEL = 1 - Count warp level or thread level instructions
    EXCLUDE_PRED_OFF = 0 - Exclude predicated off instruction from count
        TOOL_VERBOSE = 0 - Enable verbosity inside the tool
----------------------------------------------------------------------------------------------------
kernel 0 - vecAdd(double*, double*, double*, int) - #thread-blocks 98,  kernel instructions 50077, total instructions 50077
Final sum = 100000.000000; sum/n = 1.000000 (should be ~1)
```

As we can see, before the original output, there is a print showing the kernel 
call index "0", the kernel function prototype 
"vecAdd(double*, double*, double*, int)", total number of thread blocks launched
 in this kernel "98", the number of executed instructions in the kernel "50077", 
 and for the all application "50077".

When the application starts, also two banners are printed showing the environment
variables (and their current values) that can be used to control the NVBit core 
or the specific NVBit Tool.
Mostly of the NVBit core environment variable are used for core 
debugging/development purposes. 
Set the environment value NOBANNER=1 to disable the core banner if that 
information is not wanted. 

### Examples of NVBit Tools

As explained above, inside the ```tools``` folder there are few example of 
NVBit tools. Rather than describing all of them in this README file we refer 
to comment in the source code of each one them. 

The natural order (in terms of complexity) to learn these tools is:

1. instr_count: Perform thread level instruction count. Specifically, a 
function is injected before each SASS instruction. Inside this function the 
total number of active threads in a warp is computed and a global counter is 
incremented.

2. opcode_hist: Generate an histogram of all executed instructions.

3. mov_replace: Replace each SASS instruction of type MOV with an equivalent 
function. This tool make use of the read/write register functionality within 
the instrumentation function.

4. instr_countbb: Perform thread level instruction count by instrumenting 
basic blocks. The final result is the same as instr_count, but mush faster 
since less instructions are instrumented (only the first instruction in each 
basic block is instrumented and the counter).

5. mem_printf: Print memory reference addresses for each global LOAD/STORE 
using the GPU side printf. This is accomplished by injecting an 
instrumentation function before each SASS instruction performing global 
LOAD/STORE, passing the register values and immediate used by that 
instruction (used to compute the resulting memory address) and performing the 
printf. 

6. mem_trace: Trace memory reference addresses. This NVBit tool works 
similarly to the above example but instead of using a GPU side printf it uses 
a communication channel (provided in utils/channel.hpp) to transfer data from 
GPU-to-CPU and it performs the printf on the CPU side.

We also suggest to take a look to nvbit.h (and comments in it) to get 
familiar with the NVBit APIs.

In general all the NVBit tools should meet the following requirements:
1. Include nvbit.h which provide all the main NVBit APIs declarations
2. Link libnvbit.a which provides the core functions of NVBit
3. Not use shared memory
4. Be compiled as a dynamic shared library (so it can be loaded with 
LD_PRELOAD)
5. Use nvcc option  "-Xptxas -cloning=no" to prevent nvcc from eliminating 
device functions
6. Use only 16 registers to limit save/restore overhead (nvcc option 
"-maxrregcount=16")

A typical compilation line for an NVbit tool is the following:

```no-highlight
	nvcc -std=c++11 -I../../src -Xptxas -cloning=no -maxrregcount=16 -Xcompiler 
			-Wall -arch=sm_35 -O3 -Xcompiler -fPIC -shared  
			instr_count.cu -L../../src -lnvbit -lcuda -o instr_count.so
```
The use "-arch=sm_35" is not required, but is typically done so the same 
pre-compiled NVBit tool can be used across multiple GPU generations >= SM 3.5. 
If that is not a requirement then the NVBit tool can be compiled for a specific 
architecture.

