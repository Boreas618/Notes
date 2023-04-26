# Application Layer

## Principles of Network Applications

### Network Architecture

In choosing the application architecture, an application developer will likely draw on one of the two predominant architectural paradigms used in modern network applications: the **client-server** architecture or the **peer-to-peer (P2P)** architecture.

Some of the better-known applications with a client-server architecture include the Web, FTP, Telnet, and e-mail.

In a P2P architecture, there is minimal (or no) reliance on dedicated servers in data centers. Instead the application exploits direct communication between pairs of intermittently connected hosts, called peers.

One of the most compelling features of P2P architectures is their **self-scalability**. For example, in a P2P file-sharing application, although each peer generates workload by requesting files, each peer also adds service capacity to the system by distributing files to other peers.

### Process Communication

Processes on two different end systems communicate with each other by exchanging **messages** across the computer network. Processes communicating with each other reside in the application layer of the five-layer protocol stack.

**Client and Server Processes**

When there’s no confusion, we’ll sometimes also use the terminology “client side and server side of an application".**An applicatio has client and server side.**

 **The Interface Between the Process and the Computer Network**

A process sends messages into, and receives messages from, the network through a software interface called a **socket**. A socket is like a door!

![Screenshot 2023-04-26 at 6.00.20 PM](https://p.ipic.vip/jk95ql.png)

A socket is the interface between the application layer and the transport layer within a host. It is also referred to as the **Application Programming Interface (API)** between the application and the network, since the socket is the programming interface with which network applications are built.

**Addressing Processes**

To identify the receiving process, two pieces of information need to be specified: 

* the address of the host (IP address)
* an identifier that specifies the receiving process in the destination host (port number)

### Transport Services Available to Applications

Four dimensions of services a transport-layer protocol can offer to applications:

* reliable data transfer
* throughput
* timing
* security

**Reliable data transfer**: Some multimedia applications are **loss-tolerant applications**.

**Throughout**: a guaranteed throughput of r bits/sec. Applications that have throughput requirements are said to be **bandwidth-sensitive applications**. WeChat video call.

**Timing**: An example guarantee might be that every bit that the sender pumps into the socket arrives at the receiver’s socket no more than 100 msec later.

### Transport Services Provided by the Internet

 The Internet (and, more generally, TCP/IP networks) makes two transport protocols available to applications, UDP and TCP. When you (as an application developer) create a new network application for the Internet, one of the first decisions you have to make is whether to use UDP or TCP. Each of these protocols offers a different set of services to the invoking applications.

**TCP Services**

The TCP service model includes a connection-oriented service and a reliable data transfer service.

* Connection-oriented services

  Handshaking $$\rightarrow$$ TCP connection

  The connection is a full-duplex connection in that the two processes can send messages to each other over the connection at the same time. When the application finishes sending messages, it must tear down the connection.

* Reliable data transfer

  No missing or duplicate bytes

TCP congestion control also attempts to limit each TCP connection to its fair share of network bandwidth.

**UDP Services**

UDP is a no-frills, lightweight transport protocol, providing minimal services. Connectionless, no handshaking. No congestion.

Today’s Internet can often provide satisfactory service to time-sensitive applications, but it cannot provide any timing or throughput guarantees.

Internet telephony applications usually prefer to run their applications over UDP, thereby circumventing TCP’s congestion control mechanism and packet overheads. But because many firewalls are configured to block (most types of) UDP traffic, TCP serves as a backup if UDP communication fails.

![Screenshot 2023-04-26 at 6.40.01 PM](https://p.ipic.vip/xpkp3e.png)

### Application-Layer Protocols

RFC vs Proprietary application protocols

