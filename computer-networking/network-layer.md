# Data Plane

## Overview

Two important network-layer functions:

* **Forwarding** implemented in the data plane.

* **Routing** implemented in the control plane.

**Forwarding** refers to the router-local action of transferring a packet from an input link interface to the appropriate output link interface. Forwarding takes place at very short timescales (typically a few nanoseconds), and thus is typically implemented in hardware. 

**Routing** refers to the network-wide process that determines the end-to-end paths that packets take from source to destination. Routing takes place on much longer timescales (typically seconds), and as we will see is often implemented in software.

The routing algorithm function in one router communicates with the routing algorithm function in other routers to compute the values for its forwarding table.

**Connection-oriented packet forwarding**

The concept of virtual circuit number:

<img src="https://p.ipic.vip/5m9vm9.png" alt="Screenshot 2023-06-11 at 9.30.08 PM" style="zoom:50%;" />

### Two Approaches of Determining Forward Table

The traditional approach:

<img src="https://p.ipic.vip/dk4vkc.png" alt="Screenshot 2023-06-03 at 1.11.12 AM" style="zoom: 25%;" />

The SDN approach:

<img src="https://p.ipic.vip/5awbi5.png" alt="Screenshot 2023-06-03 at 1.11.32 AM" style="zoom:25%;" />

The routing device performs forwarding only, while the **remote controller** computes and distributes forwarding tables.

The control-plane approach shown in the figure above is at the heart of **software-defined networking (SDN)**. The controller that computes forwarding tables and interacts with routers is implemented in software.

### Network Service Model

**Guaranteed delivery.** This service guarantees that a packet sent by a source host will eventually arrive at the destination host.

**Guaranteed delivery with bounded delay.** This service not only guarantees delivery of the packet, but delivery within a specified host-to-host delay bound (for example, within 100 msec).

**In-order packet delivery.** This service guarantees that packets arrive at the destination in the order that they were sent.

**Guaranteed minimal bandwidth.** This network-layer service emulates the behavior of a transmission link of a specified bit rate (for example, 1 Mbps) between sending and receiving hosts.

As long as the sending host transmits bits (as part of packets) at a rate below the specified bit rate, then all packets are eventually delivered to the destination host.

**Security.** The network layer could encrypt all datagrams at the source and decrypt them at the destination, thereby providing confidentiality to all transport-layer segments.

## What's Inside a Router

<img src="https://p.ipic.vip/mxyjz2.png" alt="Screenshot 2023-06-03 at 1.25.24 AM" style="zoom:50%;" />

**Input ports** 

Performs the physical layer function of terminating an incoming physical link at a router -- the leftmost box.

An input port also performs link-layer functions needed to interoperate with the link layer at the other side of the incoming link; this is represented by the middle boxes in the input and output ports. 

A lookup function is also performed at the input port; this will occur in the rightmost box of the input port.

The term “port” here refers to the physical input and output router interfaces.

**Switching fabric**

The switching fabric connects the router’s input ports to its output ports. This

switching fabric is completely contained within the router—a network inside of a network router!

**Output ports**

An **output port** stores packets received from the switching fabric and transmits these packets on the outgoing link by performing the necessary link-layer and physical-layer functions.

The routing processor performs control-plane functions. In traditional routers, it executes the routing protocols, maintains routing tables and attached link state information, and computes the forwarding table for the router. In SDN routers, the routing processor is responsible for communicating with the remote controller in order to (among other activities) receive forwarding table entries computed by the remote controller, and install these entries in the router’s input ports.

* Destination-based forwarding: deciding the output port by the destination
* Generalized forwarding: deciding the output port by multiple factors

### Input Port Processing and Destination-Based Forwarding

The forwarding table entries are often aggregated into ranges.

