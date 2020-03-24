NVBit is released as an artifact via github, it can be downloaded at: https://github.com/NVlabs/NVBit/releases

A paper describing NVBit was published at MICRO 2019 and it can be found at: https://github.com/NVlabs/NVBit/releases/download/v1.0/MICRO_19_NVBit.pdf



# NVBit (NVidia Binary Instrumentation Tool)
NVIDIA Corporation

NVBit is covered by the same End User License Agreement as that of the
NVIDIA CUDA Toolkit. By using NVBit you agree to End User License Agreement
described in the EULA.txt file.

NVBit is not part of the official CUDA toolkit, but instead is a research prototype from the Architecture Research Group at NVIDIA and as such is provided as-is with no guarantee of support.

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
used (i.e. nvcc, pgicc, etc).

## Requirements

* SM compute capability:              >= 3.5 && <= 7.2
* Host CPU:                           x86_64, ppc64le
* OS:                                 Linux
* GCC version:                        >= 5.3.0
* CUDA version:                       >= 8.0 && <= 10.2
* CUDA driver version:                <= 440.64

Currently no Embedded GPUs or ARM hosts are supported.
