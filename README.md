# RDMA-example-application
Writing RDMA applications on Linux Example programs by Roland Dreier.

source: http://www.digitalvampire.org/rdma-tutorial-2007/notes.pdf

# Set up interfaces
This application uses RDMA to send data to a server, so make sure the NIC supports 
InfiniBand and the interface connects to the NIC. [Cloudlab](https://docs.cloudlab.us/hardware.html)
provides machines (`r320` and `c6220`) with Mellanox InfiniBand interfaces. We use 
two `r320` machines (Ubuntu 20.04) to run the application, and the corresponding 
InfiniBand interfaces are both `enp8s0d1` in our case. If no InfiniBand interfaces 
is set up, follow the instructions to install Mellanox OFED driver first.

## Install Mellanox OFED driver if no InfiniBand interfaces are set up (optional)
Download Mellanox OFED driver package: https://network.nvidia.com/products/infiniband-drivers/linux/mlnx_ofed/
For example, on `r320` machine, you will download [`4.9-4.1.7.0` version](https://developer.nvidia.com/networking/mlnx-ofed-eula?mtag=linux_sw_drivers&mrequest=downloads&mtype=ofed&mver=MLNX_OFED-4.9-4.1.7.0&mname=MLNX_OFED_LINUX-4.9-4.1.7.0-ubuntu20.04-x86_64.tgz)

Unzip the downloaded package
```
tar -xvzf MLNX_OFED_LINUX-4.9-4.1.7.0-ubuntu20.04-x86_64.tgz
```

Install Mellanox OFED driver without subnet manager (i.e., `opensm` and `opensm-doc`). Because unlike Ethernet 
where Cloudlab provisions private network VLANs. For your network links, the Infiniband fabric is a single,
shared fabric. We need to avoid affecting the other users.
```
cd MLNX_OFED_LINUX-4.9-4.1.7.0-ubuntu20.04-x86_64
sudo ./mlnxofedinstall --without-opensm --without-opensm-doc
sudo /etc/init.d/openibd restart
```

After installation, you can use `ip a` to check whether the InfiniBand interface shows up.

Remove two dependencies so that we can upgrade them later.
```
sudo apt-get --purge remove libibverbs-dev librdmacm-dev
```

## Set up interfaces and assign them ip addresses.

client: set ip as `10.10.1.1`. 
```
sudo ip link set dev enp8s0d1 down
sudo ip addr add 10.10.1.1/24 dev enp8s0d1
sudo ip link set dev enp8s0d1 up
```

server: set ip as `10.10.1.2`.
```
sudo ip link set dev enp8s0d1 down
sudo ip addr add 10.10.1.2/24 dev enp8s0d1
sudo ip link set dev enp8s0d1 up
```

# Dependencies
Install Linux RDMA user space libraries on client and server
```
sudo apt-get update 
sudo apt-get install libibverbs-dev librdmacm-dev
```

# Compile and run
Compile client and server:

Client:
```
cc -o client client.c -lrdmacm -libverbs
```
Server:
```
cc -o server server.c -lrdmacm -libverbs
```

Run server:
```
./server
```
Run client (syntax: client [servername] [val1] [val2]):
```
./client 10.10.1.2 123 567
```
Expected output on client:

```
123 + 567 = 690
```

# Additional notes
These instructions were tested on a fresh Ubuntu 20.04 LTS system.
```
gcc -v
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc/x86_64-linux-gnu/9/lto-wrapper
OFFLOAD_TARGET_NAMES=nvptx-none:hsa
OFFLOAD_TARGET_DEFAULT=1
Target: x86_64-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Ubuntu 9.3.0-17ubuntu1~20.04' --with-bugurl=file:///usr/share/doc/gcc-9/README.Bugs --enable-languages=c,ada,c++,go,brig,d,fortran,objc,obj-c++,gm2 --prefix=/usr --with-gcc-major-version-only --program-suffix=-9 --program-prefix=x86_64-linux-gnu- --enable-shared --enable-linker-build-id --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --libdir=/usr/lib --enable-nls --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --with-default-libstdcxx-abi=new --enable-gnu-unique-object --disable-vtable-verify --enable-plugin --enable-default-pie --with-system-zlib --with-target-system-zlib=auto --enable-objc-gc=auto --enable-multiarch --disable-werror --with-arch-32=i686 --with-abi=m64 --with-multilib-list=m32,m64,mx32 --enable-multilib --with-tune=generic --enable-offload-targets=nvptx-none=/build/gcc-9-HskZEa/gcc-9-9.3.0/debian/tmp-nvptx/usr,hsa --without-cuda-driver --enable-checking=release --build=x86_64-linux-gnu --host=x86_64-linux-gnu --target=x86_64-linux-gnu
Thread model: posix
gcc version 9.3.0 (Ubuntu 9.3.0-17ubuntu1~20.04)
```

```
cat /etc/issue
Ubuntu 20.04 LTS \n \l
```

```
uname -r
5.4.0-100-generic
```
