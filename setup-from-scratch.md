# Setup-From-Scratch Guide

<!--ts-->
* [Setup-From-Scratch Guide](#setup-from-scratch-guide)
   * [Hardware](#hardware)
   * [Add SSH keys](#add-ssh-keys)
   * [Install Packages](#install-packages)
   * [Build From Source](#build-from-source)
   * [Host Kernel](#host-kernel)
   * [Network](#network)
      * [Configure VF](#configure-vf)
      * [Configure IP](#configure-ip)
      * [Modify DPDK Scripts](#modify-dpdk-scripts)
   * [Evaluation Scripts](#evaluation-scripts)
   * [Note](#note)
<!--te-->

To reproduce the evaluations on your own machine, you can follow this guide to configure the machines. This guide is not a push-button style because some of the parameters such as path and PCI number differ from one machine to another. Please contact us if you encounter the unclear parts in this guide.

## Hardware

The Intel server should have at least 6 physical cores on a single NUMA node, with posted-interrupt support.

The AMD server should have at least 6 physical cores on a single NUMA node. Our evaluations are conducted on an AMD Zen 3 server, with x2APIC enabled in BIOS and AVIC is not supported. If you reproduce the evaluations on an AVIC-enabled server (e.g., Zen 4), the results might differ a lot from our paper.

## Add SSH keys

Add the SSH public key of the client machine to the `authorized_keys` file of the root user of the server machine.

Note: The first time you execute the ssh command, you should type "yes" on the shell.

## Install Packages

On both servers, execute the following command to install the required packages:

```bash
sudo apt-get update && sudo apt-get install \ 
            ca-certificates build-essential \ 
            curl git libssl-dev \ 
            pkg-config python3 wget \ 
            qemu-system-misc u-boot-qemu qemu-utils \
            flex bison bc \
            libelf-dev ninja-build \
            libglib2.0-dev libpixman-1-dev \
            libevent-dev numactl

sudo pip install qemu-affinity
```

## Build From Source

Refer to [build-from-source](./build-from-source.md). The [build and install host kernel](https://github.com/IPADS-Bifrost/ae-guide/blob/main/build-from-source.md#build-and-install-host-kernel) step should not be skipped.

## Host Kernel

You should add `isolcpus` parameter (like `isolcpus=80-90`) to the kernel command line to isolate the physical cores used for the guest VM and DPDK backend.

You should add hugepage configuration (like `hugepagesz=1GB hugepages=64 default_hugepagesz=1GB`) to the kernel command line to reserve hugepages for VM and DPDK.

Then, reboot the host machine with our kernel.

After booting the host machine, please install the `tls` kernel module:

```bash
sudo modprobe tls
```

## Network

In this section, we describe the procedure of configuring the network on our server. We assign a VF to the OvS-DPDK backend, which allows the native host OS to utilize the PF to provide existing services.

### Configure VF

We use Mellanox 200Gbps NIC on our server.

```bash
# In shell
> lspci | grep -i eth
21:00.0 Ethernet controller: Mellanox Technologies MT28908 Family [ConnectX-6]
# Skip other NICs
```

Add one VF and configure `trust` and `macaddress` this VF. The NIC name in our case is **enp33s0np0** and the MAC address is randomly generated. One should replace them with a proper NIC name and a valid MAC address in her case.

```bash
echo 0 | sudo tee /sys/class/net/enp33s0np0/device/sriov_numvfs
echo 1 | sudo tee /sys/class/net/enp33s0np0/device/sriov_numvfs
sudo ip link set enp33s0np0 vf 0 trust on
# Choose a MAC address that has no conflict with the existing environment
sudo ip link set enp33s0np0 vf 0 mac 66:11:22:33:44:78
# Replace enp33s0v0 with the VF name
sudo ip link set enp33s0v0 up
```

Get the PCI number of VF 0:

```bash
# In shell
> lspci | grep -i mell
21:00.0 Ethernet controller: Mellanox Technologies MT28908 Family [ConnectX-6] 
21:00.1 Ethernet controller: Mellanox Technologies MT28908 Family [ConnectX-6 Virtual Function] 
```

The PCI number of VF 0 is `21:00.1`. It would be used in a later step.

### Configure IP

The IP address of the Mellanox NIC of AMD server is `10.1.1.201`. (`10.1.1.xxx` is ok, if not conflict)

The IP address of the Mellanox NIC of Intel server is `10.1.1.200`. (`10.1.1.xxx` is ok, if not conflict)

The IP address of the VM is configured to be `10.1.1.222`.

### Modify DPDK Scripts

You should modify `$AE_HOME/server-script/scripts/start.sh` on the server machine. Take the script on AMD server as an example.

The following lines bind the VF to DPDK by PCI B.D.F `21:00.1`:

```bash
# Replace 21:00.1 with a proper B.D.F
ovs-vsctl add-port ovsbr1 dpdk0 -- \ 
        set Interface dpdk0 type=dpdk options:dpdk-devargs=0000:21:00.1
```

The following lines select core number of DPDK threads (in our example, 85 and 86):

```bash
PMD_CPU=85
PMD_CPU_MASK=$( printf "0x%X%016X\n" $((3 << $(($PMD_CPU - 64)))) "0")
```

## Evaluation Scripts

You should modify the following line of `$AE_HOME/client-script/env.sh` on the client machine.

```bash
BENCH_SCRIPT_PATH=/mnt/ssd/ae/author/server-script
# Modify to the $AE_HOME/server-script (replace AE_HOME here) on the server
```

You should modify the following lines of `$AE_HOME/server-script/scripts/pin-vm.sh` on the server machine.

```bash
PLUS=1
START_VCPU=80

# If the START_VCPU is 80 and PLUS is 1, then a 4-vCPU VM would be bound to core 80, 81, 82, 83
# If the START_VCPU is 10 and PLUS is 2, then a 4-vCPU VM would be bound to core 10, 12, 14, 16
```

You should modify the following lines of `$AE_HOME/server-script/test.sh` on the server machine.

```bash
NUMA_ID=1 # Change to the NUMA id on your machine

taskset -c 80-87 $path_to_qemu \ # Change 80-87 to your isolated cores
```

## Note

During the experiments, the modified DPDK and OvS in our artifacts would be installed to the system path.
If you have your own DPDK and OvS installed before the evaluation, you should re-install your own versions to the system path.
