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
| $[0, base-1]$            | Sent and acknowledged         |
| $[base, nextseqnum-1]$   | Sent but not yet acknowledged |
| $[nextseqnum, base+N-1]$ | Can be sent                   |
| $[base+N, \infty]$       | Cannot be used                |

$N$ is referred to as the **window size**. GBN protocol is a **sliding-window protocol**.  

Why we need to limit the window size to $N$ ?

<img src="https://p.ipic.vip/26s1ts.png" alt="Screenshot 2023-05-26 at 6.32.17 PM" style="zoom:50%;" />

#### Sender Side

**Invocation from above** When `rdt_send()` is called from above, the sender first checks to see if the window is full, that is, whether there are *N* outstanding, unacknowledged packets. If the window is not full, a packet is created and sent, and variables are appropriately updated.

**Receipt of an ACK**  In our GBN protocol, an acknowledgment for a packet with sequence number *n* will be taken to be a **cumulative acknowledgment**, indicating that all packets with a sequence number up to and including *n* have been correctly received at the receiver.

**Timeout Event** The protocol’s name, “Go-Back-N,” is derived from the sender’s behavior in the presence of lost or overly delayed packets. As in the stop-and-wait protocol, a timer will again be used to recover from lost data or acknowledgment packets. If a timeout occurs, the sender resends *all* packets that have been previously sent but that have not yet been acknowledged. In the illustration, there's only one timer, which can be thought of as a timer for the oldest transmitted but not yet acknowledged packet. If an ACK is received but there are still additional transmitted but not yet acknowledged packets, the timer is restarted. If there are no outstanding, unacknowledged packets, the timer is stopped.

#### Receiver Side

<img src="https://p.ipic.vip/21jxyt.png" alt="Screenshot 2023-05-26 at 6.37.05 PM" style="zoom:50%;" />

If a packet with sequence number *n* is received correctly and is in order (that is, the data last delivered to the upper layer came from a packet with sequence number n−1), the receiver sends an ACK for packet *n* and delivers the data portion of the packet to the upper layer. In all other cases, the receiver discards the packet and resends an ACK for the most recently received in-order packet.

In our GBN protocol, the receiver discards out-of-order packets.

While the sender must maintain the upper and lower bounds of its window and the position of *nextseqnum* within this window, the only piece of information the receiver need maintain is the sequence number of the next in-order packet. This value is held in the variable `expectedseqnum`.

<img src="https://p.ipic.vip/cg19tc.png" alt="Screenshot 2023-05-26 at 6.41.00 PM" style="zoom:50%;" />

### Selective Repeat

The issue of GBN is that the retransmissions are wasteful.

As the name suggests, selective-repeat protocols avoid unnecessary retransmissions by having the sender retransmit only those packets that it suspects were received in error (that is, were lost or corrupted) at the receiver. 

This individual, as-needed, retransmission will require that the receiver *individually* acknowledge correctly received packets. A window size of *N* will again be used to limit the number of outstanding, unacknowledged packets in the pipeline. However, unlike GBN, the sender will have already received ACKs for some of the packets in the window.

<img src="https://p.ipic.vip/c9z3ez.png" alt="Screenshot 2023-05-26 at 7.11.14 PM" style="zoom:50%;" />

**Out-of-order packets are buffered** until any missing packets (that is, packets with lower sequence numbers) are received, at which point a batch of packets can be delivered in order to the upper layer.

The receiver reacknowledges (rather than ignores) already received packets with certain sequence numbers *below* the current window base. You should convince yourself that this reacknowledgment is indeed needed.

