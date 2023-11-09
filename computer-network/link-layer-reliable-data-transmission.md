# Principles of Reliable Data Transfer

Assumption:  packets will be delivered in the order in which they were sent, with some packets possibly being lost; that is, the underlying channel will not reorder packets.

**ARQ(Automatic Repeat reQuest) protocols**: Use both **positive acknowledgements** and **negative acknowledgements**. Repeat the message in error.

Capabilities needed:

* **Error detection**

* **Receiver feedback** The positive (ACK) and negative (NAK) acknowledgment replies in the message-dictation scenario are examples of such feedback. Simply one bit.

* **Retransmission**

  A packet that is received in error at the receiver will be retransmitted by the sender.

<img src="https://p.ipic.vip/1mzrrk.png" alt="Screenshot 2023-05-11 at 2.50.28 PM" style="zoom: 33%;" />

# Stop-and-Wait

It is important to note that when the sender is in the wait-for-ACK-or-NAK state, it cannot get more data from the upper layer. This is known as **stop-and-wait** protocols.

However, the ACK or NAK can also be corrupted. One solution is to resend the packet when we received a garbled ACK or NAK packet. However, a new issue is introduced in this solution. The receiver of the resented packet doesn't know if the arriving packet contains new data or is a retransmission.

To address this issue, we add the **sequence number** into the packet. For the stop-and-wait protocol, we only need one bit to represent the sequence number. 

