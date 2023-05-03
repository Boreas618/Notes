# Transportation Layer

## Introduction and Transport-Layer Services

A transport-layer protocol provides for **logical communication** between application processes running on different hosts.

By *logical communication*, we mean that from an application’s perspective, it is as if the hosts running the processes were directly connected.

<img src="https://p.ipic.vip/22q5cp.png" alt="Screenshot 2023-05-03 at 1.44.52 PM" style="zoom:50%;" />

On the sending side, the transport layer converts the application-layer messages it receives from a sending application process into **transport-layer packets**, known as **transport-layer segments**. 

This is done by (possibly) breaking the application messages into smaller chunks and adding a **transport-layer header to each chunk** to create the transport-layer segment. The transport layer then passes the segment to the **network layer** at the sending end system, where the segment is encapsulated **within a network-layer packet (a datagram)** and sent to the destination.

Network routers act only on the network-layer fields of the datagram; that is, they do not examine the fields of the transport-layer segment encapsulated with the datagram.

### Relationship Between Transport and Network Layers

A transport-layer protocol provides logical communication between ***processes*** running on different hosts, a network-layer protocol provides logical communication between ***hosts***. 

The services that a transport protocol can provide are often constrained by the service model of the underlying network-layer protocol. 

**Example:** delay or bandwidth guarantees

### Overview of the Transport Layer in the Internet

We temporarily refer to TCP and UDP packets as segments, network packet as datagram.

The Internet’s network-layer protocol has a name—IP, for Internet Protocol. IP provides logical communication between hosts. The IP service model is a **best-effort delivery service**. This means that IP makes its “best effort” to deliver segments between communicating hosts, but it makes no guarantees. IP is unreliable. 

Every host has at least one network-layer address, a so-called IP address.

Extending host-to-host delivery to process-to-process delivery is called **transport-layer multiplexing** and **demultiplexing**. 

The two minimal transport-layer services—process-to-process data delivery and error checking—are the only two services that UDP provides!

TCP provides reliable data transfer through flow control, sequence numbers, acknowledgments, and timers.

TCP also provides congestion control. This is done by regulating the rate at which the sending sides of TCP connections can send traffic into the network. UDP traffic, on the other hand, is unregulated.

