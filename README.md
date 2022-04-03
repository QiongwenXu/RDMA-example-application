# RDMA-example-application
Writing RDMA applications on Linux Example programs by Roland Dreier.

source: http://www.digitalvampire.org/rdma-tutorial-2007/notes.pdf

# Set up interfaces
This application uses RDMA to send data to a server, so make sure the NIC supports 
InfiniBand and the interface connects to the NIC. [Cloudlab](https://docs.cloudlab.us/hardware.html)
provides machines (`r320` and `c6220`) with Mellanox InfiniBand interfaces. We use 
two `r320` machines to run the application, and the corresponding InfiniBand interfaces 
are both `enp8s0d1` in our case.

Set up interfaces and assign them ip addresses.

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
