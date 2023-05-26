# Principles of Network Applications

## Process Communication

Processes on two different end systems communicate with each other by exchanging **messages** across the computer network. Processes communicating with each other reside in the application layer of the five-layer protocol stack.

**Client and Server Processes:** When there’s no confusion, we’ll sometimes also use the terminology “client side and server side of an application". **An application has client and server side.**

**The Interface Between the Process and the Computer Network:** A process sends messages into, and receives messages from, the network through a software interface called a **socket**.

![](https://p.ipic.vip/jk95ql.png)

A socket is the interface between the application layer and the transport layer within a host. It is also referred to as the **Application Programming Interface (API)** between the application and the network, since the socket is the programming interface with which network applications are built.

**Addressing Processes**

To identify the receiving process, two pieces of information need to be specified:

* the address of the host (IP address)
* an identifier that specifies the receiving process in the destination host (port number)

**Only one program can bind a socket to a particular port at any given time.**

## Transport Services Available to Applications

Four dimensions of services a transport-layer protocol can offer to applications:

* reliable data transfer
* throughput
* timing
* security

**Reliable data transfer**: Some multimedia applications are **loss-tolerant applications** while some other applications requires reliable data transfer.

**Throughout**: a guaranteed throughput of r bits/sec. Applications that have throughput requirements are said to be **bandwidth-sensitive applications**. WeChat video call.

**Timing**: An example guarantee might be that every bit that the sender pumps into the socket arrives at the receiver’s socket no more than 100 ms later.

## Transport Services Provided by the Internet

The Internet (and, more generally, TCP/IP networks) makes two transport protocols available to applications, UDP and TCP. When you (as an application developer) create a new network application for the Internet, one of the first decisions you have to make is whether to use UDP or TCP. Each of these protocols offers a different set of services to the invoking applications.

**TCP Services**

The TCP service model includes a connection-oriented service and a reliable data transfer service.

*   Connection-oriented services

    Handshaking $$\rightarrow$$ TCP connection

    The connection is a full-duplex connection in that the two processes can send messages to each other over the connection at the same time. When the application finishes sending messages, it must tear down the connection.
*   Reliable data transfer

    No missing or duplicate bytes

TCP congestion control also attempts to limit each TCP connection to its fair share of network bandwidth.

**UDP Services**

UDP is a no-frills, lightweight transport protocol, providing minimal services. Connectionless, no handshaking. No congestion.

Today’s Internet can often provide satisfactory service to time-sensitive applications, but it cannot provide any timing or throughput guarantees.

Internet telephony applications usually prefer to run their applications over UDP, thereby circumventing TCP’s congestion control mechanism and packet overheads. But because many firewalls are configured to block (most types of) UDP traffic, TCP serves as a backup if UDP communication fails.

![](https://p.ipic.vip/xpkp3e.png)

## Application-Layer Protocols

RFC vs Proprietary application protocols

# The Web and HTTP

The World Wide Web is an Internet application.

## Overview of HTTP

HTTP(Hyper Text Transfer Protocol) is the Web's application-layer protocol. It is implemented in two programs: a client program and a server program. The client and server exchange HTTP messages.

A Web page consists of objects. A HTML file references the other objects in the page with the object's URLs.

**HTTP uses TCP as its underlying transport protocol.**

The HTTP client first initiates a TCP connection with the server. Once the connection is established, the browser and the server processes access TCP through their socket interfaces.

Because an HTTP server maintains no information about the clients, HTTP is said to be a **stateless protocol**.

## Non-Persistent and Persistent Connections

Although HTTP uses persistent connections in its default mode, HTTP clients and servers can be configured to use non-persistent connections instead.

**Non-Persistant Connections**

**Example:**

```
http://www.someSchool.edu/someDepartment/home.index
```

1. The HTTP client process initiates a TCP connection to the server on port number 80.
2. The HTTP client sends an HTTP request message to the server via its socket. The request message includes the path name `/someDepartment/home.index`.
3. The HTTP server process receives the request message via its socket, encapsulates the object in an HTTP response message, and sends the response message to the client via its socket.
4. The HTTP server process tells TCP to close the TCP connection.
5. The HTTP client receives the response message. The TCP connection terminates.
6. The first four steps are then repeated for each of the referenced JPEG objects.

11 TCP connections are generated.

In their default modes, most browsers open 5 to 10 parallel TCP connections, and each of these connections handles one request-response transaction.

A three-way handshake: the client sends a small TCP segment to the server, the server acknowledges and responds with a small TCP segment, and, finally, the client acknowledges back to the server.

**Persistent Connections**

Subsequent requests and responses between the same client and server can be sent over the same connection.

Multiple Web pages residing on the same server can be sent from the server to the same client over a single persistent TCP connection.

Typically, the HTTP server closes a connection when it isn’t used for a certain time (a configurable timeout interval)

## HTTP Message Format

```http
GET /somedir/page.html HTTP/1.1
Host: www.someschool.edu
Connection: close
User-agent: Mozilla/5.0
Accept-language: fr

```

Request line and header lines

`Connection: close` means non-persistent connection.

<figure><img src="https://p.ipic.vip/95tmpc.png" alt="" width="375"><figcaption></figcaption></figure>

`HEAD` leaves out the requested object. For debugging.

`PUT` upload something to the server.

```http
HTTP/1.1 200 OK
Connection: close
Date: Tue, 18 Aug 2015 15:44:04 GMT
Server: Apache/2.2.3 (CentOS)
Last-Modified: Tue, 18 Aug 2015 15:11:03 GMT
Content-Length: 6821
Content-Type: text/html

(data data data data data ...)
```

Status line and header lines

The `Date:` header line indicates the time and date when the HTTP response was created and sent by the server.

<figure><img src="https://p.ipic.vip/xqorkn.png" alt="" width="375"><figcaption></figcaption></figure>

## User-Server Interaction: Cookies

HTTP is stateless. We can use cookies to keep track of users.

Cookie technology has four components:

* a cookie header line in the HTTP response message
* a cookie header line in the HTTP request message
* a cookie file kept on the user’s end system and managed by the user’s browser
* a back-end database at the Web site.

**Example:** in the response message:

```http
Set-cookie: 1678
```

Then the messages sent by the client has the header line:

```http
Cookie: 1678
```

Cookies can thus be used to create a user session layer on top of stateless HTTP.

## Web Caching

A **Web cache**—also called a **proxy server**—is a network entity that satisfies HTTP requests on the behalf of an origin Web server. The Web cache has its own disk storage and keeps copies of recently requested objects in this storage.

<figure><img src="https://p.ipic.vip/nithz4.png" alt="" width="375"><figcaption></figcaption></figure>

A cache is both a server and a client at the same time.

Typically a Web cache is purchased and installed by an ISP.

Through the use of **Content Distribution Networks (CDNs)**, Web caches are increasingly playing an important role in the Internet.

There are shared CDNs (such as Akamai and Limelight) and dedicated CDNs (such as Google and Netflix).

**Conditional GET**

* uses `GET` method
* the request message includes an `If-Modified-Since:` header line

# DNS - The Internet's Directory Service

## Service Provided by DNS

Domain Name System

The DNS is:

* A distributed database implemented in a hierarchy of **DNS servers**
* **An application-layer protocol** that allows hosts to query the distributed databases

The DNS protocol runs over UDP and uses port 53.

1. The same user machine runs the client side of the DNS application.
2. The browser extracts the hostname, _www.someschool.edu_, from the URL and passes the hostname to the client side of the DNS application.
3. The DNS client sends a query containing the hostname to a DNS server.
4. The DNS client eventually receives a reply, which includes the IP address for the hostname.
5. Once the browser receives the IP address from DNS, it can initiate a TCP connection to the HTTP server process located at port 80 at that IP address.

DNS provides a few other important services in addition to translating hostnames to IP addresses:

*   Host aliasing

    **Canonical hostname** vs **Alias hostname** The latter is more mnemonic. DNS can be invoked by an application to obtain the canonical hostname for a supplied alias hostname as well as the IP address of the host.
* Mail server aliasing
*   Load distribution

    When clients make a DNS query for a name mapped to a set of addresses, the server responds with the entire set of IP addresses, but rotates the ordering of the addresses within each reply. Because a client typically sends its HTTP request message to the IP address that is listed first in the set, DNS rotation distributes the traffic among the replicated servers.

## Overview of How DNS Works

The application will invoke the client side of DNS, specifying the hostname that needs to be translated. (On many UNIX-based machines, `gethostbyname()` is the function call that an application calls in order to perform the translation.)

DNS in the user’s host then takes over, sending a query message into the network. All DNS query and reply messages are sent within UDP datagrams to port 53.

**A Distributed, Hierarchical Database**

In order to deal with the issue of scale, the DNS uses a large number of servers, organized in a hierarchical fashion and distributed around the world. No single DNS server has all of the mappings for all of the hosts in the Internet.

The client first contacts one of the **root servers**, which returns IP addresses for **TLD(Top Level Domain) servers** for the top-level domain _com_. The client then contacts one of these TLD servers, which returns the IP address of an **authoritative server** for `amazon.com`. Finally, the client contacts one of the authoritative servers for `amazon.com`, which returns the IP address for the hostname `www.amazon.com`.

An organization’s authoritative DNS server houses these DNS records. An organization can choose to implement its own authoritative DNS server to hold these records; alternatively, the organization can pay to have these records stored in an authoritative DNS server of some service provider.

**Local DNS server** A local DNS server does not explicitly belong to the hierarchy of servers but is nevertheless central to the DNS architecture. When a host connects to an ISP, the ISP provides the host with the IP addresses of one or more of its local DNS servers (typically through DHCP). You can easily determine the IP address of your local DNS server by accessing network status windows in Windows or UNIX. When a host makes a DNS query, the query is sent to the local DNS server, which acts a proxy, forwarding the query into the DNS server hierarchy, as we’ll discuss in more detail below.

![Screenshot 2023-05-01 at 1.01.27 PM](https://p.ipic.vip/w9ji26.png)

The query sent from `cse.nyu.edu` to `dns.nyu.edu` is a recursive query, sicne the query asks `dns.nyu.edu` to obtain the mapping on its behalf. But the subsequent three queries are iterative since all of the replies are directly returned to `dns.nyu.edu`. In theory, any DNS query can be iterative or recursive.

The query from the requesting host to the local DNS server is recursive, and the remaining queries are iterative.

**DNS Caching**

Each time the local DNS server `dns.nyu.edu` receives a reply from some DNS server, it can cache any of the information contained in the reply.

A local DNS server can also cache the IP addresses of TLD servers, thereby allowing the local DNS server to bypass the root DNS servers in a query chain. In fact, because of caching, root servers are bypassed for all but a very small fraction of DNS queries.

## DNS Records and Messages

The DNS servers store resource records(RRs), including RRs that provides hostname-to-IP address mappings. Each DNS reply message carries one or more resource records.

A four-tuple `(Name, Value, Type, TTL)`

`TTL` is the time to live of the resource record.

* `Type=A` then `Name` is a hostname and `Value` is the IP address for the hostname
* `Type=NS` then `Name` is a domain and `Value` is the hostname of an authoritative DNS server that knows how to obtain the IP address for hosts in the domain.
* `Type=CNAME` then `Value` is a canonical hostname for the alias hostname `Name`. This record can provide querying hosts the canonical name for a hostname.
* `Type=MX`, then `Value` is the canonical name of a mail server that has an alias hostname `Name`.

If a DNS server is authoritative for a particular hostname, then the DNS server will contain a Type A record for the hostname. (Even if the DNS server is not authoritative, it may contain a Type A record in its cache.)

If a server is not authoritative for a hostname, then the server will contain a Type NS record for the domain that includes the hostname; it will also contain a Type A record that provides the IP address of the DNS server in the _Value_ field of the NS record.

**DNS Messages**

Both query and reply messages have the same format. This semantics of the various fields in a DNS message are as follows:

![Screenshot 2023-05-01 at 1.26.17 PM](https://p.ipic.vip/di59ha.png)

**Inserting Records into the DNS Database**

Register your domain name at a registrar. A **registrar** is a commercial entity that verifies the uniqueness of the domain name, enters the domain name into the DNS database, and collects a small fee from you for its services.

When you register the domain name **networkutopia.com** with some registrar, you also need to provide the registrar with the names and IP addresses of your primary and secondary authoritative DNS servers.
