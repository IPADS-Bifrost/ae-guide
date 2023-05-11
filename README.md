# AE Guide for Bifrost (ATC'23)

<!--ts-->
* [AE Guide for Bifrost (ATC'23)](#ae-guide-for-bifrost-atc23)
   * [Abstract](#abstract)
   * [Scope](#scope)
   * [Contents](#contents)
   * [Evaluation Steps](#evaluation-steps)
      * [Step-0: Prepare on Client Machines](#step-0-prepare-on-client-machines)
         * [Copy from Local](#copy-from-local)
         * [Download from GitHub and Zenodo](#download-from-github-and-zenodo)
      * [Step-1: Setup Tests](#step-1-setup-tests)
      * [Step-2: Automated Testing (Estimated 12 Hours on Each Server)](#step-2-automated-testing-estimated-12-hours-on-each-server)
      * [Step-3: Generate Figures](#step-3-generate-figures)
<!--te-->

## Abstract

Bifrost (our proposed system) measures and analyzes the network performance of traditional baseline VMs, vanilla confidential VMs (CVMs) and Bifrost-optimized CVMs under network-intensive scenarios.

The artifacts consists of three main parts:
1. The breakdown of CVM-IO tax using network-intensive applications on traditional baseline VMs, vanilla confidential VMs (CVMs).
2. The performance comparison of a TCP-based TLS microbenchmark on traditional baseline VMs, vanilla confidential VMs (CVMs) and Bifrost-optimized CVMs.
3. The performance comparison of network-intensive application benchmarks on traditional baseline VMs, vanilla confidential VMs (CVMs) and Bifrost-optimized CVMs. 

The testbed includes an AMD SEV-ES/SNP server (for experiments of SEV CVMs) and an Intel server (for experiments of simulated TDX CVMs).  Both machines are equipped with one single-port Mellanox Connect-X6 200Gbps NIC and are back-to-back connected with a fabric cable.

## Scope

The artifacts of Bifrost should reproduce Figure 2-3 and Figure 6-12 of our paper.
These artifacts should present the breakdown results of the IO tax for network-intensive workloads in CVMs and show Bifrost's large performance optimizations compared with traditional baseline VMs and vanilla CVMs.

## Contents

- **Hardware:**
  - The AMD server has two 64-core AMD EPYC 7T83 CPUs at 2.45GHz (128 cores in total) and 500GB DDR4 DRAM.
  - The Intel server has two 12-core Intel Xeon Gold 5317 CPU at 3.00GHz (24 cores in total) and 188GB DDR4 DRAM.
  - Both machines are equipped with one single-port Mellanox Connect-X6 200Gbps NIC and are back-to-back connected with a fabric cable.
- **Software:**
  - Host kernel: the AMD server uses Linux v5.19.0-rc6 with SEV-ES/SNP support, the Intel server uses Linux v5.4.0 (default in Ubuntu 20.04).
  - Network backend: OpenvSwitch v2.17.3 and DPDK v21.11.2.
  - Guest kernel: Linux v6.0-rc1.
  - Guest Virtual Storage:
    - Base image: [Ubuntu server 20.04 cloud image](https://cloud-images.ubuntu.com/focal/current/)
    - Dependencies: OpenSSL with kTLS enabled (e.g., v3.1.0) as well as applications with TLS enabled.
- **Metrics:**
  - Breakdown results: percentage of consumed CPU cycles.
  - Benchmark results: normalized overhead of throughput compared with baseline VMs and vanilla CVMs.
- **Estimated time:** Two 12-hour evaluations (24 hours in total).
- **Available:**
  - AE Guidance: https://github.com/IPADS-Bifrost/ae-guide/
  - Guest Virtual Storage: https://doi.org/10.5281/zenodo.7920234

## Evaluation Steps

The following content guides the reproduction of Figure 2-3 and Figure 6-12 in the paper.

To evaluate the artifacts in a **push-button** style, please send us your public keys via HotCRP to connect to an environment pre-configured by us.
For simplicity, we provide all pre-built images. If you want to check their building from source, please refer to [build-from-source.md](./build-from-source.md).

> Note: Please be sure to hide your personal information at the end of the public key to keep anonymous!

Our evaluation testbed includes both an Intel server and an AMD server with SEV-ES/SNP support.

- The server side, which runs various applications in VMs/CVMs to handle request.
- The client side, which runs benchmark and send request to server machine. 

When benchmarking server-side applications in a CVM atop the AMD machine, the Intel machine is used as the client side, and vice versa.

```
Server(AMD) <====> Client(Intel)
Server(Intel) <====> Client(AMD)
```

If you are equipped with a similar testbed already, you can reproduce results on your own machines.
If our AE scripts are not compatible with your local environment, please refer to [OvS-DPDK Installation](https://docs.openvswitch.org/en/latest/intro/install/dpdk/) and [Linux kernel build](https://kernelnewbies.org/KernelBuild) for building and installing necessary components.

The following is the quick start of the AE. We recommend reviewers to run following steps within a `tmux` session to keep scripts alive.

### Step-0: Prepare on Client Machines

First, connect to client machines via SSH to prepare the environment.
For security reason, we will only send the SSH commands to the reviewers through HotCRP.

Next, acquire the source trees and images that are necessary for the evaluation.

Due to the low-speed connectivity to the GitHub, we suggest reviewers to [copy them from a local directory](#copy-from-local) to accelerate the AE procedure.
Reviewers can also insist on [downloading the source code and images from GitHub and Zenodo](#download-from-github-and-zenodo).

#### Copy from Local

On the AMD server:

```bash
# please replace reviewer_x with your reviewer ID (e.g., reviewer_1)
tmux new -s reviewer_x
export AE_HOME=/home/ae
cd $AE_HOME
cp -r ./author/ ./reviewer_x/
```

On the Intel server:

```bash
# please replace reviewer_x with your reviewer ID (e.g., reviewer_1)
tmux new -s reviewer_x
export AE_HOME=/mnt/ssd/ae
cd $AE_HOME
cp -r ./author/ ./reviewer_x/
```

Please jump to Step-1 if you have finished above copies locally.

#### Download from GitHub and Zenodo

Please refer to [build-from-source.md](./build-from-source.md)

### Step-1: Setup Tests

On the AMD server, enter your own directory.
We have already configured the huge pages `hugepagesz=1GB hugepages=64 default_hugepagesz=1GB` and isolated CPU cores with `isolcpus=80-90` in the `/etc/default/grub`.

```bash
# please replace reviewer_x with your reviewer ID (e.g., reviewer_1)
export BENCH_CLIENT_PATH=$AE_HOME/reviewer_x/client-script
```

On Intel server, enter your own directory.
We have already configured the huge pages `hugepagesz=1GB hugepages=64 default_hugepagesz=1GB` and isolated CPU cores with `isolcpus=6-16,30-40` in the `/etc/default/grub`.

```bash
# please replace reviewer_x with your reviewer ID (e.g., reviewer_1)
export BENCH_CLIENT_PATH=$AE_HOME/reviewer_x/client-script
```

### Step-2: Automated Testing (Estimated 12 Hours on Each Server)

First, take the AMD server as the client machine, and start the evaluation on the AMD server:

```
cd $BENCH_CLIENT_PATH
./run.sh
```

Second, **after** the evaluation on the AMD server finishes, take the Intel server as the client machine, and start the evaluation on the Intel server:

```
cd $BENCH_CLIENT_PATH
./run.sh
```

> Note: We will run a shared `soc_term` in a shared tmux session named `soc-term` to receive the console output from VMs and CVMs.
> If you want to see the console output, please attach to this tmux session.

### Step-3: Generate Figures

Clone the scripts for drawing figures on the Intel machine:

```bash
git clone https://github.com/IPADS-Bifrost/ae-script.git -b draw-figures draw-figures
cd ./draw-figures
```

Copy the evaluation raw results to the Intel machine. This script receives one parameter as the directory name of yours (e.g., `reviewer_1`). If no parameter is given, this script would copy data from `author`.

```bash
./copy.sh reviewer_1
# Executing `./copy.sh` is the same as executing `./copy.sh author`
```

After that, you can draw the figures.

```bash
./figure.sh
```

The output figures are `.eps` files in `./figures/` directory.

```bash
ls ./figures

# The output should be like:
# fig-02-a.eps  fig-03.eps    fig-07-a.eps  fig-08-a.eps  fig-09.eps    fig-11-b.eps  fig-12-b.eps
# fig-02-b.eps  fig-06-a.eps  fig-07-b.eps  fig-08-b.eps  fig-10.eps    fig-11-c.eps  fig-12-c.eps
# fig-02-c.eps  fig-06-b.eps  fig-07-c.eps  fig-08-c.eps  fig-11-a.eps  fig-12-a.eps
```

You can compress the `./figures/` directory in to a tarball and copy it to your local machine.

```bash
tar czf figures.tar.gz ./figures/
# then scp to your local machine and check the figures
```
