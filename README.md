# AWS OFI RCCL

AWS OFI RCCL is a plug-in which enables EC2 developers to use
[libfabric](https://github.com/ofiwg/libfabric) as a network provider while
running [AMD's RCCL](https://github.com/ROCmSoftwarePlatform/rccl) based applications.

## Overview

Machine learning frameworks running on top of AMD GPUs use a library called
[RCCL](https://github.com/ROCmSoftwarePlatform/rccl) which provides standard collective
communication routines for an arbitrary number of GPUs installed across single
or multiple nodes.

This project implements a plug-in which maps RCCLs connection-oriented
transport APIs to [libfabric's](https://ofiwg.github.io/libfabric/)
connection-less reliable interface. This allows RCCL applications to take
benefit of libfabric's transport layer services like reliable message support
and operating system bypass.

## Requirements

The plug-in currently supports the following distributions:
* Amazon Linux
* Amazon Linux 2
* Redhat Enterprise Linux 7 and 8
* Ubuntu 18.04 and 20.04 LTS
* CentOS 7 and 8

It requires [Libfabric](http://github.com/ofiwg/libfabric/)
and [RCCL](http://github.com/https://github.com/ROCmSoftwarePlatform/rccl).  Please see the
[Release notes](http://github.com/aws/aws-ofi-rccl/releases) for
information on version compatibility.

Libfabric supports various providers. The plug-in can choose only those which
support the following features as defined in the
[libfabric API documentation](https://github.com/ofiwg/libfabric/tree/master/man/).

* Tagged messaging (`FI_TAGGED`, `FI_MSG`)
* Data transfer context structures (`FI_CONTEXT`)
* Reliable datagram endpoints (`FI_EP_RDM`)
* Send after Send ordering semantics (`FI_ORDER_SAS`)
* Communication with remote endpoints (`FI_REMOTE_COMM`)

For GPUDirect RDMA support, it requires these additional features from libfabric
providers. If these are not supported by any provider on system, plug-in turns off
GPUDirect RDMA support.

* Transfers to/from device memory (`FI_HMEM`)
* Remote memory operations (`FI_RMA`, `FI_READ`)

## Getting Started

### Dependencies

`aws-ofi-rccl` requires working installations of RCCL and libfabric. You can
find the instructions for installing the first two at
[RCCL installation](https://github.com/ROCmSoftwarePlatform/rccl) and
[libfabric installation](https://github.com/ofiwg/libfabric) respectively.

### Build latest RCCL develop branch
```
cd ~
git clone https://github.com/ROCmSoftwarePlatform/rccl.git
cd rccl
mkdir build
cd build/
CXX=/opt/rocm-5.2.0/bin/hipcc cmake -DCMAKE_PREFIX_PATH=/opt/rocm/ ..
make -j
```
### Plugin build Instructions

The plugin uses GNU autotools for its build system. You can build it as follows:

```
$ ./autogen.sh
$ CC=cc ./configure --with-libfabric=/opt/cray/libfabric/1.15.0.0 --with-hip=/opt/rocm-5.2.0 --with-rccl=path-to-rccl-build-folder
$ make
$ sudo make install
```

"--with-rccl=path-to-rccl-build-folder": Let's suppose we build RCCL at /home/username/rccl/build,
then "--with-rccl=/home/username/rccl/build".

If you want to install the plugin in a custom path, use the `--prefix`
configure flag to provide the path. You can also point the build to custom
dependencies with the following flags:

```
  --with-libfabric=PATH   Path to non-standard libfabric installation
  --with-hip=PATH         Path to non-standard ROCm installation
  --with-rccl=PATH        Path to non-standard RCCL installation
  --with-mpi=PATH         Path to non-standard MPI installation
```

To enable trace messages for debugging (disabled by default), use the
following config option:

```
   --enable-trace         Enable printing trace messages
```

By default, tests are built.  To disable building tests, use the following
config option:

```
   --disable-tests        Disable build of tests.
```

### Plugin Configurations

The plugin allows to configure the following variables at run-time according to your environment.

<table>
   <thead>
      <th>Parameter</th>
      <th>Description</th>
      <th>Type</th>
      <th>Accepted Value</th>
   </thead>
   <tr>
      <td><code>OFI_NCCL_USE_IPV6_TCP</code></td>
      <td>Allow using endpoints with IPv6 addressing format for TCP provider. Users can specify to use a preferred libfabric provider with `FI_PROVIDER` environment variable.</td>
      <td>Boolean</td>
      <td>0/1 (Default: 0)</td>
   </tr>
   <tr>
      <td><code>OFI_NCCL_TCP_EXCLUDE_IF</code></td>
      <td>List of interface names to be filtered out for TCP provider. Users can specify to use a preferred libfabric provider with `FI_PROVIDER` environment variable.</td>
      <td>String</td>
      <td>Comma-separated list of interface names (Default: "lo,docker0")</td>
   </tr>
   <tr>
      <td><code>OFI_NCCL_GDR_FLUSH_DISABLE</code></td>
      <td>Disable flush operation when using GPUDirect.</td>
      <td>Boolean</td>
      <td>0/1 (Default: 1)</td>
   </tr>
   <tr>
      <td><code>OFI_NCCL_CUDA_FLUSH_ENABLE</code></td>
      <td>When using GPUDirect use the cudaDeviceFlushGPUDirectRDMAWrites to
      enforce data consistency at the receiving GPU. Requires CUDA 11.3 or
      later. Note that this function only provides a GPU memory fence and
      requires that data has already been delivered to GPU memory. Some
      networks and PCIe configurations require an additional network-level
      flush that is not provided by this option.</td>
      <td>Boolean</td>
      <td>0/1 (Default: 0)</td>
   </tr>
</table>


### Running Unit Tests

Running unit tests requires a working MPI installation and a
[MPI setup](https://www.open-mpi.org/faq/?category=running) between the
communicating hosts.  To install MPI, you can use standard packages provided
for your linux distribution. Once MPI is setup, you can use commands like below
for running any test of your choice.

```
mpirun -n 2 --host <host-1>,<host-2> $INSTALL_PREFIX/bin/rccl_message_transfer
```

**Note:** All tests require 2 MPI ranks to run except [ring.c](tests/ring.c)
which requires atleast 3 ranks.


### Running rccl-perf tests

To run standard `rccl-perf` tests with the `aws-ofi-rccl` plugin, you can
follow the instructions below.

1. Clone the repository
```
git clone https://github.com/ROCmSoftwarePlatform/rccl-tests.git
```

2. Build the tests
```
cd  rccl-tests/
MPI_HOME=/opt/cray/pe/mpich/8.1.15/ofi/cray/10.0 ROCM_PATH=/opt/rocm-5.2.0 MPI=1 NCCL_HOME=~/rccl/build/ make -j
```

3. Run perf tests
```
export NCCL_DEBUG=INFO
export FI_CXI_ATS=0
export LD_LIBRARY_PATH=/home/${USER}/rccl/build:/home/${USER}/aws-ofi-rccl/src/.libs/:/opt/cray/libfabric/1.15.0.0/lib64/:/opt/rocm-5.2.0/lib
export FI_LOG_LEVEL=info
export NCCL_NET_GDR_LEVEL=3

srun -N 4 ~/rccl-tests/build/all_reduce_perf -b 8 -e 1G -f 2 -g 8
```

If you installed the AWS libfabric plugin in a custom prefix, ensure
`LD_LIBRARY_PATH` is set to include that prefix so the perf test binaries can
find the plugin.

## Getting Help

If you have any issues in building or using the package or if you think you may
have found a bug, please open an
[issue](https://github.com/ROCmSoftwarePlatform/aws-ofi-rccl/issues).

## Contributing

Reporting issues and sending pull requests are always welcome. To learn how you
can contribute, please look at our
[contributing guidelines](CONTRIBUTING.md#contributing-guidelines).

## License

This library is licensed under the [Apache 2.0 License](LICENSE).
