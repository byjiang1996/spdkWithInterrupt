# spdkWithInterrupt
UIUC CS523 by Binyao Jiang -- Interrupt Support and Coalescing Enhancement for SPDK When Online Applications Co-running with Offline Applications

## Basics

Report can be found at [here](report.pdf). Poster can be found at [here](poster.pdf). Gem5 files including disk image can be found at [here](https://uofi.box.com/s/lxew4sm5a4l6niz7qr253pugt0tlugfp). Customized v4.14.146 linux kernel can be found [here](https://uofi.box.com/s/9x5go63mmifjy66cm5o3zddu4cwx8yx1).

Add `export M5_PATH=<m5_path>` into your environmental variable, and run:

```
git submodule update --init --recursive
```

*Note: if you want to check how the whole system is changed to support interrupt and interrupt coalescing, please check the details mentioned in the report and commits in each git submodule.*

## Installation

### Download ARM Cross Compiler Toolchain

Download linaro gcc aarch64 linux toolchain from [here](https://releases.linaro.org/components/toolchain/binaries/7.4-2019.02/aarch64-linux-gnu/gcc-linaro-7.4.1-2019.02-x86_64_aarch64-linux-gnu.tar.xz) and decompress it. 

Add `export PATH=$PATH:<gcc_linaro_path>/bin` into your environmental variable.

### Build SimpleSSD FullSystem

Download our customized SimpleSSD repo.

```
cd <SimpleSSD_path>
apt install build-essential
apt install scons python-dev zlib1g-dev m4 cmake
apt install libprotobuf-dev protobuf-compiler
apt install libgoogle-perftools-dev
apt install device-tree-compiler
scons build/ARM/gem5.fast -j4
```

### Build Benchmark Tools in Linux Disk Image

This step is `optional` if you use the provided linux disk image.

First, you need to mount the linux disk image to your system folder. 

```
<SimpleSSD_path>/util/gem5img.py mount <m5_path>/disks/ubuntu-16.04-arm.img /mnt
```

And then, you need to change the root by copying `qemu-aarch64-static` executable downloaded [here](https://github.com/multiarch/qemu-user-static/releases/download/v4.1.1-1/qemu-aarch64-static) into linux disk image:

```
mv qemu-aarch64-static /mnt/usr/bin
```

Then, we can change the root into linux disk image, and start building your benchmark tools including FIO, spdk, rocksdb:

```
cd /mnt
chroot .
cd root
# Build FIO according to the instruction in https://github.com/axboe/fio
# Build our customized spdk according to the instructions in https://github.com/byjiang1996/spdk/tree/master/examples/nvme/fio_plugin
# Build rocksdb according to the instructions in https://spdk.io/doc/blobfs.html
```

Finally, run the following instuction to umount the linux disk image.

```
<SimpleSSD_path>/util/gem5img.py umount /mnt
```

### Build Customized v4.14.146 Linux Kernel

You can use our provided linux kernel image and customized uio driver. Or you can refer to online documents to compile and build your linux kernel. After downloading the kernel code, please remember to replace the uio folder with our provided customized uio. 

Please remember to put downloaded or generated `byuio.ko` and `byuio_pci_generic.ko` to your linux disk image path: `/root/xxx.ko` (use gem5img.py to mount this disk image mentioned above).

## Run Benchmarks

Start gem5 simulator: `bash <SimpleSSD_path>/arm_test.sh`. When running, the stdout will output the listenned system terminal port, e.g. 3456. *Note that: vmlinux in the shell means the linux kernel image, plase place it to a correct location*.

Run `<SimpleSSD_path>/util/term/m5term 127.0.0.1 <listenned_terminal_port>`. And then you can operate on your gem5 simulator just like operating on a normal linux system. *Note that: to transfer file between simulator and host, please put your file to `<SimpleSSD_path>/m5file`, and in simulator, run `m5 readfile`*.

Run FIO experiment: refer to `https://github.com/byjiang1996/spdk/tree/master/examples/nvme/fio_plugin/gem5_config`

Run RocksDB experiment: `bash <spdk_path>/test/blobfs/rocksdb/rocksdb.sh`