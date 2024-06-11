# POW for Bitbi
This pow is a fork of RandomX, a proof-of-work (PoW) algorithm that is optimized for general-purpose CPUs. RandomX uses random code execution (hence the name) together with several memory-hard techniques to minimize the efficiency advantage of specialized hardware.


## Overview

This POW utilizes a virtual machine that executes programs in a special instruction set that consists of integer math, floating point math and branches. These programs can be translated into the CPU's native machine code on the fly (example: [program.asm](doc/program.asm)). At the end, the outputs of the executed programs are consolidated into a 256-bit result using a cryptographic hashing function ([Blake2b](https://blake2.net/)).

Pow can operate in two main modes with different memory requirements:

* **Fast mode** - requires 2080 MiB of shared memory.
* **Light mode** - requires only 256 MiB of shared memory, but runs significantly slower

Both modes are interchangeable as they give the same results. The fast mode is suitable for "mining", while the light mode is expected to be used only for proof verification.

## Documentation

Full specification is available in [specs.md](doc/specs.md).

Design description and analysis is available in [design.md](doc/design.md).


## Build

RandomX is written in C++11 and builds a static library with a C API provided by header file [randomx.h](src/randomx.h). Minimal API usage example is provided in [api-example1.c](src/tests/api-example1.c). The reference code includes a `randomx-benchmark` and `randomx-tests` executables for testing.

### Linux

Build dependencies: `cmake` (minimum 3.5) and `gcc` (minimum version 4.8, but version 7+ is recommended).

To build optimized binaries for your machine, run:
```
git clone https://github.com/bitbi-core/pow.git
cd pow
mkdir build && cd build
cmake -DARCH=native ..
make
```

To build portable binaries, omit the `ARCH` option when executing cmake.

### Windows

On Windows, it is possible to build using MinGW (same procedure as on Linux) or using Visual Studio (solution file is provided).

### Precompiled binaries

Precompiled `randomx-benchmark` binaries are available on the [Releases page](https://github.com/tevador/RandomX/releases).

## Proof of work

RandomX was primarily designed as a PoW algorithm for [Monero](https://www.getmonero.org/). The recommended usage is following:

* The key `K` is selected to be the hash of a block in the blockchain - this block is called the 'key block'. For optimal mining and verification performance, the key should change every 2048 blocks (~2.8 days) and there should be a delay of 64 blocks (~2 hours) between the key block and the change of the key `K`. This can be achieved by changing the key when `blockHeight % 2048 == 64` and selecting key block such that `keyBlockHeight % 2048 == 0`.
* The input `H` is the standard hashing blob with a selected nonce value.

RandomX was successfully activated on the Monero network on the 30th November 2019.

If you wish to use RandomX as a PoW algorithm for your cryptocurrency, please follow the [configuration guidelines](doc/configuration.md).

**Note**: To achieve ASIC resistance, the key `K` must change and must not be miner-selectable. We recommend to use blockchain data as the key in a similar way to the Monero example above. If blockchain data cannot be used for some reason, use a predefined sequence of keys.

### CPU performance
The table below lists the performance of selected CPUs using the optimal number of threads (T) and large pages (if possible), in hashes per second (H/s). "CNv4" refers to the CryptoNight variant 4 (CN/R) hashrate measured using [XMRig](https://github.com/xmrig/xmrig) v2.14.1. "Fast mode" and "Light mode" are the two modes of RandomX.

|CPU|RAM|OS|AES|CNv4|Fast mode|Light mode|
|---|---|--|---|-----|------|--------------|
Intel Core i9-9900K|32G DDR4-3200|Windows 10|hw|660 (8T)|5770 (8T)|1160 (16T)|
AMD Ryzen 7 1700|16G DDR4-2666|Ubuntu 16.04|hw|520 (8T)|4100 (8T)|620 (16T)|
Intel Core i7-8550U|16G DDR4-2400|Windows 10|hw|200 (4T)|1700  (4T)|350 (8T)|
Intel Core i3-3220|4G DDR3-1333|Ubuntu 16.04|soft|42 (4T)|510 (4T)|150 (4T)|
Raspberry Pi 3|1G LPDDR2|Ubuntu 16.04|soft|3.5 (4T)|-|20 (4T)|

Note that RandomX currently includes a JIT compiler for x86-64, ARM64 and RISCV64. Other architectures have to use the portable interpreter, which is much slower.

### GPU performance

SChernykh is developing GPU mining code for RandomX. Benchmarks are included in the following repositories:

* [CUDA miner](https://github.com/SChernykh/RandomX_CUDA) - NVIDIA GPUs.
* [OpenCL miner](https://github.com/SChernykh/RandomX_OpenCL) - only for AMD Vega and AMD Polaris GPUs (uses GCN machine code).

The code from the above repositories is included in the open source miner [XMRig](https://github.com/xmrig/xmrig).

Note that GPUs are at a disadvantage when running RandomX since the algorithm was designed to be efficient on CPUs.
