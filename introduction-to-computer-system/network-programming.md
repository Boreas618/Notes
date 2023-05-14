# Network Programming

It is important to realize that clients and servers are **processes** and not machines, or *hosts* as they are often called in this context.

## Networks

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

## The Global Internet

### IP Addresses

TCP/IP defines a uniform **network byte order** (big-endian byte order) for any integer data item, such as an IP address, that is carried across the network in a packet header. Addresses in IP address structures are always stored in (big-endian) network byte order, even if the host byte order is little-endian.

```c
#include <arpa/inet.h>

uint32_t htonl(uint32_t hostlong);
uint16_t htons(uint16_t hostshort);

// returns value in network byte order

uint32_t ntohl(uint32_t netlong);
uint16_t ntohs(unit16_t netshort);

// returns value in host byte order
```

Application programs can convert back and forth between IP addresses and dotted-decimal strings using the functions `inet_pton` and `inet_ntop`.

```c
#include <arpa/inet.h>

int inet_pton(AE_INET, const char *src, void *dst);
// returns: 1 if OK, 0 if src is invalid dotted decimal, −1 on error

const char *inet_ntop(AF_INET, const void *src, char *dst, socklen_t size);
// returns: pointer to a dotted-decimal string if OK, NULL on error
```

`AE_INET` for IPv4, `AE_INET6` for IPv6.

### Internet Domain Names

<img src="https://p.ipic.vip/gx2rzp.png" alt="Screenshot 2023-05-03 at 12.09.24 PM" style="zoom:50%;" />

### Internet Connections

Full duplex: the data can flow in both directions

A **socket** is an end point of a connection. Each socket has a corresponding *socket address* that consists of an Internet address and a 16-bit integer **port** and is denoted by the notation `address:port`.

The port in the server’s socket address is typically some *well-known port* that is permanently associated with the service. The mapping between well-known names( smtp, http ) and well-known ports( 25, 80 ) is contained in a file called `/etc/services`.

By default, HTTP requests use port 80 for non-secure (HTTP) connections and port 443 for secure (HTTPS) connections. However, if the server is running on a different port, you can specify the port number in the request URL, like `http://fduhole.com:8080/`.

A socket tuple: `(cliaddr:cliport, servaddr:servport)`

## The Sockets Interface

<img src="https://p.ipic.vip/jstaky.png" alt="Screenshot 2023-05-03 at 12.50.20 PM" style="zoom:50%;" />

From the perspective of the Linux kernel, a socket is an end point for communi- cation. From the perspective of a Linux program, a socket is an open file with a corresponding descriptor.

Internet socket addresses are stored in 16-byte structures having the type `sockaddr_in`. 

```c
/* IP socket address structure */
struct sockaddr_in  {
	uint16_t        sin_family;  /* Protocol family (always AF_INET) */
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

### The `socket` Function

Clients and servers use the `socket` function to create a *socket descriptor*.

```c
#include <sys/types.h>
#include <sys/socket.h>

int socket(int domain, int type, int protocol);

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

The `connect` function attempts to establish an Internet connection with the server at the socket address `addr`, where addrlen is `sizeof(sockaddr_in)`. The `connect `function blocks until either the connection is successfully established or an error occurs. If successful, the `clientfd` descriptor is now ready for reading and writing, and the resulting connection is characterized by the socket pair `(x:y, addr.sin_addr:addr.sin_port)`. As with socket, the best practice is to use `getaddrinfo` to supply the arguments to connect.

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
  
  getaddrinfo(hostname, port &hints, &lisp);
  
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

## Web Servers

### Web Content

| MIME type              | Description                         |
| ---------------------- | ----------------------------------- |
| text/html              | HTML page                           |
| text/plain             | Unformatted text                    |
| application/postscript | Postscript document                 |
| image/gif              | Binary image encoded in GIF format  |
| image/png              | Binary image encoded in PNG format  |
| image/jpeg             | Binary image encoded in JPEG format |

Web servers provide content to clients in two different ways:

* Fetch a disk file return its content to the client. **Serving static content**
* Run a executable file and return its output to the client. **Serving dynamic content**

The minimal URL suffix is the ‘/’ character, which all servers expand to some default home page such as /index.html.

### HTTP Transactions

```
request line
request headers

request body
```

A request line has the form `model URI version`

The GET method instructs the server to generate and return the content identified by the **URI**. The URI is the suffix of the corresponding URL that includes the filename and optional arguments.

The Host header is required in HTTP/1.1 requests, but not in HTTP/1.0 requests. The Host header is used by **proxy caches**, which sometimes serve as intermediaries between a browser and the **origin server** that manages the requested file. Multiple proxies can exist between a client and an origin server in a so-called proxy chain. The data in the Host header, which identifies the domain name of the origin server, allow a proxy in the middle of a proxy chain to determine if it might have a locally cached copy of the requested content.

```
response line
response headers

response body
```

A response line has the form `version status-code status-message`

