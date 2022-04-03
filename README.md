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
