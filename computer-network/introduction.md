# What is Internet

ARPANET introduced the concepts of **resource subnet** and **communication subnet**. Resource subnet is the edge of the network. Communication subnet is **Access Network**+ **Core Network**. Core network is comprised of many ISPs which provides **Internet accessing service**. Access network connects resource subnet to Internet.

In modern Internet, there are billions of connected computing **devices**. The end systems are called the **hosts**. They run network apps at Internet's "edge".

There are **packet switches** forwarding packets(chunks of data), which makes the Internet really connected. 

There are routers and switches that send and forward packets.

There are **communication links** like fibers, coppers, radios, satellites, etc.

The **network**s are collections of devices, routers and links.

<img src="https://p.ipic.vip/3n6xef.png" alt="Screenshot 2023-04-23 at 12.06.53 PM" style="zoom:50%;" />

The network can be classified into the following categories:

* **Personal Area Network** (PAN): Bluetooth 
* **Local Area Network** (LAN): WiFi, Ethernet 
* **Metropolitan Area Network** (MAN): Wired communication, DSL, WiMAX, etc. 
* **Wide Area Network** (WAN): Major ISPs
* **Global Area Network** (GAN): Internet

Internet is the "network of networks". ***Internet* is an instance of *internet*.**

# The Network Edge

## Access Networks

Access network is the network that physically connects an end system to the first router on a path from the end system to any other distant end system.

There are 3 types of access network:

* Residential access network
* Institutional access network
* Mobile access network

## **Residential access networks: cable-based access**

<img src="https://p.ipic.vip/38gpyq.png" alt="Screenshot 2023-04-23 at 12.25.44 PM" style="zoom:50%;" />

**Frequency division multiplexing (FDM)**: different channels transmitted in different frequency bands

Cable-based access networks are typically asymmetric: up to 40 Mbps - 1.2 Gas downstream transmission rate, 30-100 Mbps upstream transmission rate.

Network of cable, fiber attaches homes to ISP router

Homes share access network to cable headend.

**Residential access networks: digital subscriber line (DSL)**

Using existing telephone line to central office DSLAM

Also asymmetric. 

<img src="https://p.ipic.vip/de57nh.png" alt="Screenshot 2023-04-23 at 1.09.46 PM" style="zoom:50%;" />

**Residential access networks: home picture**

<img src="https://p.ipic.vip/f2tgua.png" alt="Screenshot 2023-04-23 at 1.09.17 PM" style="zoom:50%;" />

**Wireless access networks**

* Wireless Local Area Networks (WLAN)
* Wide-area Cellular Access Networks

**Host: sends packets of data**

Host sending function

* Takes application message
* Break into smaller chunks, known as packets, of length L bits
* Tranmits packet into access network at **transmission rate R**
* Link transmission rate, aka link **capacity**, aka **line bandwidth**

# Switching Technology

**Node**: devices participating in the network.

**One-hop link**: connect ***adjacent*** nodes with transmission media.

<img src="https://p.ipic.vip/4tkgwc.png" alt="Screenshot 2023-09-10 at 1.40.15 AM" style="zoom:50%;" />

* **Point-to-point link** (点到点电路): There is a node at each end of the link.
* **Point-to-multipoint link** (点到多点电路): One end of the link is a single node, while the other end connects to multiple nodes.

**Switching**: in multi-hop transmission, how to allocate link resource to forward information? Fully connected graph is not feaisble. Two approaches are raised:

* **Circuit Switching** （电路交换）
* **Group Switching**（分组交换）

## Circuit Switching

The actual lines between nodes are called **physical circuit**s.

Physical circuits can be devided into multiple **logical circuit**s with **multiplexing**.

* Frequency Division Multiplexing (频分多路复用)

* Time Division Multiplexing (时分多路复用)

Circuit switching adopts a connection-oriented approach to achieve end-to-end reserved resources. **Once the connection is established, the transmission of information is guranteed.**

The drawbacks of this approach are that the circuit is occupied exclusively and the failures of the equipments may interrupt the transmission. Also, this design can not handle some burst loads.

## Packet Switching

The content to be sent is called **packet**（分组或包）. The original content is called **message**（报文）or **datagram**（数据报） sometimes.

**Store and forward**: the packet is received from the input port and processed. Then the packets are forwarded to the next hop.

**Message switching**: the message from the sender is not allowed to be further devided.

### Router and Forward

路由与转发

**Statistical Time Division Multiplexing**: 统计时分多路复用，分组按需共享信道

