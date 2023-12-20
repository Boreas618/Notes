# Inter-process Communication

**Two Primitives**:

* `send(destination, message)`
* `receive(source, message)`

## Addressing

**Direct addressing** or **Indirect addressing**

### Direct Addressing

With symmetric direct addressing, both the sender and the receiver processes must name the other to communicate.

```pseudocode
send(P, message)
receive(Q, message)
```

With asymmetric direct addressing, only the sender **name**s the recipient, the recipient is not required to **name** the sender.

```pseudocode
send(P, message) // Send a message to process P
receive(id, message) // Receive a message from any process; the id is set to the name of the sender
```

### Indirect Addressing

Messages are sent to a shared data structure consisting of queues. Such queues are generally called **mailboxes**.

**Benefit of indirect addressing**: (more flexibility) The relationship between senders and receivers can be one-to-one, many-to-one, one-to-many, or many-to-many.

**The ownership of a mailbox**：

* **Process mailbox ownership**: Only the owner process may receive messages from the mailbox, and other processes may send to this mailbox.
* **System mailbox ownership**

## Synchronization

Sender and receiver may or may not be blocking.

* **Blocking send, blocking receive**: Both sender and receiver are blocked until message is

  delivered. Called a rendezvous.

* **Nonblocking send, blocking receive**: Sender continues processing such as sending messages as

  quickly as possible; Receiver is blocked until the requested message arrives.

* **Nonblocking send, nonblocking receive**: Neither party is required to wait.

## Buffering

- **Zero capacity**: the sender must block until the recipient receive the message
- **Bounded capacity**: the queue has finite length `n`; full or not-full?
- **Unbounded capacity**: the queue length is potentially infinite.The sender never blocks.
