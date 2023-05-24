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

We temporarily refer to TCP and UDP packets as **segments**, network packet as **datagram**.

> In real world terminology:
>
> - "Packets" are used in the network layer (e.g., IP packets).
> - "Datagrams" are used in the transport layer for UDP.
> - "Segments" are used in the transport layer for TCP.

The Internet’s network-layer protocol has a name—IP, for Internet Protocol. IP provides logical communication between hosts. The IP service model is a **best-effort delivery service**. This means that IP makes its “best effort” to deliver segments between communicating hosts, but it makes no guarantees. IP is unreliable. 

Every host has at least one network-layer address, a so-called IP address.

Extending host-to-host delivery to process-to-process delivery is called **transport-layer multiplexing** and **demultiplexing**. 

The two minimal transport-layer services—process-to-process data delivery and error checking—are the only two services that UDP provides!

> ***UDP can provide error checking!***

TCP provides reliable data transfer through flow control, sequence numbers, acknowledgments, and timers.

TCP also provides congestion control. This is done by regulating the rate at which the sending sides of TCP connections can send traffic into the network. UDP traffic, on the other hand, is unregulated.

## Mutiplexing and Demultiplexing

Extending the host-to-host delivery service provided by the network layer to a process-to-process delivery service for applications running on the hosts.

The transport layer receives segements fromt the network layer and deliver these segments to the processes.

A process can have one or more sockets. The transport layer in the receiving host does not actually deliver data directly to a process, but instead to an intermediary socket.

> A process **can** have one or more sockets
>
> A port **can** be shared by several sockets. For example, a connection is identified by the 4-tuple (source_ip, source_port, dest_ip, dest_port) and a server can handle mutiple connections on the same port.
>
> In general, a port **cannot** be shared by multiple applications (processes), but it can be done with forking.

Each transport-layer segment has a set of fields to direct incoming transport-layer segements to the appropriate sockets. This job of delivering the data in a transport-layer segment to the correct socket is called **demultiplexing**.

The job of gathering data chunks at the source host from different sockets, encapsulating each data chunk with header information (that will later be used in demultiplexing) to create segments, and **passing the segments** to the network layer is called **multiplexing**.

* Multiplexing: add header information and pass it to the network layer
* Demultiplexing: parse header information and pass it the socket

<img src="https://p.ipic.vip/7ru303.png" alt="Screenshot 2023-05-08 at 5.31.21 PM" style="zoom:50%;" />

UDP way of demultiplexing: a segment $$\rightarrow$$ host $$\rightarrow$$ corresponding socket $$\rightarrow$$ process.

**Connectionless Mutiplexing and Demultiplexing**

```Python
# If you plan to use datagram-based protocol like UDP, you should specify the SOCK_DGRAM option
clientSocket = socket(AF_INET, SOCK_DGRAM)
```

We can associate a specific port number to this UDP socket via the socket `bind()` method.

```python
clientSocket.bind(('', 19157))
```

**A UDP socket is fully identified by a *two-tuple* consisting of a destination IP address and a destination port number.**

A UDP communication is fully identified by a four-tuple consisting of a source IP address, source port number, destination IP address, and destination port number. The source IP address and port number are needed because the destination may need to send some information back to the source.

**Connection-Oriented Mutiplexing and Demutiplexing**

A TCP socket is identified by a ***four-tuple***: (source IP address, source port number, destination IP address, destination port number)

In contrast with UDP, two arriving TCP segments with different source IP addresses or source port numbers will (with the exception of a TCP segment carrying the original connection- establishment request) be directed to two different sockets.

* The TCP server application has a "welcoming socket", that waits for connection-establishment request from TCP clients on port 12000.

* The TCP client creates a socket and sends a connection establishment request segement with the lines:

  ```python
  # TCP is a stream-based protocol
  clientSocket = socket(AF_INET, SOCK_STREAM)
  clientSocket.connect((serverName, 12000))
  ```

* A connection-establishment request is nothing more than a TCP segment with destination port number 12000 and **a special connection-establishment bit** set in the TCP header. **The segment also includes a source port number that was chosen by the client.**

* When the host operating system of the computer running the server process receives the incoming connection-request segment with destination port 12000, it locates the server process that is waiting to accept a connection on port number 12000. The server process then creates a new socket:

  ```python
  # A server socket is created after receving the connection-estabishement request from the client
  connectionSocket, addr = serverSocket.accept()
  ```

* Also, the transport layer at the server notes the following four values in the connection-request segment: (1) the source port number in the segment, (2) the IP address of the source host, (3) the destination port number in the segment, and (4) its own IP address. The newly created connection socket is identified by these four values.

**Web Servers and TCP**

* Persistent HTTP
* Non-persistent HTTP 

They are all based on TCP. 

## Connectionless Transport: UDP

UDP takes messages from the application process, attaches source and destination port number fields for the multiplexing/demultiplexing service, adds two other small fields, and passes the resulting segment( datagram more specificly) to the network layer. The network layer encapsulates the transport-layer segment into an IP datagram(packet more specificly) and then makes a best-effort attempt to deliver the segment to the receiving host.

