# CNDP - Cloud Native Data Plane

## Installation Instructions

This provides a minimal instructions to build and run CNDP applications.
For more information, refer to the CNDP documentation.

### Prerequisites

If behind a proxy server you may need to setup a number of configurations to allow access via the server.
Some commands i.e. apt, dnf, git, ssh, curl, wget and others will need configuration to work correctly.
Please refer to apt, dnf, git and other documentations to enable access through a proxy server.

### Dependencies

The following package are required to build CNDP libraries and examples.

#### UBUNTU
```bash
sudo apt-get update && sudo apt-get install -y \
    build-essential libbsd-dev libelf-dev libjson-c-dev libnl-3-dev libnl-cli-3-dev libnuma-dev \
    libpcap-dev meson pkg-config git gcc-multilib clang llvm lld m4
```
#### Fedora
```bash
dnf update && dnf -y install \
    @development-tools git libbsd-devel json-c-devel libnl3-devel libnl3-cli \
    numactl-libs libbpf-devel libbpf meson ninja-build gcc-c++ \
    libpcap libpcap-devel clang llvm lld m4
```

#### Optional packages needed to build documentation

```bash
doxygen and python3-sphinx
```

### libbpf

The [libbpf](https://github.com/libbpf/libbpf) is a dependency of CNDP.

#### _Note:_

Older version of Kernel will not work with latest libbpf so older version like v0.5.0 or v0.6.1
should be used.

### Install libbpf from source

Clone libxdp:
```bash
git clone https://github.com/libbpf/libbpf.git
cd libbpf
git checkout v0.5.0  # this step is optional only needed if the kernel version is old.
make -C src && sudo make -C src install
```

The library and pkgconfig file is installed to /usr/lib64, which is not where the loader or
pkg-config looks. Fix this by editing the ldconfig file and PKG_CONFIG_PATH.

### Install libxdp from source

Clone libxdp:

```cmd
git clone https://github.com/xdp-project/xdp-tools.git
cd xdp-tools/
```

Run ./configure

```bash
./configure
Found clang binary 'clang' with version 14 (from 'Ubuntu clang version 14.0.0-1ubuntu1')
libbpf support: Submodule 'libbpf' (https://github.com/xdp-project/libbpf/) registered for path 'lib/libbpf'
Cloning into '/xdp-tools/lib/libbpf'...
Submodule path 'lib/libbpf': checked out '87dff0a2c775c5943ca9233e69c81a25f2ed1a77'
submodule v0.8.0
  perf_buffer__consume support: yes (submodule)
  btf__load_from_kernel_by_id support: yes (submodule)
  btf__type_cnt support: yes (submodule)
  bpf_object__next_map support: yes (submodule)
  bpf_object__next_program support: yes (submodule)
  bpf_program__insn_cnt support: yes (submodule)
  bpf_map_create support: yes (submodule)
  perf_buffer__new_raw support: yes (submodule)
  bpf_xdp_attach support: yes (submodule)
zlib support: yes
ELF support: yes
pcap support: yes
secure_getenv support: yes
```
In the top level xdp-tools directory:

```cmd
make -j; PREFIX=/usr make -j install
```

check pkg-config can find libxdp:

```cmd
pkg-config --modversion libxdp
1.2.2
```

on some systems it maybe necessary to run:

```bash
export PKG_CONFIG_PATH=/usr/lib/pkgconfig
```

## Hugepage Configuration

Hugepage support is optional, but preferred as it provides a performance boost. Details of Hugepage Configuration can be found [here](https://github.com/CloudNativeDataPlane/cndp/blob/9fa83c17c75930eee2355476e23cf786a533756c/doc/guides/linux_gsg/linux_gsg.rst)

## Build CNDP

### Clone and build CNDP

```bash
git clone https://github.com/CloudNativeDataPlane/cndp.git
cd cndp
make
```

Other targets exist, most are wrappers around tools/cne-build.sh.

```bash
make help
or
make rebuild # rebuild will clean and build CNDP with -O3
or
make clean debug # to build a debug image with -O0
or
make docs
```

## Run CNDP examples

### helloworld

The most basic example is `helloworld`.

```bash
./builddir/examples/helloworld/helloworld

Max threads: 512, Max lcores: 32, NUMA nodes: 1, Num Threads: 1
hello world! from thread index 0 for index 0
Ctrl-C to exit
```

### cndpfwd

An example that uses networking is `cndpfwd`. It requires the underlying network interface
uses, e.g. AF_XDP sockets. Make sure the kernel on which you intend to run the application
supports AF_XDP sockets, i.e. CONFIG_XDP_SOCKETS=y.

```bash
grep XDP_SOCKETS= /boot/config-`uname -r`
```

Configure an ethtool filter to steer packets to a specific queue.

```bash
sudo ethtool -N <devname> flow-type udp4 dst-port <dport> action <qid>
sudo ip link set dev <devname> up
```

Instruct `cndpfwd` to receive, count, and drop all packets on the previously configured
queue. To configure `cndpfwd`, edit the examples/cndpfwd/fwd.jsonc configuration file. Make
sure the "lports" section has the same netdev name and queue id for which the ethtool filter
is configured. Make sure the "threads" section has the correct "lports" configured. Then
launch the application, specifying the updated configuration file.

```bash
sudo ./builddir/examples/cndpfwd/cndpfwd -c examples/cndpfwd/fwd.jsonc drop
```