**Router**: global operation. Control panel. It decides the router licenses and algotihms.

**Forward**: local operation. Data panel. It sends the packets received to the next hop.

# Performance Metrics

**Data Rate**: The data rate, also known as bit rate, is the amount of data that can be transmitted over a network in a given period of time. It is usually measured in **bits per second (bps)** or **bytes per second (Bps)**. Note that when using K, M, G, T for packet lengths, the default is binary. The majority of network textbooks still use this descriptive method.
$$
\text{KiB} = \text{Kilo Binary Byte} \\
\text{KB} = \text{Kilo Byte}
$$

$$
\text{Data Rate} = \frac{\text{Number of bits transmitted}}{\text{Time taken to transmit those bits}}
$$

**Bandwidth**: The frequency bandwidth of signals allowed to pass through a link is called the bandwidth of the link. It is measured in $\text{Hz}$. Bandwidth refers to the maximum data transfer rate of a network or internet connection. It indicates the capacity of the connection to transmit data.

- It is often confused with throughput, but it refers to the maximum possible data transfer rate, not the actual rate.

**Throughput**: Throughput is the actual data rate that is achieved during data transmission. It is usually less than or equal to the bandwidth.

It can be calculated as:
$$
\text{Throughput} = \frac{\text{Total data transmitted}}{\text{Total time taken to transmit the data}}
$$
**Utilization**: Utilization is calculated as:
$$
\text{Utilization} = \left( \frac{\text{Useful data transmitted + Header + ...}}{\text{Bandwidth} \times \text{Time}} \right) \times 100\%
$$
**Efficiency**:  Efficiency refers to the proportion of the network resources that are effectively used for transmitting the actual data, excluding the overhead and retransmissions.

It can be calculated as:
$$
\text{Effective Utilization} = \left( \frac{\text{Useful data transmitted}}{\text{Bandwidth} \times \text{Time}} \right) \times 100\% = \frac{\text{Throughput}}{\text{Link Rate}} \times 100 \%
$$
**Delay/Latency**: 
$$
\text{d}_{\text{nodal}} = \text{d}_{\text{proc}}+\text{d}_{\text{queue}}+\text{d}_{\text{trans}}+\text{d}_{\text{prop}}
$$

* Processing delay
* Queueing delay
* Transmission delay
* Propogation delay

<img src="https://p.ipic.vip/nuw400.png" alt="Screenshot 2023-09-11 at 11.01.25 AM" style="zoom:50%;" />

**RTT (Round-Trip Time)**: RTT is the time taken for a data packet to travel from the source to the destination and back again.

**BDP (Bandwidth-Delay Product)**: BDP is the product of the bandwidth of a network link and the round-trip time of that link. It is the data temporarily in the link. It can be used to decide the window size of the sender.

It can be calculated as:
$$
\text{BDP} = \text{Bandwidth} \times \text{Delay}
$$

# Network Architecture

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

# Network Model

## Encapsulation and Decapsulation

An interface defines the primitives that a lower layer can provide to the upper layer. The lower layer provides services to the upper layer through a **service access point**. The data exchanged at the interface is called a **service data unit**. Equivalent entities exchange **protocol data units**.

Each layer receives an SDU from the upper layer, encapsulates it into a new PDU, and sends the new PDU to an even lower layer.

<img src="https://p.ipic.vip/8c4lw7.png" style="zoom:50%;" />

* **Application PDU:** Message (消息或报文)

* **Transport PDU: ** (TCP)Segment/(UDP)Datagram (段/数据报)

* **Network PDU:** Datagram/Packet (数据报/分组)

* **Link PDU:** Frame (帧)

* **Physical PDU:**  bit (比特)

## OSI model

**Physical Layer**: How to transmit bits

**Data Link Layer**: How to transmit frames between adjacent nodes. The data link layer combines bits into frames. It provides error detection and (optional) error control. It also supports flow control.

These two layers can be packed as subnet layer in TCP/IP.

**Network Layer**: (IP) How to transmit (or route) packets from a source node through multiple hops to a destination node. It supports congestion control, QoS control. IP address is used to uniquely identify nodes in the network.

It can be called **Internet Layer** in TCP/IP.

**Transport Layer**: How to send data between end systems. 

**Session Layer**: How to connect and organize streams of packets. Like RPC and SOCKET.

**Presentation Layer**: Information representation, security, etc.

**Application Layer**: How to implement a certain type of network application

For TCP/IP, session layer and presentation  layer is dropped but can be implemented within application layer.