![Screenshot 2023-10-15 at 1.03.37 AM](https://p.ipic.vip/w3uoiu.png) 

In computer networks, it is common for the underlying channel to lose packets. To address this, we assign the task of detecting and recovering from lost packets to the sender. The sender waits for a specific time period to check if a packet has been lost. The sender strategically chooses a time value that increases the chances of detecting packet loss. However, if a packet encounters a significant delay, the sender may retransmit it even if neither the data packet nor its acknowledgment (ACK) has been lost. Luckily, rdt2.2 can handle this scenario of duplicate packets.

To implement a time-based retransmission mechanism, a countdown timer is needed. This timer interrupts the sender after a set amount of time. The sender should be able to (1) start the timer whenever a packet is sent, (2) respond to a timer interrupt appropriately, and (3) stop the timer.

<img src="https://p.ipic.vip/tasxuk.png" alt="Screenshot 2023-05-23 at 8.51.50 PM" style="zoom: 33%;" />

## Performance Analysis

**Effective Utilization**: the actual rate of transmitting effective data on the channel, also known as the normalized throughput.

Capacity of the channel: **B** bps

Length of the frame: **L** bit

Turnaround transmission delay: **2R** s
$$
U = \frac{\frac{L}{B}}{\frac{L}{B} + 2R}
$$
**Throughput**: the amount of (valid) data passed within a unit of time.
$$
\text{Throughput} = U \times B
$$

# Pipelined Reilable Data Transfer Protocols

The issue with the above design of reliable protocol is that it is a **stop-and-wait** protocol. We adopt pipelining to address this issue.

The range of sequence number needs to be increased now. The sender and receiver sides of the protocols may have to buffer more than one packet.

The utilization of the channel is calculated as:
$$
U_{\text{pipeline}} = \frac{W\times \frac{L}{B}}{2R + \frac{L}{B}}
$$
Where **W** is the window size.

> The utilization rate is calculated from the perspective of the sender. It is defined as the effective time for transmitting data within a transmission cycle.

In order to achieve 100% effective utilization, the window size $W$ is $\frac{L + 2R\times B}{L}$.

Two pipelined error recovery can be identified: **Go-Back-N** and **selective repeat**

## Go-Back-N

**Assumption:** the buffer holds packets rather than bytes.

The sender is allowed to transmit multiple packets (when available) without waiting for an acknowledgment, but is constrained to have no more than some maximum allowable number, $N$, of unacknowledged packets in the pipeline.

![Screenshot 2023-05-23 at 9.17.21 PM](https://p.ipic.vip/e13cdw.png)

We define `base` to be the sequence number of the oldest unacknowledged packet and `nextseqnum` to be the smallest unused sequence number.

| Interval                               | Description                   |
| -------------------------------------- | ----------------------------- |
| $[0, \text{base}-1]$                   | Sent and acknowledged         |
| $[\text{base}, \text{nextseqnum}-1]$   | Sent but not yet acknowledged |
| $[\text{nextseqnum}, \text{base+N-1}]$ | Can be sent                   |
| $[\text{base+N}, \infty]$              | Cannot be used                |

$N$ is referred to as the **window size**. **GBN** protocol is a **sliding-window protocol**.  

Why we need to limit th e window size to $N$ ?

![Screenshot 2023-10-15 at 12.32.47 AM](https://p.ipic.vip/d5zus5.png)

### Sender Side

**Invocation from above** When `rdt_send()` is called from above, the sender first checks to see if the window is full, that is, whether there are *N* outstanding, unacknowledged packets. If the window is not full, a packet is created and sent, and variables are appropriately updated.

**Receipt of an ACK**  In our GBN protocol, an acknowledgment for a packet with sequence number *n* will be taken to be a **cumulative acknowledgment**, indicating that all packets with a sequence number up to and including *n* have been correctly received at the receiver.

**Timeout Event** The protocol’s name, “Go-Back-N,” is derived from the sender’s behavior in the presence of lost or overly delayed packets. As in the stop-and-wait protocol, a timer will again be used to recover from lost data or acknowledgment packets. If a timeout occurs, the sender resends **all** packets that have been previously sent but that have not yet been acknowledged. In the illustration, there's only one timer, which can be thought of as a timer for the oldest transmitted but not yet acknowledged packet. If an ACK is received but there are still additional transmitted but not yet acknowledged packets, the timer is restarted. If there are no outstanding, unacknowledged packets, the timer is stopped.

### Receiver Side

ACK N means **expecting N** in COMP130136.

In our GBN protocol, the receiver discards out-of-order packets.

While the sender must maintain the upper and lower bounds of its window and the position of *nextseqnum* within this window, the only piece of information the receiver need maintain is **the sequence number of the next in-order packet**. This value is held in the variable `expectedseqnum`.

<img src="https://p.ipic.vip/cg19tc.png" alt="Screenshot 2023-05-26 at 6.41.00 PM" style="zoom: 25%;" />

## Selective Repeat

The issue of GBN is that the retransmissions are wasteful.

As the name suggests, selective-repeat protocols avoid unnecessary retransmissions by having the sender retransmit only those packets that it suspects were received in error (that is, were lost or corrupted) at the receiver. 

This individual, as-needed, retransmission will require that the receiver *individually* acknowledge correctly received packets. A window size of *N* will again be used to limit the number of outstanding, unacknowledged packets in the pipeline. However, unlike GBN, the sender will have already received ACKs for some of the packets in the window.

<img src="https://p.ipic.vip/c9z3ez.png" alt="Screenshot 2023-05-26 at 7.11.14 PM" style="zoom: 33%;" />

**Out-of-order packets are buffered** until any missing packets (that is, packets with lower sequence numbers) are received, at which point a batch of packets can be delivered in order to the upper layer.

**The receiver reacknowledges (rather than ignores) already received packets with certain sequence numbers *below* the current window base.** You should convince yourself that this reacknowledgment is indeed needed.

**The receiver won't acknowledges the received packets with certain sequence numbers *above* the current window base+size.**

<img src="https://p.ipic.vip/mjqqb5.png" alt="Screenshot 2023-05-26 at 7.38.30 PM" style="zoom: 25%;" />

If a packet has been received by the receiver in SR, and the same packet is sent once again, the receiver would typically acknowledge it as if it was the first time the packet was received. If the sender doesn't receive the acknowledgement, the window won't move forward.

**The receiver window should be smaller or same size as the sender window.** 

**The window size must be less than or equal to half the size of the sequence number space for SR protocols.** Otherwise, the receiver cannot tell apart a new packet and a retransmission.

<img src="https://p.ipic.vip/a956cn.png" alt="Screenshot 2023-05-26 at 7.55.09 PM" style="zoom:50%;" />

> Implementing SR with NAK
>
> 1. **Frame Numbering**: Both the sender and receiver maintain a window of acceptable frame numbers. This is to keep track of which frames are in transit and which are acknowledged.
> 2. **Sending Frames**: The sender transmits a number of frames up to its window size without waiting for acknowledgments for each individual frame.
> 3. **Receiving Frames & Sending NAKs**: When the receiver gets a frame:
>    - If the frame is correctly received and is within the expected window, it sends an acknowledgment (ACK) for that frame.
>    - If the frame is erroneous (detected using mechanisms like CRC) or if an expected frame didn't arrive in a certain time period, the receiver sends a NAK for that specific frame number.
> 4. **Handling NAKs**: When the sender receives a NAK for a specific frame:
>    - It retransmits only that specific frame (not all the frames).
>    - The sender continues to transmit new frames (if available and within the window).
> 5. **Advancing the Window**: As frames are acknowledged, both the sender and receiver slide their window of acceptable frame numbers.
> 6. **Timeouts**: There may still be timeouts at the sender's side to handle cases where neither ACK nor NAK is received for a frame within a certain time frame. Upon timeout, the sender would retransmit the specific frame.
> 7. **Sequence Number Size**: The sequence number size determines the range of frame numbers. This size should be designed considering the window size to ensure that the frames in transit (i.e., frames that have been sent but not yet acknowledged) are uniquely identifiable.