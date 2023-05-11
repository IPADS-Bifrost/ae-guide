# Build-From-Source Guide

<!--ts-->
* [Build-From-Source Guide](#build-from-source-guide)
   * [Clone Repositories](#clone-repositories)
   * [Build and Install Host Kernel](#build-and-install-host-kernel)
   * [Build Guest Kernel](#build-guest-kernel)
   * [Build QEMU](#build-qemu)
   * [Build TLS Microbenchmark](#build-tls-microbenchmark)
   * [Download and Configure Guest Disk Image](#download-and-configure-guest-disk-image)
   * [Build Memtier_benchmark](#build-memtier_benchmark)
   * [Build wrk](#build-wrk)
   * [OVS and DPDK](#ovs-and-dpdk)
<!--te-->

We provide this guide for those who would like to build from source. This guide describes how to build all things required for this AE.

## Clone Repositories

Create an empty directory named `reviewer_x`:

```bash
# please replace reviewer_x with your reviewer ID (e.g., reviewer_1)
mkdir reviewer_x
cd reviewer_x
export AE_BASE=$(pwd)
```

Then, execute the following command on both servers:

```bash
git clone https://github.com/IPADS-Bifrost/cvm-opt-guest
wget https://download.qemu.org/qemu-6.2.0.tar.xz
git clone https://github.com/OP-TEE/build soc_term
git clone https://github.com/IPADS-Bifrost/memtier_benchmark
git clone https://github.com/IPADS-Bifrost/ktls_test
git clone https://github.com/IPADS-Bifrost/cvm-opt-ovs
git clone https://github.com/IPADS-Bifrost/cvm-opt-dpdk
```

The following repositories are only required on Intel server:

```bash
git clone https://github.com/IPADS-Bifrost/ae-script -b server-on-intel server-script
git clone https://github.com/IPADS-Bifrost/ae-script -b client-on-intel client-script
git clone https://github.com/IPADS-Bifrost/host-intel
```

The following repositories are only required on the AMD server:

```bash
git clone https://github.com/IPADS-Bifrost/ae-script -b server-on-amd server-script
git clone https://github.com/IPADS-Bifrost/ae-script -b client-on-amd client-script
git clone https://github.com/IPADS-Bifrost/host-sev-snp
```

## Build and Install Host Kernel

> Note: We do not recommend reinstalling the host kernel, because our modification on the host kernel only involves code for breakdown (on both server) and CVM emulation (on Intel server). We do not implement any optimization on the host kernel.

Executing the on following command on Intel Server to build and install host kernel:

```bash
cd $AE_BASE/host-intel
cp ../server-script/config/host-config .config
# Optional: Change the local name of the kernel to identify that the kernel is built by you
# make menuconfig
# -> General Setup -> Local version
make -j$(nproc) && \
    sudo make INSTALL_MOD_STRIP=1 modules_install -j$(nproc) && \
    sudo make install
```

Executing the on following command on AMD Server to build and install host kernel:

```bash
cd $AE_BASE/host-sev-snp
cp ../server-script/config/host-config .config
# Optional: Change the local name of the kernel to identify that the kernel is built by you
# make menuconfig
# -> General Setup -> Local version
make -j$(nproc) && \
    sudo make INSTALL_MOD_STRIP=1 modules_install -j$(nproc) && \
    sudo make install
```

Installing host kernel requires `sudo`. To install the host kernel on our test machines, please contact the authors.

## Build Guest Kernel

On both server, execute the following command to build guest kernels:

```bash
cd $AE_BASE/server-script
./build-guest.sh
```

> Note: This step involves building 9 Linux kernels, which would estimatedly last about 20 minutes.

The guest kernels and kernel modules for breakdown are saved at `$AE_BASE/server-script/assets` and `$AE_BASE/server-script/assets/breakdown`.

## Build QEMU

```bash
cd $AE_BASE
tar xf qemu-6.2.0.tar.xz
cd qemu-6.2.0
mkdir build
cd build
../configure --target-list=x86_64-softmmu
make -j$(nproc)
```

## Build TLS Microbenchmark

On both server, execute the following command:

```bash
cd $AE_BASE/ktls_test
./build.sh
```

## Download and Configure Guest Disk Image

Go to [our uploaded images on Zenodo](https://doi.org/10.5281/zenodo.7920234).
On both server, execute the following command to download the virtual disk image for guest VM.

```bash
cd $AE_BASE/server-script/assets
# Copy the download url
wget <amd-focal.img / intel-focal.img>
mv <amd-focal.img / intel-focal.img> focal.img
```

Copy the kernel modules to `/root` of the disk image to the disk, and add ssh public keys of client to the disk:

```bash
./mount.sh
sudo cp breakdown/* ./mnt/root/
sudo cp $AE_BASE/ktls_test/tls_client_13 ./mnt/root
sudo cp $AE_BASE/ktls_test/tls_server ./mnt/root
sudo cp -r $AE_BASE/ktls_test/certs ./mnt/root

# Add ssh public keys of client
# cat /path/to/client.pub | sudo tee -a ./mnt/root/.ssh/authorized_keys
./umount.sh
```

This step requires `sudo`. To finish this step on our test machines, please contact the authors.

## Build Memtier_benchmark

We use memtier_benchmark as clients for 

On the AMD server, execute the following command:

```bash
cd $AE_BASE/memtier_benchmark
autoreconf -ivf
./configure
make -j$(nproc)
```

Then, use `scp` to copy it to the Intel server:

```
# On the Intel server
cd $AE_BASE/memtier_benchmark
scp ae@amd-server:~/reviewer_x/memtier_benckmark/memtier_benchmark .
```
## Build wrk

We use wrk-4.2.0 to build the wrk for nginx benchmark.
You can download from the official website, and add the installation path to `PATH`.

## OVS and DPDK

OVS and DPDK is compiled during the tests, so building them is not required here.

