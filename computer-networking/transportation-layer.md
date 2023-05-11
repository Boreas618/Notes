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

## Mutiplexing and Demultiplexing

Extending the host-to-host delivery service provided by the network layer to a process-to-process delivery service for applications running on the hosts.

The transport layer receives segements fromt the network layer and deliver these segments to the processes.

A process can have one or more sockets. The transport layer in the receiving host does not actually deliver data directly to a process, but instead to an intermediary socket.

Each transport-layer segment has a set of fields to direct incoming transport-layer segements to the appropriate sockets. This job of delivering the data in a transport-layer segment to the correct socket is called **demultiplexing**.

The job of gathering data chunks at the source host from different sockets, encapsulating each data chunk with header information (that will later be used in demultiplexing) to create segments, and passing the segments to the network layer is called **multiplexing**.

* Multiplexing: add header information
* Demultiplexing: parse header information

<img src="https://p.ipic.vip/7ru303.png" alt="Screenshot 2023-05-08 at 5.31.21 PM" style="zoom:50%;" />

UDP way of demultiplexing: a segment $$\rightarrow$$ host $$\rightarrow$$ corresponding socket $$\rightarrow$$ process.

**Connectionless Mutiplexing and Demultiplexing**

```Python
clientSocket = socket(AF_INET, SOCK_DGRAM)
```

We can associate a specific port number to this UDP socket via the socket `bind()` method.

```python
clientSocket.bind(('', 19157))
```

A UDP socket is fully identified by a two-tuple consisting of a destination IP address and a destination port number.

The source port number is needed because the destination needs to send some information back to the source.

**Connection-Oriented Mutiplexing and Demutiplexing**

A TCP socket is identified by a four-tuple: (source IP address, source port number, destination IP address, destination port number)

In contrast with UDP, two arriving TCP segments with different source IP addresses or source port numbers will (with the exception of a TCP segment carrying the original connection- establishment request) be directed to two different sockets.

* The TCP server application has a "welcoming socket", that waits for connection-establishment request from TCP clients on port 12000.

* The TCP client creates a socket and sends a connection establishment request segement with the lines:

  ```python
  clientSocket = socket(AF_INET, SOCK_STREAM)
  clientSocket.connect((serverName, 12000))
  ```

* A connection-establishment request is nothing more than a TCP segment with destination port number 12000 and a special connection-establishment bit set in the TCP header. The segment also includes a source port number that was chosen by the client.

* When the host operating system of the computer running the server process receives the incoming connection-request segment with destination port 12000, it locates the server process that is waiting to accept a connection on port number 12000. The server process then creates a new socket:

  ```python
  connectionSocket, addr = serverSocket.accept()
  ```

* Also, the transport layer at the server notes the following four values in the connection-request segment: (1) the source port number in the segment, (2) the IP address of the source host, (3) the destination port number in the segment, and (4) its own IP address. The newly created connection socket is identified by these four values.

**Web Servers and TCP**

* Persistent HTTP
* Non-persistent HTTP 

They are all based on TCP. 

## Connectionless Transport: UDP

UDP takes messages from the application process, attaches source and destination port number fields for the multiplexing/demultiplexing service, adds two other small fields, and passes the resulting segment to the network layer. The network layer encapsulates the transport-layer segment into an IP datagram and then makes a best-effort attempt to deliver the segment to the receiving host.

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
