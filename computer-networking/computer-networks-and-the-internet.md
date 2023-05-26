# Computer Networks and the Internet

## What is Internet

### A Nuts-and-Bolts Description

There are billions of connected computing **devices**:

The end systems are called the **hosts**. They run network apps at Internet's "edge".

There are **packet switches** forwarding packets(chunks of data), which makes the Internet really connected. 

* routers
* switches

 There are **communication links** like fibers, coppers, radios, satellites, etc.

The **network**s are collections of devices, routers and links.

<img src="https://p.ipic.vip/3n6xef.png" alt="Screenshot 2023-04-23 at 12.06.53 PM" style="zoom:50%;" />

Internet is the "network of networks".

**Protocols** are everywhere. They control the sending and receiving of messages.

### A Service Description

 The Internet is the infrastructure that provides services to applications. It provides **programming interface** to distributed applications.

### What's a Protocol

Protocols define the **format**, **order** of **messages sent and received** among network entities, and **actions taken** on msg transmission, receipt.

## The Network Edge

### Access Networks

Access network is the network that physically connects an end system to the first router on a path from the end system to any other distant end system.

There are 3 types of access network:

* Residential access network
* Institutional access network
* Mobile access network

**Residential access networks: cable-based access**

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

* Wireless local area networks (WLAN)
* Wide-area cellular access networks

**Host: sends packets of data**

Host sending function

* Takes application message
* Break into smaller chunks, known as packets, of length L bits
* Tranmits packet into access network at **transmission rate R**
* Link transmission rate, aka link **capacity**, aka **line bandwidth**

### Physical Media

**Guided media**

Signals propagate in solid media: copper, fiber, coax

**Unguided media**

Signals propagate freely: radio

**Twisted pair (TP)**

Two insulated copper wires<img src="https://p.ipic.vip/epem0d.png" alt="Screenshot 2023-04-23 at 1.20.36 PM" style="zoom:25%;" />

**Coaxial cable**

Two concentric copper conductors<img src="https://p.ipic.vip/dytkhp.png" alt="Screenshot 2023-04-23 at 1.22.15 PM" style="zoom:25%;" />

Bidirectional

**Fiber optic cable**

<img src="https://p.ipic.vip/huzhk1.png" alt="Screenshot 2023-04-23 at 1.23.14 PM" style="zoom:25%;" />

**Wireless radio**

* Wireless LAN (WiFi)

* Wide-area (4G cellular)
* Bluetooth
* ...

## The Network Core

The network core is a mesh of packet switches and links that interconnects the Internet's end systems.

### Packet Switching

**Packet-switching**: hosts break application-layer messages into packets.

Between source and destination, each packet travels through communication links and packet switches (routers and link-layer switches). 

Most packets switches use store

## Network Architecture

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

The two predominant architectural paradigms used in modern network applications: the **client-server** architecture or the **peer-to-peer (P2P)** architecture.
