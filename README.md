# BladeBit Chia Plotter

A fast **RAM-only**, **k32-only**, Chia plotter.

## Requirements
**416 GiB of RAM are required** to run it, plus a few more megabytes for stack space and small allocations. 

## Building

### Prerequisites
This repository depends on Chia's [bls-signatures](https://github.com/Chia-Network/bls-signatures) repository to generate keys and plot ids, which requires [Cmake](https://cmake.org/). At the time of this writing **Cmake 3.14+** is required by bls-signatures. See the repository for any updated build instructions.

64-bit is supported only, for obvious reasons.
Only Linux is currently completed. There are several platform abstractions misisng for macOS and Windows.

### Build

```bash
# Clone the repo & its submodules
git clone --recursive https://github.com/harold-b/bladebit.git

# Build bls library. Only needs to be done once.
./build-bls

# For x86
make clean && make -j$(nproc --all)

# For ARM
make clean && make -j$(nproc --all) CONFIG=release.arm
```

## Usage
Run `bladebit -h` for usage and command line options.


## Support
You can support me by donating to:
- XCH: **xch1xd9vlw5t3ul0chqlg3pqffy5ecq8tude8sc4v6q5r66raavczuqsr0mkwc**
- BTC: **bc1qy3uz29ns7jlaql36ml0ajhus4r24v8jfcja69e**
- ETH: **0x70978b4236aeBDd79c6ea8602656b53acC03f591**


## License
Licensed under AGPLv3 See [LICENSE](LICENSE).
For special licensing for commercial use, please contact me.


## Build Tools
This project synchronizes Makefiles, VS project files and VS Code c_cpp_properties.json files by a custom tool called psync which is not currently published. I created the tool a while back for cross-platform game engine development and it is scrappy-ly coded and unclean. At some point I will publish it to facilitate development.


# Other Details

## Disk I/O
Writes to disk only occur to the final plot file, and it is done sequentially, un-buffered, with direct I/O. This means that writes will be block-aligned. If you've gotten faster writes elsewhere in your drive than you will get with this, it is likely that it is using buffered writes, therefore it "finishes" before it actually finishes writing to disk. The kernel will handle the I/O in the background from cache (you can confirm this with tools such as iotop). The final writes here ought to pretty much saturate your sequential writes. Writes begin happening in the background at Phase 3 and will continue to do so, depending on the disk I/O throughput, through the next plot, if it did not finish beforehand. At some point in Phase 1 of the next plot, it might stall if it still has not finished writing to disk and a buffer it requires is still being written to disk. On the system I tested, there was no interruption when using an NVMe drive.


## Pool Plots
Pool plots are implemented, but I have not had the time to test them. The code follows the chia-blockchain reference to create taproot memos. So it should work, but again, since they are untested, I can't guarantee anything. 


## NUMA systems
There are certain instances in plotting where memory touched is unpredictable (for example, sorting). These parts can become slow already since they will likely have tons of cache misses because of the random nature of memory access patterns. If you have NUMA system, this part will become even more inefficient because, even if memory affinity is set, at some point memory from another core will have to be touched.

Memory policies can be set, which will certainly help with sequential access portions. However, since I don't have a NUMA system to test in, this has not been implemented at all.


## Huge TLBs
This is not supported yet. Some folks have reported some gains when using huge page sizes. Although this was something I wanted to test, I focused first instead on things that did not necessarily depended on system config. But I'd like to add support for it in the future (trivial from the development point of view, I have just not configured the test system with huge page sizes).

## Other Optimizations
Memory accesses are pretty tight. So some spots are saturated, and completely memory bound. That is, the CPU is already completely saturated, and won't be able to do more unless access to memory is faster. However, there are more pending optimizations, some ARM-specific, some x86-specific, and other experiments for either architecture that I'd like to implement/test. I hope to have the time to do this. But your supports will surely help.


Copyright 2021 Harold Brenes

