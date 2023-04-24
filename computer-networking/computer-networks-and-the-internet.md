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