![Screenshot 2023-06-03 at 10.08.34 PM](https://p.ipic.vip/mypsdb.png)

**Longest prefix matching**:

When looking for forwarding table entry for given destination address, use longest address prefix that matches destination address.

![Screenshot 2023-06-03 at 10.13.00 PM](https://p.ipic.vip/nd6voa.png)

This mechanism is often performed using ternary content addressable memories (TCAMs) in hardware. We can retrieve address in one clock cycle, regardless of table size.

### Switching

Transfer packet from input link to appropriate output link.

The switching rate is the rate at which packets can be transfer from inputs to outputs.

![Screenshot 2023-06-03 at 10.21.22 PM](https://p.ipic.vip/jk7dq3.png)

Three types of switching fabric:

![Screenshot 2023-06-03 at 10.21.50 PM](https://p.ipic.vip/yqt18q.png)

**Switching via memory**

They are the first generation routers. The packet are copied into system's memory and the speed is limited by the memory bandwidth.

**Switching via a bus**

Datagram from input port memory to output port memory via a shared bus. The switching speed is limited by bus bandwidth.

**Switching via interconnection network**

<img src="https://p.ipic.vip/te4e10.png" alt="Screenshot 2023-06-03 at 10.28.26 PM" style="zoom:25%;" />

Crossbar, Clos networks.

We can exploit parallelism in this scenario. We fragment datagram into fixed length cells on entry. We switch the cells through the fabric and reassemble datagrams at exit.

Scaling

![Screenshot 2023-06-03 at 10.31.42 PM](https://p.ipic.vip/ab3zzx.png)

### Queuing

If the switch fabric slower than input ports combined, queuing may occur at input queues. 

Output port contention: only one red datagram can be transferred. 

<img src="https://p.ipic.vip/8m4tpx.png" alt="Screenshot 2023-06-03 at 10.44.19 PM" style="zoom:50%;" />

**Head-of-the-Line (HOL) blocking** queued datagram at front of queue prevents others in queue from moving forward.

<img src="https://p.ipic.vip/jdq5wf.png" alt="Screenshot 2023-06-03 at 10.45.52 PM" style="zoom:50%;" />

**Output port queuing** Buffering required when datagrams arrive from fabric faster than link transmission rate.

**Drop policy** which datagrams to drop if no free buffers? Datagrams can be lost due to congestion, lack of buffers.

**Scheduling discipline** chooses among queued datagrams for transmission. 

But too much buffering can increase delays (particularly in home routers)

### Buffer Management

Drop policy:

* Tail drop: drop arriving packet
* Priority: drop/remove on priority basis

Marking: which packets to mark to signal congestion (ECN, RED)

### Packet Scheduling

* FCFS/FIFO

* Priority

  Arriving traffic classified, queued by class.

  Any header fields can be used for classification.

  FCFS within priority class

* Rounding Robin scheduling

* Weighted Fair Queuing (WFQ)

  Generalized Round Robin

  Each class has weight and gets weighted amount of service in each cycle:

  $\frac{W_i}{\sum_{j}W_j}$

  Minimum bandwidth guarantee (per-traffic-class)

 ## The Internet Protocol

### IPv4 Datagram Format

<img src="https://p.ipic.vip/sdibg6.png" alt="Screenshot 2023-06-04 at 12.10.55 AM" style="zoom:50%;" />

The maximum length of an IP datagram is 64K bytes, but typically the Datagram is 1500 bytes or less.

The **"type" of service** attribute: 

* diffserv(0:5) (different services) 
* ECN(6:7) (congestion control)

The **TTL** attribute: denoting the remaining max hops; decremented at each router

The upper layer protocol indicates the adopted protocol of the payload data.

### IPv4 Addressing

IP address is a 32-bit identifier associated with each host or router interface.

Interface is connection between host/router and physical link. Routers typically have mutiple interfaces. A host typically has one or two interfaces (a laptop has wired Ethernet and wireless 802.11).

<img src="https://p.ipic.vip/tmf4yp.png" alt="Screenshot 2023-06-04 at 12.20.56 AM" style="zoom:50%;" />

**Subnet** is device interfaces that can physically reach each other without passing through an intervening router.

IP address have structure:

**subnet part** devices in same subnet have common high order bits. 

**host part** is the remaining low order bits.

Let's assume that the **subnet mask** is the high-order 24 bits.

<img src="https://p.ipic.vip/w9lm85.png" alt="Screenshot 2023-06-04 at 12.29.19 AM" style="zoom:50%;" />

**CIDR**: Classless InterDomain Routing

Subnet portion of address of arbitrary length. The address format is **a.b.c.d/x**, where x is the number of bits in subnet portion of address

**How does *host* get IP address?**

The IP address is hard-coded by sysadmin in config file (e.g., /etc/rc.config in UNIX)

**DHCP** means Dynamic Host Configuration Protocol. It is used to dynamically get address from as server.

<img src="https://p.ipic.vip/1ey4mu.png" alt="Screenshot 2023-06-04 at 12.38.50 AM" style="zoom:50%;" />

Typically, DHCP server will be co-located in router, serving all subnets to which router is attached.

![Screenshot 2023-06-04 at 12.43.30 AM](https://p.ipic.vip/pnc8o5.png)

The first two steps are optional.

The IP address 255.255.255.255 is a special broadcast address in IPv4 networking.

Ports 67 and 68 are used for the DHCP (Dynamic Host Configuration Protocol) service.

- Port 67 is used by the DHCP server. This is the port on which the server listens for client requests.
- Port 68 is used by the DHCP client. This is the port on which the client listens for responses from the server.

DHCP can return more than just allocated IP address on subnet:

* Address of first-hop router for client
* Name and IP address of DNS server
* Network mask (indicating network versus host portion of address)

The subnet gets allocated portion of its provider ISP's address space.

![Screenshot 2023-06-04 at 12.52.33 AM](https://p.ipic.vip/ec8t3i.png)

What if an organization from one ISP switches to another ISP? We need more specific routes.

![Screenshot 2023-06-04 at 12.56.34 AM](https://p.ipic.vip/qtoshu.png)

**How does an ISP get block of addresses?**

**ICANN**: Internet Corporation for Assigned Names and Numbers. ICANN allocates IP addresses, through 5 regional registries (RRs) (who may then allocate to local registries)

### NAT

All devices in local network share just one IPv4 address as far as outside world is concerned.

All devices in local network have 32-bit addresses in a "private" IP address space (10/8, 172.16/12, 192.168/16) that can only be used in local network.

![Screenshot 2023-06-04 at 1.43.23 AM](https://p.ipic.vip/4siswu.png)

**Implementation** NAT router must (transparently):

* **Outgoing datagrams: replace** (source IP address, port #) of every outgoing datagram to (NAT IP address, new port #)
* **Remember (in NAT translation table)** every (source IP address, port #) to (NAT IP address, new port #) translation pair
* **Incoming datagrams: replace** (NAT IP address, new port #) in destination fields of every incoming datagram with corresponding (source IP address, port #) stored in NAT table

### IPv6 Datagram Format

![Screenshot 2023-06-04 at 1.55.27 AM](https://p.ipic.vip/ir1tp2.png)

IPv6 provides mechanism but not policy for low handling. The policy issue is up to ISPs. The priority field is used to identify priority among datagrams in flow.

What's missing compared with IPv4: there's no checksum (to speed processing at routers) there's no fragmentation/reassembly there's no options (available as upper-layer, next-header protocol at router)

**Transition from IPv4 to IPv6** 

**Tunneling**: IPv6 datagram carried as payload in IPv4 datagram among IPv4 routers ("packet within a packet") Tunneling is used extensively in other contexts.

![Screenshot 2023-06-04 at 2.02.36 AM](https://p.ipic.vip/hbv12f.png)

![Screenshot 2023-06-04 at 2.06.05 AM](https://p.ipic.vip/nxdxr2.png)

## Generalized Forwarding

Many header fields can determine action

Many action possible: drop/copy/modify/log packet

### Flow Table Abstraction

**Flow** is defined by header fields. A flow is a sequence of packets from a source to a destination that the network treats as a single unit because they share certain characteristics. 

**Generalized forwarding:** **simple** packet-handling rules

* Match: pattern values in packet header fields

* Actions: for matched packet: drop, forward, modify, send to controller
* Priority: disambiguate overlapping patterns
* Counters: #bytes and #packets

OpenFlow

![Screenshot 2023-06-04 at 2.42.20 AM](https://p.ipic.vip/3pbrdb.png)

![Screenshot 2023-06-04 at 2.43.21 AM](https://p.ipic.vip/a2il3m.png)

Orchestrated tables can create **network-wide** behavior, e.g.,:

> Datagrams from host h5 and h6 should be sent to h3 or h4 via s1 and from there to s2

## Middleboxes

Any intermediary box performing functions apart from normal, standard functions of an IP router on the data path between a source host and destination host.

Examples of Middleboxes:

![Screenshot 2023-06-04 at 2.59.18 AM](https://p.ipic.vip/glzl2m.png)

Initially the Middleboxes are proprietary (closed) hardware solutions. Recently, they are moving towards "whitebox" hardware implementing open API. They have programmable local actions via match+action. 

### The IP hourglass

![Screenshot 2023-06-04 at 3.04.15 AM](https://p.ipic.vip/jcqykr.png)

But recently, the IP hourglass is at its middle age: <img src="https://p.ipic.vip/ed7o9u.png" alt="Screenshot 2023-06-04 at 3.05.08 AM" style="zoom:25%;" />

![Screenshot 2023-06-04 at 3.11.26 AM](https://p.ipic.vip/msk530.png)

The intelligence can reside in the hosts as well as middleboxes and data centers in modern age.

# Control Plane

**Forwarding**: determine route taken by packets from source to destination (local function)

**Routing**: move packets from router's input to appropriate router output (global function)

**Per-router control plane** Individual routing algorithm components in each and every router in the control plane 

**Software-Defined Networking(SDN) control plane** Remote controller computes, installs forwarding tables in routers

## Routing Algorithms

Routing algorithm goal is to determine "good" paths (equivalently, routes), from sending hosts to receiving host, through network of routers.

**Graph Abstraction**

<img src="https://p.ipic.vip/2vlkuv.png" alt="Screenshot 2023-06-04 at 4.21.54 PM" style="zoom:50%;" />

**Global Algorithm**: all routers have complete, link cost info. They are often called "link state" algorithms.

**Decentralized Algorithm**: iterative process of computation, exchange of info with neighbors. Routers initally only know costs to attached neighbors.

**Dynamic Algorithm**: routes change more quickly. Periodic updates.

**Static Algorithm**: routes change slowly over time.

### Dijkstra's Link-state Routing Algorithm

Every node dsitribute their link state information throughout the network.

A centralized algorithm: network topology and link costs are known to all nodes.

The centralized feature is accomplished via "link state broadcast" and all nodes have same info.

It has the iterative feature that after k iterations, we can know least cost path to k destinations.

> **Notation**
>
> $C_{a,b}$: from node a to b
>
> $D(a)$: current estimate of cost of least-cost-path from source to destination $a$
>
> $p(a)$: the predecessor node along path from source to $a$
>
> $N^{\prime}$: set of nodes whose least-cost-path definitely known

![Screenshot 2023-06-04 at 4.39.21 PM](https://p.ipic.vip/323n69.png)

<img src="https://p.ipic.vip/dc4oda.png" alt="Screenshot 2023-06-04 at 4.43.49 PM" style="zoom:50%;" />

We can derive the forwarding table based on the result.

 **Algorithm Complexity**

In each of the $n$ iteration, we need to check all nodes that is not in $N^{\prime}$. There's a total of $\frac{n(n+1)}{2}$ comparisons. Therefore, the time complexity is $O(n^2)$. With a more efficient implementation, the complexity can be reduced to $O(n\log{n})$

**Message Complexity**

Each router must broadcast its link state information to other $n$ routers

Efficient broadcast algorithms require $O(n)$ link crossings to disseminate a broadcast message from one source.

Each router's message crosses $O(n)$ links: overall message complexity: $O(n^2)$

**Possible Oscillations**

When link costs depend on traffic volume, route oscillations is possible.

Routing to destination a, traffic entering at d, c, e with rates 1, e, 1.

![Screenshot 2023-06-04 at 5.13.50 PM](https://p.ipic.vip/tcyhrf.png)

### Distance Vector

Based on Bellman-Ford equation (dynamic programming):

> Bellman-Ford Equation
>
> Let $D_x(y)$ : cost of least-cost path from $x$ to $y$
>
> Then:
> $$
> D_x(y) = \min_{v} \{c_{x,v}+D_v(y)\}
> $$

The key idea is that from time to time, each node sends its own distance vector estimate to neighbors.

When x receives new DV estimate from any neighbor, it updates its own DV using B-F equation.

For each node, there're three steps:

* **wait** for (change in local link cost or msg from neighbor)
* **recompute** my DV estimates using DV received from neighbor
* If my DV to any destination has changed, **send my new DV** to my neighbors, else do nothing.

The algorithm is **iterative** and **asynchronous**. Each local iteration is caused by: local link cost change or DV update message from neighbor

The algorithm is **distributed** and **self-stopping**. Each node notifies neighbors only when its DV changes. Neighbors notify their neighbors only if necessary. If there's no notification received, then no actions should be taken.

If there're link cost changes, the node will detect the local link cost change and updates its routing information. The local DV is recalculated and the neighbors are notified.

**Good news travels fast**:

<img src="https://p.ipic.vip/s3apfb.png" alt="Screenshot 2023-06-05 at 12.51.17 AM" style="zoom:50%;" />

![Screenshot 2023-06-05 at 12.51.42 AM](https://p.ipic.vip/sjbd3h.png)

The cost goes down and the updates travels fast.

**Bad news travels slow**: count-to-infinity problem

### Comparison of LS and DV Algorithms

**Complexity**: 

* LS: n routers, $O(n^2)$ messages sent
* DV: exchange between neighbors; convergence time varies

 **Robustness**:

What happens if router malfunctions, or is compromised?

* LS: router can advertise incorrect **link** cost. Each router computes only its own table.

* DV: router can advertise incorrect path cost ("I have a really low-cost path to everywhere"): **black-holing** 

  Each router's DV is used by others and error propagates through the network.

## OSPF

**Scale issue**: there are billions of destinations and we can't store all destinations in routing tables. This will cause the routing table exchange to flood the bandwidth.

**Administrative autonomy**: the internet is a network of networks and each network admin may want to control routing in its own network.

### Internet Approach towards Scalable Routing

We aggregate the routers into regions known as "autonomous systems" (AS) (a.k.a "domains")

**Intra-AS (a.k.a "intra-domain")**  routing among routers within same AS ("network") 

* All routers in AS must run same intra-domain protocol
* Routers in different AS can run different intra-domain routing protocols
* **Gateway router**: at "edge" of its own AS, has link to routers in other AS'es

**Inter-AS (a.k.a "inter-domain")** routing among AS'es

* Gateways performs inter-domain routing (as well as intra-domain routing)

### Intra-AS routing: routing with an AS

Most common intra-AS routing protocols:

* **RIP** Routing Information Protocol

  Classic DV: DVs exchanged every 30 secs

* **OSPF** Open Shortest Path First

  Classic link-state routing

  Each router floods OSPF link-state advertisements (directly over IP rather than using TCP/UDP) to all other routers in entire AS

  Each router has full topology, uses Dijkstra's algorithm to compute forwarding table

  Multiple link costs metrics possible: bandwidth, delay

  **Security** all OSPF messages authenticated (to prevent malicious intrusion)

  **Hierarchical OSPF** Two-level hierarchy: local area and backbone

  Link-state advertisements flooded only in area, or backbone

  Each node has detailed area topology

  The **area border routers** summarize distances to destinations in own area and advertise in backbone.

  ![Screenshot 2023-06-05 at 2.13.00 AM](https://p.ipic.vip/2zsybt.png)

* **EIGRP** Enhanced Interior Gateway Routing Protocol

  DV based

  Became open in 2013

## BGP

BGP is the de facto inter-domain routing protocol. It is said to be the glue that holds the Internet together.

BGP allows the subnet to advertise its existence and the destinations it can reach to the rest of Internet: "I am here, here is who I can reach, and how"

BGP provides each AS a means to obtain destination network reachability info from neighboring ASes (**eBGP**)

BGP can determine routes to other networks based on reachability information and policy (like avoid certain countries or ISPs)

BGP propagate reachability information to all AS-internal routers (**iBGP**)

BGP advertise (to neighboring networks) destination reachability information

Gateway routers run both eBGp and iBGP protocols.

### BGP basics

**BGP session**: two BGP routers ("peers", "speakers") exchange BGP messages over semi-permanent TCP connection.

BGP advertising paths to different destinations network prefixes (e.g., to a destination/16 network). Therefore, BGP is called a "path vector" protocol.

![Screenshot 2023-06-12 at 8.47.49 PM](https://p.ipic.vip/wrdkx9.png)

When AS3 gateway 3a advertises **path AS3, X** to AS2 gateway 2c, AS3 promises to AS2 it will forward datagrams towards X.

### BGP protocol messages

BGP messages exchanged between peers over TCP connection.

BGP messages:

**OPEN**: opens TCP connection to remote BGP peer and authenticates sending BGP peer.

**UPDATE**: advertises new path (or withdraw old)

**KEEPALIVE**: keeps connection alive in absence of UPDATES; also ACKs OPEN request

**NOTIFICATION**: reports errors in previous msg; also used to close connection

### Path Advertisement

BGP advertised path: prefix + attributes

* Path prefix: destination being advertised

* Two important attributes:

  **AS-PATH**: list of ASes through which prefix advertisements has passed

  **NEXT_HOP**: indicates specific internal-AS router to next hop AS

BGP is policy-based:

* Routers receiving route advertisement to destination X uses policy to accept/reject a path (e.g., never route through AS W or country Y).

* Router uses policy to decide whether to advertise a path to neighboring AS Z.

![Screenshot 2023-06-12 at 8.59.52 PM](https://p.ipic.vip/d1l6dh.png)

Gateway routers may learn about multiple paths to destination.

### Routing Policy

<img src="https://p.ipic.vip/lbm5f5.png" alt="Screenshot 2023-06-12 at 9.03.45 PM" style="zoom:50%;" />

Assuming that w is not a customer of B, B doesn't advertise BAw to C. C will route Caw to get to w.

x does not want to route from B to C via x, so x will not advertise to B a route to C.

### Hot potato routing

Choose local gateway that has least intra-domain cost.

![Screenshot 2023-06-12 at 9.11.28 PM](https://p.ipic.vip/oq0ywc.png)

2d will choose to route to X via 2a.

## Internet Control Message Protocol

ICMP is used by hosts and routers to communicate network-level information.

Error reporting: unreachable host, network, port, protocol

Echo request/reply (used by ping)

ICMP messages are carried in IP datagrams, protocol number: 1

ICMP message: type, code plus header and first 8 bytes of IP datagram causing error.