![Screenshot 2023-05-26 at 7.38.30 PM](https://p.ipic.vip/mjqqb5.png)

If a packet has been received by the receiver in SR, and the same packet is sent once again, the receiver would typically acknowledge it as if it was the first time the packet was received. If the sender doesn't receive the acknowledgement, the window won't move forward.

**The window size must be less than or equal to half the size of the sequence number space for SR protocols.** Otherwise, the receiver cannot tell apart a new packet and a retransmission.

![Screenshot 2023-05-26 at 7.55.09 PM](https://p.ipic.vip/a956cn.png)

## Connection-Oriented Transportation: TCP

### The TCP Connection

* Full-duplex service
* Point-to-point

Once the data passes through the socket, the data is in the hands of TCP running in the client. TCP directs this data to the connection's send buffer, which is one of the buffers that is set aisde during the initial three-way handshake. From time to time, TCP will grab chunks of data from the send buffer and pass the data to the network layer.

The maximum amount of data that can be grabbed and placed in a segment is limited by the **maximum segment size (MSS)**. The MSS is typically set by first determining the length of the largest link-layer frame that can be sent by the local sending host (the so-called **maximum transmission unit, MTU**), and then setting the MSS to ensure that a TCP segment (when encapsulated in an IP datagram) plus the TCP/IP header length (typically 40 bytes) will fit into a single link-layer frame.

TCP pairs each chunk of client data with a TCP header, thereby forming **TCP segments**. The segments are passed down to the network layer, where they are separately encapsulated within network-layer IP datagrams. The IP datagrams are then sent into the network. When TCP receives a segment at the other end, the segment’s data is placed in the TCP connection’s receive buffer.

TCP is buffer capable. TCP entity can choose to buffer prior to sending or not. Buffering reduces overhead (fewer headers), but increases delay.

![Screenshot 2023-05-26 at 8.12.09 PM](https://p.ipic.vip/wbsjve.png)

### TCP Segment Structure

<img src="https://p.ipic.vip/mmiq60.png" alt="Screenshot 2023-05-26 at 8.15.22 PM" style="zoom:50%;" />

The 4-bit **header length field** specifies the length of the TCP header in 32-bit words. The TCP header can be of variable length due to the TCP options field. (Typically, the options field is empty, so that the length of the typical TCP header is 20 bytes.)

The optional and variable-length **options field** is used when a sender and receiver negotiate the maximum segment size (MSS) or as a window scaling factor for use in high-speed networks. A timestamping option is also defined

The **flag field** contains 6 bits. 

* The **ACK bit** is used to indicate that the value carried in the acknowledgment field is valid; that is, the segment contains an acknowledgment for a segment that has been successfully received. 
* The **RST**, **SYN**, and **FIN** bits are used for connection setup and teardown. 
* The **CWR** and **ECE** bits are used in explicit congestion notification. 
* Setting the **PSH** bit indicates that the receiver should pass the data to the upper layer immediately. 
* Finally, the **URG** bit is used to indicate that there is data in this segment that the sending-side upper-layer entity has marked as “urgent.” 
* The location of the last byte of this urgent data is indicated by the 16-bit **urgent data pointer field**. TCP must inform the receiving-side upper- layer entity when urgent data exists and pass it a pointer to the end of the urgent data. (In practice, the **PSH**, **URG**, and the urgent data pointer are not used. However, we mention these fields for completeness.)

**Sequence Numbers and Acknowledgement Numbers**

TCP views data as an unstructured, but ordered, stream of **bytes**.

The sequence number is viewed upon the byte, **not segment**.

The **sequence number for a segment** is therefore the byte-stream number of the first byte in the segment. 

Each of the segments that arrive from Host B has a sequence number for the data flowing from B to A. **The acknowledgment number that Host A puts in its segment is the sequence number of the next byte Host A is expecting from Host B.**

In truth, both sides of a TCP connection randomly choose an initial sequence number. This is done to minimize the possibility that a segment that is still present in the network from an earlier, already-terminated connection between two hosts is mistaken for a valid segment in a later connection between these same two hosts (which also happen to be using the same port numbers as the old connection)

**A Case Study for Sequence and Acknowledgment Numbers**

![Screenshot 2023-05-30 at 4.40.57 PM](https://p.ipic.vip/z7v9jt.png)

The third segment intends to acknowledge the data it has received from the server. It also has a sequence number but the data is empty. But because TCP has a sequence number field, the segment needs to have some sequence number.

### Round-Trip Time Estimation and Timeout

$$
EstimatedRTT= (1-\alpha)\cdot EstimatedRTT+\alpha \cdot SampleRTT
$$

The recommended value of α is $\alpha = 0.125$.

In statistics, such an average is called an **exponential weighted moving average (EWMA)**. The word “exponential” appears in EWMA because the weight of a given *SampleRTT* decays exponentially fast as the updates proceed

n addition to having an estimate of the RTT, it is also valuable to have a measure of the variability of the RTT. **DevRTT** is an estimate of how much *SampleRTT* typically deviates from *EstimatedRTT*:
$$
DevRTT = (1-\beta) \cdot DevRTT + \beta\cdot |SampleRTT-EstimatedRTT|
$$
The recommended value of β is 0.25.

**Setting and Managing the Retransmission Timeout Interval**

The timeout should be set a little bit larger than the average estimated RTT.
$$
TimeoutInterval=EstimatedRTT+4⋅DevRTT
$$

### Reliable Data Transfer

A singe-timer practice:

![Screenshot 2023-05-30 at 5.40.48 PM](https://p.ipic.vip/63hqi2.png)

You can imagine that the timer is associated with the oldest unacknowledged segment.

A subtlety:

![Screenshot 2023-05-30 at 5.47.16 PM](https://p.ipic.vip/4i35hg.png)

If `ACK=120` is received, even if `ACK=100` is not received, the Host A doesn't need to retransmit the segment.

**Modifications**

Each time TCP retransmits, it sets the next timeout interval to twice the previous value. This modification provides a limited form of congestion control.

The receiver will send a duplicate ACK to the sender to trigger a fast retransmit.

| Event                                                        | TCP Receiver Action                                          |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Arrival of in-order segment with expected sequence number. All data up to expected sequence number already acknowledged. | Delayed ACK. Wait up to 500 msec for arrival of another in-order segment. If next in-order segment does not arrive in this interval, send an ACK. |
| Arrival of in-order segment with expected sequence number. One other in-order segment waiting for ACK transmission. | One Immediately send single cumulative ACK, ACKing both in-order segments. |
| Arrival of out-of-order segment with higher- than-expected sequence number. Gap detected. | Immediately send duplicate ACK, indicating sequence number of next expected byte (which is the lower end of the gap). |
| Arrival of segment that partially or completely fills in gap in received data. | Immediately send ACK, provided that segment starts at the lower end of gap. |

If the TCP sender receives three duplicate ACKs for the same data, it takes this as an indication that the segment following the segment that has been ACKed three times has been lost. In the case that three duplicate ACKs are received, the TCP sender performs a **fast retransmit**, retransmitting the missing segment *before* that segment’s timer expires.

<img src="https://p.ipic.vip/ev4ziu.png" alt="Screenshot 2023-05-30 at 6.09.15 PM" style="zoom:50%;" />

**The difference between TCP and GBN** Auctually the case above.

Considering the introduction of selective acknowledgement policy, TCP’s error-recovery mechanism is probably best categorized as a hybrid of GBN and SR protocols.

### Flow Control

Receiver buffer overflow.

TCP provides flow control by having the *sender* maintain a variable called the **receive window**. Informally, the receive window is used to give the sender an idea of how much free buffer space is available at the receiver.
$$
rwnd=RcvBuffer−[LastByteRcvd−LastByteRead]
$$
Host B tells Host A how much spare room it has in the connection buffer by placing its current value of *rwnd* in the receive window field of every segment it sends to A. Initially, Host B sets *rwnd = RcvBuffer*. Note that to pull this off, Host B must keep track of several connection-specific variables.

By keeping the amount of unacknowledged data less than the value of *rwnd*, Host A is assured that it is not overflowing the receive buffer at Host B.

The problem is, the rand is sent by sending data or ACK. If the buffer is full and then emptied, no message will be sent to the sender to inform it that the buffer is available again.

To solve this problem, the TCP specification requires Host A to continue to send segments with one data byte when B’s receive window is zero. These segments will be acknowledged by the receiver. Eventually the buffer will begin to empty and the acknowledgments will contain a nonzero *rwnd* value.

### TCP Connection Management

**Eatablish a TCP connection:**

* Step1: 

  SYN = 1 

  a random sequence number 

  **SYN segment**

* Step2: 

  Allocate TCP buffers and variables to the connection

  Send a connection-granted segment to the client 

  SYN = 1  

  The acknowledgment field of the TCP segment header is set to *client_isn+1*

  Choose its own sequence number

  **SYNACK** segment

* Step3:

  Allocate TCP buffers and variables to the connection

  SYN = 0

  The acknowledgment field of the TCP segment header is set to *server_isn+1*

  May carry client-to-server data in the segment payload.

In the following message exchanges, the SYN bit will be set up to zero.

**Close a TCP connection**

The client issues a close command. A special segment with FIN set to 1 is sent to the server. The client goes into FIN_WAIT_1. The ACK is received  and the client proceeds to FIN_WAIT_2. When the client receives FIN from the server, goes into TIME_WAIT and wait 30 seconds to enter CLOSED.

<img src="https://p.ipic.vip/jvkehg.png" alt="Screenshot 2023-05-30 at 7.20.22 PM" style="zoom:50%;" />

![Screenshot 2023-05-30 at 7.25.30 PM](https://p.ipic.vip/bqy60u.png)

If a illegal connection establishing segment is received by the server, a special segment with RST set to 1 is sent back to client.

## Principles of Congestion Control

* End-to-end congestion control

  TCP segment loss (as indicated by a timeout or the receipt of three duplicate acknowledgments) is taken as an indication of network congestion, and TCP decreases its window size accordingly. We’ll also see a more recent proposal for TCP congestion control that uses increasing round-trip segment delay as an indicator of increased network congestion

* Network-assisted congestion control

  With network-assisted congestion control, routers provide explicit feedback to the sender and/or receiver regarding the congestion state of the network.

For network-assisted congestion control, congestion information is typically fed back from the network to the sender in one of two ways. Direct feedback may be sent from a network router to the sender. This form of notification typically takes the form of a choke packet (essentially

saying, “I’m congested!”). The second and more common form of notification occurs when a router marks/updates a field in a packet flowing from sender to receiver to indicate congestion. Upon receipt of a marked packet, the receiver then notifies the sender of the congestion indication. This latter form of notification takes a full round-trip time.

## TCP Congestion Control

TCP must use end-to-end congestion control rather than network-assisted congestion control, since the IP layer provides no explicit feedback to the end systems regarding network congestion.

The approach taken by TCP is to have each sender limit the rate at which it sends traffic into its connection as a function of perceived network congestion.

A congestion window to limit the send rate:
$$
LastByteSent-LastByteAcked \leq min\{cwnd, rwnd\}
$$
We define a loss event at the sender side. The loss event is either a timeout ot a three duplicate ACK. If the network is way too congested, some datagrams may be discarded -- cause of loss event.

If there's no loss events, TCP will increse its congestion window size. Because TCP uses acknowledgments to trigger (or clock) its increase in congestion window size, TCP is said to be **self-clocking**. The faster the ACKs arrive, the faster the window size grows.

**A lost segment implies congestion, and hence, the TCP sender’s rate should be decreased when a segment is lost.**

**An acknowledged segment indicates that the network is delivering the sender’s segments to the receiver, and hence, the sender’s rate can be increased when an ACK arrives for a previously unacknowledged segment.** 

### Slow Start

When a TCP connection begins, the value of *cwnd* is typically initialized to a small value of 1 MSS. Thus, in the **slow-start** state, the value of *cwnd* begins at 1 MSS and increases by 1 MSS every time a transmitted segment is first acknowledged.

<img src="https://p.ipic.vip/enebed.png" alt="Screenshot 2023-05-31 at 8.51.34 PM" style="zoom: 25%;" />

The three ways in which the TCP slow start process can end are:

1. **Loss Event (Timeout):** If a loss event such as network congestion is detected, marked by a timeout, the TCP sender sets the congestion window size (*cwnd*) to 1 and initiates the slow start process from the beginning. Alongside, it also sets the slow start threshold (*ssthresh*) to *cwnd/2*, which is half the *cwnd* value when the congestion was noticed.

2. **Reaching Slow Start Threshold:** The second way slow start ends is related to the value of *ssthresh*. Since *ssthresh* is set to half the value of *cwnd* when congestion was last detected, it would be imprudent to continue doubling *cwnd* when it matches or surpasses the *ssthresh* value. Therefore, when *cwnd* equals *ssthresh*, slow start concludes, and TCP shifts into congestion avoidance mode. Don

3. **Three Duplicate ACKs:** The final scenario in which slow start can terminate is if three duplicate ACKs (acknowledgements) are detected. In this case, TCP executes a fast retransmit and enters the fast recovery state. This mechanism allows TCP to quickly respond to packet losses and recover the lost data.

### Congestion Avoidance

TCP adopts a more conservative approach and increases the value of *cwnd* by just a single MSS every RTT. The TCP sender increases the size of the congestion window (cwnd) every time it receives an acknowledgement (ACK) for a packet it has sent. **The amount by which it increases cwnd is equal to MSS divided by the current size of the cwnd.** 

TCP’s congestion-avoidance algorithm behaves the same when a timeout occurs as in the case of slow start: The value of *cwnd* is set to 1 MSS, and the value of *ssthresh* is updated to half the value of *cwnd* when the loss event occurred. Recall, however, that a loss event also can be triggered by a triple duplicate ACK event.

<img src="https://p.ipic.vip/31o3l4.png" alt="Screenshot 2023-05-31 at 9.28.44 PM" style="zoom:50%;" />

### Fast Recovery

The value of *cwnd* is increased by 1 MSS for every duplicate ACK received for the missing segment that caused TCP to enter the fast-recovery state.

Eventually, when an ACK arrives for the missing segment, TCP enters the congestion-avoidance state after deflating *cwnd*. If a timeout event occurs, fast recovery transitions to the slow-start state after performing the same actions as in slow start and congestion avoidance: The value of *cwnd* is set to 1 MSS, and the value of *ssthresh* is set to half the value of *cwnd* when the loss event occurred.

TCP congestion control is often referred to as an **additive-increase, multiplicative-decrease (AIMD)** form of congestion control.

Assuming that *RTT* and *W* are approximately constant over the duration of the connection, the TCP transmission rate ranges from $\frac{W}{2\times RTT}$ to $\frac{W}{RTT}$.

The average throughput of a connection is $\frac{3}{4}\times\frac{W}{RTT}$

TCP congestion control converges to provide an equal share of a bottleneck link’s bandwidth among competing TCP connections.

<img src="https://p.ipic.vip/8przzv.png" alt="Screenshot 2023-06-02 at 7.37.43 PM" style="zoom:50%;" />

A to B to C ... converging to the intersection.

### Explicit Congestion Notification

At the network layer, two bits (with four possible values, overall) in the Type of Service field of the IP datagram header are used for ECN.

<img src="https://p.ipic.vip/osvvf6.png" alt="Screenshot 2023-06-02 at 7.46.11 PM" style="zoom:50%;" />

The TCP sender, in turn, reacts to an ACK with an ECE congestion indication by halving the congestion window, as it would react to a lost segment using fast retransmit, and sets the CWR (Congestion Window Reduced) bit in the header of the next transmitted TCP sender-to-receiver segment.

## The Sockets Interface

<img src="https://p.ipic.vip/jstaky.png" alt="Screenshot 2023-05-03 at 12.50.20 PM" style="zoom:50%;" />

From the perspective of the Linux kernel, a socket is an end point for communication. From the perspective of a Linux program, a socket is an open file with a corresponding descriptor.

Internet socket addresses are stored in 16-byte structures having the type `sockaddr_in`. 

```c
/* IP socket address structure */
struct sockaddr_in  {
	uint16_t        sin_family;  /* Protocol family*/
  uint16_t        sin_port;    /* Port number in network byte order */
  struct in_addr  sin_addr;    /* IP address in network byte order */
  unsigned char   sin_zero[8]; /* Pad to sizeof(struct sockaddr) */
};
```

`sin_addr` is a 32-bit address. The IP address and port number are always stored in network byte order.

The `connect`, `bind`, and `accept` functions require a pointer to a protocol-specific socket address structure.

In old days, the sockets functions expect a pointer to a generic `sockaddr` structure.

```c
struct sockaddr {
	uint16_t  sa_family;    /* Protocol family */
	char      sa_data[14];  /* Address data  */
};
```

Pointers pointing to protocol-specific structures should be cast to `sockaddr`.

Now, we use a generic `void *` pointer.

Literally we setup a server through the following procedures:

* We invoke `getaddrinfo(const char *host, const char *service, const struct addrinfo *hints, struct addrinfo **result)` to obtain a list of `addrinfo` structures that represent the possible network addresses we can use.

  For a server, the `host` is set to `NULL` and the `service` is set to the port number.

* We iterate through each of the `addrinfo` structures in the result list and try to use them with the `socket` and `bind` functions. The goal is to create a socket and bind it to an address and port where it can listen for incoming connections. If the `socket` or `bind` call fails, we try the next `addrinfo` structure in the list until we find one that works or exhaust all options.

* We invoke the `listen` function on the bound socket to turn it into a listening socket. This tells the operating system that we want this socket to be used to accept incoming connection requests.

* We call `accept` in a loop to wait for and accept incoming connections. When a client connects, `accept` returns a new socket that's specifically linked to that client. We can then use this new socket to communicate with the client.

### The `socket` Function

Clients and servers use the `socket` function to create a *socket descriptor*.

```c
#include <sys/types.h>
#include <sys/socket.h>

int socket(int domain, int type, int protocol);

// for the parameter protocol you can just use 0 and the system will choose the correct protocol based on the type.
// Returns: nonnegative descriptor if OK, −1 on error

clientfd = socket(AF_INET, SOCK_STREAM, 0);
```

`AF_INET` indicates that we are using 32-bit IP addresses and `SOCK_ STREAM ` indicates that the socket will be an end point for a connection.

The best practice is to use the `getaddrinfo` function to generate these parameters automatically, so that the code is protocol-independent.

### The `connect` Function

```c
#include <sys/socket.h>

int connect(int clientfd, const struct sockaddr *addr, socklen_t addrlen);

// Returns: 0 if OK, −1 on error
```

The `connect` function attempts to establish an Internet connection with the server at the socket address `addr`, where addrlen is `sizeof(sockaddr_in)`. **The `connect `function blocks until either the connection is successfully established or an error occurs.** If successful, the `clientfd` descriptor is now ready for reading and writing, and the resulting connection is characterized by the socket pair `(x:y, addr.sin_addr:addr.sin_port)`. As with socket, the best practice is to use `getaddrinfo` to supply the arguments to connect.

### The `bind` Function

```c
#include <sys/socket.h>

int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);

// Returns: 0 if OK, −1 on error
```

The `bind` function asks the kernel to associate the server’s socket address in `addr` with the socket descriptor `sockfd`.

### The `listen` Function

The client is the active side that initiates connection requests. The server side is the passive entities that wait for connection requests from clients. By default, the kernel assumes that a descriptor created by the `socket` function is the client side of a connection. A server calls the `listen` function to tell the kernel that the descriptor is on the server side.

```c
#include <sys/socket.h>

int listen(int sockfd, int backlog);
```

The `backlog` argument is a hint about the number of outstanding connection requests that the kernel should queue up before it starts to refuse requests.

### The `accept` Function

Servers wait for connection requests from clients by calling the `accept` function.

```c
#include <sys/socket.h>

int accept(int listenfd, struct sockaddr *addr, int *addrlen);

// Returns: nonnegative connected descriptor if OK, −1 on error
```

The accept function waits for a connection request from a client to arrive on the listening descriptor listenfd, then fills in the client’s socket address in addr, and returns a **connected descriptor** that can be used to communicate with the client using Unix I/O functions.

The **listening descriptor** serves as an end point for client connection requests. It is typically created once and exists for the lifetime of the server. 

The **connected descriptor** is the end point of the connection that is established between the client and the server. It is created each time the server accepts a connection request and exists only as long as it takes the server to service a client.

![Screenshot 2023-05-13 at 9.13.48 PM](https://p.ipic.vip/901bi8.png)

![Screenshot 2023-05-27 at 7.12.33 AM](https://p.ipic.vip/wph8pk.png)

This implies that a port in a host can have multiple connections. A connection is identified by the 5-tuple.

### Host and Service Conversion

`getaddrinfo` and `getnameinfo` converts back and forth between binary socket address strcutures and the string representations of hostnames, host addresses, service names and port numbers.

**The `getaddrinfo`** Function

The `getaddrinfo` function converts string representations of hostnames, host addresses, service names, and port numbers into socket address structures.

It is reentrant and works with any protocol.

```c
#include <sys/types.h>
#include <sys/socket.h>
#include <netdb.h>

int getaddrinfo(const char *host, const char *service,
                const struct addrinfo *hints,
                struct addrinfo **result);

// Returns: 0 if OK, nonzero error code on error

void freeaddrinfo(struct addrinfo *result);

// Returns: nothing

const char *gai_strerror(int errcode);

// Returns: error message

struct addrinfo {
  int ai_flags;     /* Hints argument flags */
	int ai_family;    /* First arg to socket function */
	int ai_socktype;  /* Second arg to socket function */
	int ai_protocol;  /* Third arg to socket function */
  char *ai_canoname; /* Canonical hostname */
  size_t ai_addrlen; /* Size of ai_addr struct */
  struct sockaddr *ai_addr; /* Ptr to socket address structure */
  struct addrinfo *ai_next; /* Ptr to next item in linked list */
};
```

Given `host` and `service` (the two components of a socket address), `getaddrinfo` returns a result that points to a linked list of `addrinfo` structures, each of which points to a socket address structure that corresponds to `host` and `service`.

After a client calls `getaddrinfo`, it walks this list, trying each socket address in turn until the calls to socket and connect succeed and the connection is established.

Similarly, a server tries each socket address on the list until the calls to socket and bind succeed and the descriptor is bound to a valid socket address.

To avoid memory leaks, the application must eventually free the list by calling `freeaddrinfo`. If `getaddrinfo` returns a nonzero error code, the application can call `gai_strerror` to convert the code to a message string.

The optional `hints` argumet is an `addrinfo` structure that provides finer control over the list of socket addresses that `getaddrinfo` returns. When passes as a `hints` argument, only the `ai_family`, `ai_socktype`, `ai_protocol`, and `ai_flags` fields can be set. The other fields must be set to zero (or NULL). We use `memset` to zero the entire structure and set a few selected fields:

* Setting `ai_family` to `AF_INET` restricts the list to IPv4 addresses. Setting it to `AF_INET6` restricts the list to IPv6 addresses.

* Setting `ai_socktype` to `SOCK_STREAM` restricts the list to at most one `addrinfo ` structure for each unique address, one whose socket address can be used as the end point of a connection. 

  `SOCK_STREAM` is a type of reliable and connection-oriented socket adopted by TCP. UDP uses `SOCK_DGRAM`.

* The `ai_flags` field is a bit mask that further modifies the default behavior. You create it by oring combinations of various values. 

  * `AI_ADDRCONFIG`. This flag is recommended if you are using connections. It asks getaddrinfo to return IPv4 addresses only if the local host is configured for IPv4. Similarly for IPv6.

  * `AI_CANONNAME`. By default, the ai_canonname field is NULL. If this flag is set, it instructs `getaddrinfo` to point the ai_canonname field in the first addrinfo structure in the list to the canonical (official) name of host.

  * `AI_NUMERICSERV`. By default, the service argument can be a service name or a port number. This flag forces the service argument to be a port number.
  * `AI_PASSIVE`. Bydefault, `getaddrinfo` returns socket addresses that can be used by clients as active sockets in calls to connect. This flag instructs it to return socket addresses that can be used by servers as listening sockets. In this case, the host argument should be NULL. The address field in the resulting socket address structure(s) will be the *wildcard address*, which tells the kernel that this server will accept requests to any of the IP addresses for this host. This is the desired behavior for all of our example servers.

When `getaddrinfo` creates an `addrinfo` structure in the output list, it fills in each field except for `ai_flags`. 

One of the elegant aspects of `getaddrinfo` is that the fields in an `addrinfo `structure are opaque, in the sense that they can be passed directly to the functions in the sockets interface without any further manipulation by the application code.

**The `getnameinfo` Function**

The `getnameinfo` function is inverse of `getaddrinfo`. It is reentrant and protocol-independent.

```c
#include <sys/socket.h>
#include <netdb.h>

int getnameinfo(const struct sockaddr *sa, socklen_t salen,
                char *host, size_t hostlen,
                char *service, size_t servlen, int flags);

// Returns: 0 if OK, nonzero error code on error
```

 The `sa` argument points to a socket address structure of size `salen` bytes, `host` to a buffer of size `hostlen` bytes and `service` to a buffer of size `servlen` bytes.

If `getnameinfo` returns a nonzero error code, the application can convert it to a string by calling `gai_strerror`.

The `flags` argument is a bit mask that modifies the default behavior.

* `NI_NUMERICHOST`. By default, `getnameinfo` tries to return a domain name in host. Setting this flag will cause it to return a numeric address string instead.
* `NI_NUMERICSERV`. By default, `getnameinfo` will look in /etc/services and if possible, return a service name instead of a port number. Setting this flag forces it to skip the lookup and simply return the port number.

### Helper Functions for the Sockets Interface

```c
int open_clientfd(char *hostname, char *port) {
  int clientfd;
  struct addrinfo hints, *listp, *p;
  
  memset(&hints, 0, sizeof(struct addrinfo));
  hints.ai_socktype = SOCK_STREAM;
  hints.ai_flags = AI_NUMERICSERV;
  hints.ai_flags |=  AI_ADDRCONFIG;
  
  getaddrinfo(hostname, port, &hints, &lisp);
  
  for(p = listp; p; p = p->ai_next) {
    if ((clientfd = socket(p->ai_family, p->ai_socktype, p->ai_protocol)) < 0)
      continue;
    if (connect(clientfd, p->ai_addr, p->ai_addrlen) != -1)
      break;
    close(clientfd);
  }
  
  freeaddrinfo(listp);
  if(!p)
    return -1;
  else
    return clientfd;
}
```

```c
int open_listenfd(char *port) {
  struct addrinfo hints, *listp, *p;
  int listenfd, optval=1;
  
  memset(&hints, 0, sizeof(struct addrinfo));
  hints.ai_socktype = SOCK_STREAM;
  hints.ai_flags = AI_PASSIVE | AI_ADDRCONFIG; // on any IP addresses
  hint.ai_flags = AI_NUMERICSERV; // using port number
  getaddrinfo(NULL, port, &hints, &listp);
  
  for(p = listp; p; p = p->ai_next) {
    if((listenfd = socket(p->ai_family, p->ai_socktype, p->ai_protocol)) < 0)
      continue;
    
    // Eliminates "Address already in use" error from bind
    setsockopt(listenfd, SOL_SOCKET, SO_REUSEADDR, (const void *)&optval, sizeof(int));
    
    if (bind(listenfd, p->addr, p->ai_addrlen) == 0)
      break;
    
    close(listenfd);
  }
  
  freeaddrinfo(listp);
  if(!p)
    return -1;
  
  if(listen(listenfd, LISTRENQ) < 0) {
    close(listenfd);
    return -1;
  }
  return listenfd;
}
```

It is good programming practice to explicitly close any descriptors that you have opened.
