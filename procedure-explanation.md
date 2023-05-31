# Explanation of the Test Procedure

<!--ts-->
* [Explanation of the Test Procedure](#explanation-of-the-test-procedure)
   * [Booting Guest VM](#booting-guest-vm)
   * [Tests of Microbenchmark and Application](#tests-of-microbenchmark-and-application)
   * [Tests of Breakdown](#tests-of-breakdown)
<!--te-->

This guide explains the test procedure of this AE, including tests scripts and the test output.

In [Automated Testing](https://github.com/IPADS-Bifrost/ae-guide#step-2-automated-testing-estimated-12-hours-on-each-server) part of AE, `run.sh` is executed. `run.sh` would execute `bench.sh` and `breakdown.sh`. Both of them use `ssh` to boot the guest VM on the server machine and then start testing.

 `bench.sh` runs tests of microbenchmark and application reproducing figure 2, 6, 7, 8, 11 and 12, while `breakdown.sh` runs breakdown reproducing figure 3, 9 and 10.

## Booting Guest VM

The entry of `bench.sh` and `breakdown.sh` is both `main()`. It would call `test` with multiple parameter pairs. The parameter pair is like

```
test <mode> <swiotlb_opt> <hit_rate> <eval>
```

The first and second parameter `mode` and `swiotlb_opt` select the testbed of guest VM:

* `mode` selects the guest kernel.
* `swiotlb_opt` decides whether to use run in confidential VM mode. 0 means non-CVM and non-zero value means running in CVM.

Tests of microbenchmark and application of `CVM`:

| Figure Label | Client Script Parameter (mode, swiotlb_opt) | How the Guest Kernel is built                                |
| ------------ | ------------------------------------------- | ------------------------------------------------------------ |
| Baseline     | vanilla, 0                                  | [Building Baseline](https://github.com/IPADS-Bifrost/ae-script/blob/server-on-amd/build-guest.sh#L15) |
| CVM          | vanilla, 2                                  | [Building CVM](https://github.com/IPADS-Bifrost/ae-script/blob/server-on-amd/build-guest.sh#L15) |

Tests of microbenchmark and application of `CVM+RIF`:

| Figure Label           | Client Script Parameter (mode, swiotlb_opt) | How the Guest Kernel is built                                |
| ---------------------- | ------------------------------------------ | ------------------------------------------------------------ |
| Baseline               | vo, 0                                      | [Building Baseline](https://github.com/IPADS-Bifrost/ae-script/blob/server-on-amd/build-guest.sh#L15) |
| CVM+RIF                | vo, 2                                      | [Building CVM+RIF](https://github.com/IPADS-Bifrost/ae-script/blob/server-on-amd/build-guest.sh#L15) |
| +ZC                    | no, 2                                      | [Building +ZC](https://github.com/IPADS-Bifrost/ae-script/blob/server-on-amd/build-guest.sh#L20) |
| +PRPR                  | vgo, 2                                     | [Building +PRPR](https://github.com/IPADS-Bifrost/ae-script/blob/server-on-amd/build-guest.sh#L25) |
| Bifrost                | ngo, 2                                     | [Building Bifrost](https://github.com/IPADS-Bifrost/ae-script/blob/server-on-amd/build-guest.sh#L30) |
| w/o TOCTTOU protection | ngonp, 2                                   | [Building w/o TOCTTOU protection](https://github.com/IPADS-Bifrost/ae-script/blob/server-on-amd/build-guest.sh#L35) |

Tests of microbenchmark and application of `CVM+PI`:

| Figure Label           | Client Script Parameter (mode, swiotlb_opt) | How the Guest Kernel is built                                |
| ---------------------- | ------------------------------------------ | ------------------------------------------------------------ |
| Baseline               | vanilla, 0                                 | [Building Baseline](https://github.com/IPADS-Bifrost/ae-script/blob/server-on-intel/build-guest.sh#L15) |
| CVM+PI                 | vanilla, 1                                 | [Building CVM_PI](https://github.com/IPADS-Bifrost/ae-script/blob/server-on-intel/build-guest.sh#L15) |
| +ZC                    | numatx, 1                                  | [Building +ZC](https://github.com/IPADS-Bifrost/ae-script/blob/server-on-intel/build-guest.sh#L20) |
| +PRPR                  | vg, 1                                      | [Building +PRPR](https://github.com/IPADS-Bifrost/ae-script/blob/server-on-intel/build-guest.sh#L25) |
| Bifrost                | ng, 1                                      | [Building Bifrost](https://github.com/IPADS-Bifrost/ae-script/blob/server-on-intel/build-guest.sh#L30) |
| w/o TOCTTOU protection | ngnp, 1                                    | [Building w/o TOCTTOU protection](https://github.com/IPADS-Bifrost/ae-script/blob/server-on-intel/build-guest.sh#L35) |

Tests of breakdown of `CVM`:

| Figure Label | Client Script Parameter (mode, swiotlb_opt) | How the Guest Kernel is built                                |
| ------------ | ------------------------------------------- | ------------------------------------------------------------ |
| Baseline     | bd, 0                                       | [Building Baseline](https://github.com/IPADS-Bifrost/ae-script/blob/server-on-amd/build-guest.sh#L42) |
| CVM          | bd, 2                                       | [Building CVM](https://github.com/IPADS-Bifrost/ae-script/blob/server-on-amd/build-guest.sh#L42) |

Tests of breakdown of `CVM+RIF`:

| Figure Label | Client Script Parameter (mode, swiotlb_opt) | How the Guest Kernel is built                                |
| ------------ | ------------------------------------------ | ------------------------------------------------------------ |
| Baseline     | vobd, 0                                    | [Building Baseline](https://github.com/IPADS-Bifrost/ae-script/blob/server-on-amd/build-guest.sh#L42) |
| CVM+RIF      | vobd, 2                                    | [Building CVM+RIF](https://github.com/IPADS-Bifrost/ae-script/blob/server-on-amd/build-guest.sh#L42) |
| +ZC          | nobd, 2                                    | [Building +ZC](https://github.com/IPADS-Bifrost/ae-script/blob/server-on-amd/build-guest.sh#L51) |
| +PRPR        | vgobd, 2                                   | [Building +PRPR](https://github.com/IPADS-Bifrost/ae-script/blob/server-on-amd/build-guest.sh#L58) |
| Bifrost      | ngobd, 2                                   | [Building Bifrost](https://github.com/IPADS-Bifrost/ae-script/blob/server-on-amd/build-guest.sh#L65) |

Tests of breakdown of `CVM+PI`:

| Figure Label | Client Script Parameter (mode, swiotlb_opt) | How the Guest Kernel is built                                |
| ------------ | ------------------------------------------ | ------------------------------------------------------------ |
| Baseline     | bd, 0                                      | [Building Baseline](https://github.com/IPADS-Bifrost/ae-script/blob/server-on-intel/build-guest.sh#L42) |
| CVM+PI       | bd, 1                                      | [Building CVM_PI](https://github.com/IPADS-Bifrost/ae-script/blob/server-on-intel/build-guest.sh#L42) |
| +ZC          | nbd, 1                                     | [Building +ZC](https://github.com/IPADS-Bifrost/ae-script/blob/server-on-intel/build-guest.sh#L51) |
| +PRPR        | vgbd, 1                                    | [Building +PRPR](https://github.com/IPADS-Bifrost/ae-script/blob/server-on-intel/build-guest.sh#L58) |
| Bifrost      | ngbd, 1                                    | [Building Bifrost](https://github.com/IPADS-Bifrost/ae-script/blob/server-on-intel/build-guest.sh#L65) |

## Tests of Microbenchmark and Application

In `bench.sh`, tests for microbenchmark (TLS) and application (Memcached, Redis and Nginx) would be run 10 times for each guest kernel to be tested (tests for `CVM` would only run tests for application).

Each microbenchmark or application would be run for **10** times.

Reference output of TLS microbenchmark:

```log
Connecting to 10.1.1.222 port 2222...
Connecting to 10.1.1.222 port 3333...
Connecting to 10.1.1.222 port 4444...
Connecting to 10.1.1.222 port 5555...
Connected to server: TLSv1.3 TLS_AES_128_GCM_SHA256
Buffer size is 32768
Connected to server: TLSv1.3 TLS_AES_128_GCM_SHA256
Buffer size is 32768
Connected to client: TLSv1.3 TLS_AES_128_GCM_SHA256
Connected to server: TLSv1.3 TLS_AES_128_GCM_SHA256
Buffer size is 32768
Connected to client: TLSv1.3 TLS_AES_128_GCM_SHA256
Connected to server: TLSv1.3 TLS_AES_128_GCM_SHA256
Connected to client: TLSv1.3 TLS_AES_128_GCM_SHA256
Buffer size is 32768
Connected to client: TLSv1.3 TLS_AES_128_GCM_SHA256
In total sent 8589934592 and received 0 bytes
ERROR: read error 0
received 8589934616 Bytes in 24811163 us
throughput 330.173963 MB/s
In total sent 8589934592 and received 0 bytes
ERROR: read error 0
received 8589934616 Bytes in 24960063 us
throughput 328.204301 MB/s
In total sent 8589934592 and received 0 bytes
ERROR: read error 0
received 8589934616 Bytes in 25036358 us
throughput 327.204141 MB/s
In total sent 8589934592 and received 0 bytes
ERROR: read error 0
received 8589934616 Bytes in 25061380 us
throughput 326.877451 MB/s
```

Reference output of Memcached:

```log
Writing results to stdout
[RUN #1] Preparing benchmark client...
[RUN #1] Launching threads now...
[RUN #1 3%,   0 secs]  8 threads:       53172 ops,   57453 (avg:   57453) ops/sec, 1.31GB/sec (avg[RUN #1 6%,   1 secs]  8 threads:      118993 ops,   65795 (avg:   61786) ops/sec, 1.46GB/sec (avg[RUN #1 10%,   2 secs]
# Skip some of the log during testing

8         Threads
32        Connections per thread
30        Seconds


ALL STATS 
============================================================================================================================
Type         Ops/sec     Hits/sec   Misses/sec    Avg. Latency     p50 Latency     p99 Latency   p99.9 Latency       KB/sec
----------------------------------------------------------------------------------------------------------------------------
Sets         5817.81          ---          ---         4.51660         3.71100        14.01500        61.69500   1489603.69
Gets        58137.26      5845.21     52292.05         3.94853         3.21500        12.86300        22.01500      2037.46
Waits           0.00          ---          ---             ---             ---             ---             ---          ---
Totals      63955.07      5845.21     52292.05         4.00021         3.26300        12.99100        22.65500   1491641.15
```

The output of Redis is similar to Memcached. Both of them use `memtier_benchmark` as client.

Reference output of Nginx:

```
Running 30s test @ https://10.1.1.222/32kb.bin
  4 threads and 16 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     0.96ms  188.25us  19.33ms   90.85%
    Req/Sec     4.06k   150.80     5.47k    80.88%
  485451 requests in 30.10s, 14.93GB read
Requests/sec:  16128.06
Transfer/sec:    507.94MB
```



## Tests of Breakdown

In `breakdown.sh`, tests for Memcached 4vCPU-256KB would be run once for each guest kernel to be tested. The breakdown log would be collected and analyzed after testing.

Reference output of breakdown:

```
Writing results to stdout
[RUN #1] Preparing benchmark client...
[RUN #1] Launching threads now...
[RUN #1 3%,   0 secs]  8 threads:       53172 ops,   57453 (avg:   57453) ops/sec, 1.31GB/sec (avg[RUN #1 6%,   1 secs]  8 threads:      118993 ops,   65795 (avg:   61786) ops/sec, 1.46GB/sec (avg[RUN #1 10%,   2 secs]
# Skip some of the log during testing

[959708.354675] vcpu 0 reason 0x60 cnt 92023 cycle 853669630 percent 1, cycle1 1599055930, percent 3
[959708.364659] vcpu 0 reason 0x61 cnt 2 cycle 179225 percent 0, cycle1 195425, percent 0
[959708.373552] vcpu 0 reason 0x64 cnt 5863 cycle 20937214 percent 0, cycle1 68427514, percent 0
[959708.383130] vcpu 0 reason 0x72 cnt 128 cycle 818015 percent 0, cycle1 1854815, percent 0
[959708.392318] vcpu 0 reason 0x78 cnt 193 cycle 28829149 percent 0, cycle1 30392449, percent 0
[959708.401805] vcpu 0 reason 0x7b cnt 1 cycle 101262 percent 0, cycle1 109362, percent 0
[959708.410708] vcpu 0 reason 0x7c cnt 22041 cycle 205637369 percent 0, cycle1 384169469, percent 0
[959708.420584] vcpu 0 reason 0x3ff cnt 29 cycle 7914193 percent 0, cycle1 8149093, percent 0
[959708.429879] vcpu 1 reason 0x60 cnt 833193 cycle 4606277673 percent 9, cycle1 11355140973, percent 23
[959708.440239] vcpu 1 reason 0x61 cnt 2 cycle 182753 percent 0, cycle1 198953, percent 0
[959708.449138] vcpu 1 reason 0x64 cnt 71639 cycle 232286686 percent 0, cycle1 812562586, percent 1
[959708.459010] vcpu 1 reason 0x72 cnt 110 cycle 774289 percent 0, cycle1 1665289, percent 0
[959708.468201] vcpu 1 reason 0x78 cnt 5 cycle 29957 percent 0, cycle1 70457, percent 0
[959708.476905] vcpu 1 reason 0x7c cnt 30960 cycle 282799876 percent 0, cycle1 533575876, percent 1
[959708.486782] vcpu 1 reason 0x3ff cnt 100 cycle 3519897 percent 0, cycle1 4329897, percent 0
[959708.496170] vcpu 2 reason 0x60 cnt 91919 cycle 841200272 percent 1, cycle1 1585744172, percent 3
[959708.506137] vcpu 2 reason 0x61 cnt 2 cycle 174742 percent 0, cycle1 190942, percent 0
[959708.515036] vcpu 2 reason 0x64 cnt 5958 cycle 21134449 percent 0, cycle1 69394249, percent 0
[959708.524622] vcpu 2 reason 0x72 cnt 124 cycle 790244 percent 0, cycle1 1794644, percent 0
[959708.533816] vcpu 2 reason 0x78 cnt 203 cycle 26569948 percent 0, cycle1 28214248, percent 0
[959708.543302] vcpu 2 reason 0x7b cnt 20 cycle 603947 percent 0, cycle1 765947, percent 0
[959708.552298] vcpu 2 reason 0x7c cnt 21833 cycle 204075656 percent 0, cycle1 380922956, percent 0
[959708.562174] vcpu 2 reason 0x3ff cnt 57 cycle 1535466 percent 0, cycle1 1997166, percent 0
[959708.571463] vcpu 3 reason 0x60 cnt 835856 cycle 4624889228 percent 9, cycle1 11395322828, percent 23
[959708.581832] vcpu 3 reason 0x61 cnt 2 cycle 182752 percent 0, cycle1 198952, percent 0
[959708.590732] vcpu 3 reason 0x64 cnt 72851 cycle 235045747 percent 0, cycle1 825138847, percent 1
[959708.600607] vcpu 3 reason 0x72 cnt 146 cycle 1068220 percent 0, cycle1 2250820, percent 0
[959708.609894] vcpu 3 reason 0x77 cnt 1 cycle 16369 percent 0, cycle1 24469, percent 0
[959708.618595] vcpu 3 reason 0x78 cnt 58 cycle 2225467 percent 0, cycle1 2695267, percent 0
[959708.627783] vcpu 3 reason 0x7c cnt 31019 cycle 282756325 percent 0, cycle1 534010225, percent 1
[959708.637661] vcpu 3 reason 0x3ff cnt 88 cycle 2019692 percent 0, cycle1 2732492, percent 0
[   49.329639] Total 50353519748 cycles
[   49.330617] cpu 0 netrx cnt 39736 cycles 94555559 percent 0
[   49.331575] cpu 0 tcp_send cnt 4239361 cycles 23170072766 percent 46
[   49.332705] cpu 0 swiotlb_memcpy cnt 1699171 cycles 193729386 percent 0
[   49.333874] cpu 0 swiotlb_find cnt 1698740 cycles 480480250 percent 0
[   49.335107] cpu 0 swiotlb_release cnt 933090 cycles 265023080 percent 0
[   49.336301] cpu 1 netrx cnt 859084 cycles 12297097843 percent 24
[   49.337411] cpu 1 swiotlb_memcpy cnt 1079118 cycles 180351717 percent 0
[   49.338604] cpu 1 swiotlb_find cnt 1079114 cycles 486733631 percent 0
[   49.339766] cpu 1 swiotlb_release cnt 1835198 cycles 603493426 percent 1
[   49.340921] cpu 2 netrx cnt 40121 cycles 95295950 percent 0
[   49.341902] cpu 2 swiotlb_memcpy cnt 1682802 cycles 196321520 percent 0
[   49.343049] cpu 2 swiotlb_find cnt 1682782 cycles 476130125 percent 0
[   49.344177] cpu 2 swiotlb_release cnt 929120 cycles 262252346 percent 0
[   49.345381] cpu 3 netrx cnt 861817 cycles 12447368557 percent 24
[   49.346450] cpu 3 swiotlb_memcpy cnt 1075901 cycles 175548478 percent 0
[   49.347673] cpu 3 swiotlb_find cnt 1075901 cycles 487728313 percent 0
[   49.348810] cpu 3 swiotlb_release cnt 1839134 cycles 602484763 percent 1

8         Threads
32        Connections per thread
30        Seconds


ALL STATS 
============================================================================================================================
Type         Ops/sec     Hits/sec   Misses/sec    Avg. Latency     p50 Latency     p99 Latency   p99.9 Latency       KB/sec
----------------------------------------------------------------------------------------------------------------------------
Sets         5817.81          ---          ---         4.51660         3.71100        14.01500        61.69500   1489603.69
Gets        58137.26      5845.21     52292.05         3.94853         3.21500        12.86300        22.01500      2037.46
Waits           0.00          ---          ---             ---             ---             ---             ---          ---
Totals      63955.07      5845.21     52292.05         4.00021         3.26300        12.99100        22.65500   1491641.15
```
