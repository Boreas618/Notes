# Network Programming

It is important to realize that clients and servers are **processes** and not machines, or *hosts* as they are often called in this context.

## Networks

The lowest level of the network is the LAN. Ethernet is the most popular LAN technology. An **Ethernet segment** consists of some wires (usually twisted pairs of wires) and a small box called a **hub**. One end is attached to an **adapter** on a host, and the other end is attached to a **port** on the hub. Every host sees every bit the hub receives.

Each Ethernet adapter has a globally unique 48-bit address that is stored in a nonvolatile memory on the adapter.

A host can send a chunk of bits called **frame** to any other host on the segment. The **header** bits record source, destination, length. They are followed by the **payload** bits. Every host adapter sees the frame, but only the destination host actually reads it.

Mutiple Ethernet segments are connected into **bridged Ethernets**, using a set of wires and small boxes called **bridges**. There are hub-bridge wires and bridge-bridge wires. 

<img src="https://p.ipic.vip/dq2bw4.png" alt="Screenshot 2023-05-03 at 1.50.30 AM" style="zoom:50%;" />

Multiple incompatible LANs can be connected by specialized computers called **routers** to form an **internet** (interconnected network). Each router has an adapter (port) for each network that it is connected to. **In general, routers can be used to build internets from arbitrary collections of LANs and WANs.**

<img src="https://p.ipic.vip/trtw42.png" alt="Screenshot 2023-05-03 at 11.08.34 AM" style="zoom:50%;" />

A layer of **protocol software** running on each host and router smooth out the difference between different networks. The protocol must provide two basic capabilities:

* Naming scheme

  Each host is assigned at least one of internet address that uniquely identifies it.

* Delivery mechanism

  The internet protocol defines a uniform way to bundle up data bits into discrete chunks called **packets**.

<img src="https://p.ipic.vip/ha5o1u.png" alt="Screenshot 2023-05-03 at 11.17.40 AM" style="zoom:50%;" />

1. The client copies the data from its virtual address space into the kernel buffer.

2. The protocol softeware on host A creates **a LAN1 frame** by appending an **internet packet header** and a **LAN1 frame header** to the data.

   **internet packet header** is addressed to internet host B.

   LAN1 **frame header** is addressed to the router (i.e. MAC address).

   The payload of the LAN1 frame is an internet packet, whose payload is the actual user data.

3. The LAN1 adapter copies the frame to the network.

4. When the frame reaches the router, the router’s LAN1 adapter reads it from the wire and passes it to the protocol software
5. The router fetches the destination internet address from the internet packet header and uses this as an index into a routing table to determine where to forward the packet, which in this case is LAN2. The router then strips off the old LAN1 frame header, prepends a new LAN2 frame header addressed to host B, and passes the resulting frame to the adapter.
6. The router’s LAN2 adapter copies the frame to the network.
7. When the frame reaches host B, its adapter reads the frame from the wire and passes it to the protocol software.
8. Finally, the protocol software on host B strips off the packet header and frame header. The protocol software will eventually copy the resulting data into the server’s virtual address space when the server invokes a system call that reads the data.

## The Global Internet

### IP Addresses

TCP/IP defines a uniform **network byte order** (big-endian byte order) for any integer data item, such as an IP address, that is carried across the network in a packet header. Addresses in IP address structures are always stored in (big-endian) network byte order, even if the host byte order is little-endian.

```c
#include <arpa/inet.h>

uint32_t htonl(uint32_t hostlong);
uint16_t htons(uint16_t hostshort);

// returns value in network byte order

uint32_t ntohl(uint32_t netlong);
uint16_t ntohs(unit16_t netshort);

// returns value in host byte order
```

Application programs can convert back and forth between IP addresses and dotted-decimal strings using the functions `inet_pton` and `inet_ntop`.

```c
#include <arpa/inet.h>

int inet_pton(AE_INET, const char *src, void *dst);
// returns: 1 if OK, 0 if src is invalid dotted decimal, −1 on error

const char *inet_ntop(AF_INET, const void *src, char *dst, socklen_t size);
// returns: pointer to a dotted-decimal string if OK, NULL on error
```

`AE_INET` for IPv4, `AE_INET6` for IPv6.

### Internet Domain Names

<img src="https://p.ipic.vip/gx2rzp.png" alt="Screenshot 2023-05-03 at 12.09.24 PM" style="zoom:50%;" />

### Internet Connections

Full duplex: the data can flow in both directions

A **socket** is an end point of a connection. Each socket has a corresponding *socket address* that consists of an Internet address and a 16-bit integer **port** and is denoted by the notation `address:port`.

The port in the server’s socket address is typically some *well-known port* that is permanently associated with the service. The mapping between well-known names( smtp, http ) and well-known ports( 25, 80 ) is contained in a file called `/etc/services`.

By default, HTTP requests use port 80 for non-secure (HTTP) connections and port 443 for secure (HTTPS) connections. However, if the server is running on a different port, you can specify the port number in the request URL, like `http://fduhole.com:8080/`.

A socket tuple: `(cliaddr:cliport, servaddr:servport)`

## The Sockets Interface

<img src="https://p.ipic.vip/jstaky.png" alt="Screenshot 2023-05-03 at 12.50.20 PM" style="zoom:50%;" />

From the perspective of the Linux kernel, a socket is an end point for communi- cation. From the perspective of a Linux program, a socket is an open file with a corresponding descriptor.