Why some application developer chooses UDP rather than TCP?

* UDP doesn't have a congestion-control
* UDP is faster without three-way handshake
* No connection state. A server devoted to a particular application can typically support more active clients
* Small packet header overhead. The TCP segment has 20 bytes of header overhead in every segment, whereas UDP has only 8 bytes of overhead.

### UDP Segment Structure

<img src="https://p.ipic.vip/n12f7a.png" alt="Screenshot 2023-05-09 at 12.16.00 AM" style="zoom:50%;" />

The UDP header has only four fields, each consisting of two bytes.

The length field specifies the number of bytes in the UDP segment (header plus data).

The checksum is used by the receiving host to check whether errors have been introduced into the segment.

### UDP Checksum

UDP at the sender side performs the 1s complement of the sum of all the 16-bit words in the segment, with any overflow encountered during the sum being wrapped around.

In the receiver side, all 16-bit words are added, including the checksum. The expected outcome is 1111111111111111. If we encounter 0s, then errors are introduced.

Although UDP provides error checking, it does not do anything to recover from an error. Some implementations of UDP simply discard the damaged segment; others pass the damaged segment to the application with a warning.

## Principles of Reliable Data Transfer

Assumption:  packets will be delivered in the order in which they were sent, with some packets possibly being lost; that is, the underlying channel will not reorder packets.

**ARQ(Automatic Repeat reQuest) protocols**: 

Use both **positive acknowledgements** and **negative acknowledgements**. Repeat the message in error.

Capabilities needed:

* **Error detection**

* **Receiver feedback**

  The positive (ACK) and negative (NAK) acknowledgment replies in the message-dictation scenario are examples of such feedback. 

  Simply one bit.

* **Retransmission**

  A packet that is received in error at the receiver will be retransmitted by the sender.

<img src="https://p.ipic.vip/1mzrrk.png" alt="Screenshot 2023-05-11 at 2.50.28 PM" style="zoom: 33%;" />

It is important to note that when the sender is in the wait-for-ACK-or-NAK state, it cannot get more data from the upper layer. This is known as **stop-and-wait** protocols.

However, the ACK or NAK can also be corrupted. One solution is to resend the packet when we received a garbled ACK or NAK packet. However, a new issue is introduced in this solution. The receiver of the resented packet doesn't know if the arriving packet contains new data or is a retransmission.

To address this issue, we add the **sequence number** into the packet. For the stop-and-wait protocol, we only need one bit to represent the sequence number. 

<img src="https://p.ipic.vip/346xzp.png" alt="Screenshot 2023-05-11 at 3.08.32 PM" style="zoom:50%;" />

<img src="https://p.ipic.vip/5bzdv1.png" alt="Screenshot 2023-05-11 at 3.07.55 PM" style="zoom:67%;" />

Chances are that the underlying channel can lose packets as well, a not-uncommon event in today's computer network. We put the burden of detecting and recovering from lost lost packets on the sender. The sender waits for a certain length of time to decide that a packet has been lost. The sender judiciously choose a time value such that packet loss is likely, although not guaranteed, to have happened. But If a packet experiences a particularly large delay, the sender may retransmit the packet even though neither the data packet nor its ACK have been lost. But rdt2.2 above can already handle this duplicate.

Implementing a time-based retransmission mechanism requires a **countdown timer** that can interrupt the sender after a given amount of time has expired. The sender will thus need to be able to (1) start the timer each time a packet (either a first-time packet or a retransmission) is sent, (2) respond to a timer interrupt (taking appropriate actions), and (3) stop the timer.

<img src="https://p.ipic.vip/tasxuk.png" alt="Screenshot 2023-05-23 at 8.51.50 PM" style="zoom:50%;" />

### Pipelined Reilable Data Transfer Protocols

The issue with the above design of reliable protocol is that it is a **stop-and-wait** protocol. We adopt pipelining to address this issue.

The range of sequence number needs to be increased now. The sender and receiver sides of the protocols may have to buffer more than one packet.

Two pipelined error recovery can be identified: **Go-Back-N** and **selective repeat**

### Go-Back-N

The sender is allowed to transmit multiple packets (when available) without waiting for an acknowledgment, but is constrained to have no more than some maximum allowable number, $N$, of unacknowledged packets in the pipeline.

![Screenshot 2023-05-23 at 9.17.21 PM](https://p.ipic.vip/e13cdw.png)

We define `base` to be the sequence number of the oldest unacknowledged packet and `nextseqnum` to be the smallest unused sequence number.

| Interval                 | Description                   |
| ------------------------ | ----------------------------- |
| $[0, base-1]$            | Transmitted and acknowledged  |
| $[base, nextseqnum-1]$   | Sent but not yet acknowledged |
| $[nextseqnum, base+N-1]$ | Can be sent                   |
| $[base+N, \infty]$       | Cannot be used                |

$N$ is referred to as the **window size**. GBN protocol is a **sliding-window protocol**.  
