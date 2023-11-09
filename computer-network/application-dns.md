# DNS

DNS stands for **Domain Name System**. The DNS is:

* A distributed database implemented in a hierarchy of **DNS servers**
* **An application-layer protocol** that allows hosts to query the distributed databases

The DNS protocol runs over **UDP** and uses port 53.

## DNS for Resolving Domain Names

A domain name has a maximum length of 255 bytes. It's formed by a series of labels. Each label:

* Has a maximum length of 63 bytes.
* Can include alphanumeric characters and hyphens (-), but no other symbols.

**Domain Name Servers (DNS servers)** are responsible for managing **zones**. These servers are categorized into:

* Root Domain Name Server
* Top-Level Domain Server
* Authoritative Domain Name Server
* Local Domain Name Server

It's worth noting that a single server can function as both a Local and an Authoritative Domain Name Server simultaneously. For instance, a university's DNS server might store cached Resource Records (RRs) while also housing the actual address of its domain name.

>  DNS provides a few other important services in addition to translating hostnames to IP addresses:
>
> * Host aliasing
>
>   **Canonical hostname** vs **Alias hostname** The latter is more memonic. DNS can be invoked by an application to obtain the canonical hostname for a supplied alias hostname as well as the IP address of the host.
>
> * Mail server aliasing
>
> * Load distribution
>
>   When clients make a DNS query for a name mapped to a set of addresses, the server responds with the entire set of IP addresses, but rotates the ordering of the addresses within each reply. Because a client typically sends its HTTP request message to the IP address that is listed first in the set, DNS rotation distributes the traffic among the replicated servers.

## How DNS Works

The application will invoke the client side of DNS, specifying the hostname that needs to be translated. (On many UNIX-based machines, `gethostbyname()` is the function call that an application calls in order to perform the translation.)

DNS in the user’s host then takes over, sending a query message into the network. All DNS query and reply messages are sent within UDP datagrams to port 53.

The client first contacts one of the **root servers**, which returns IP addresses for **TLD(Top Level Domain) servers** for the top-level domain _com_. The client then contacts one of these TLD servers, which returns the IP address of an **authoritative server** for `amazon.com`. Finally, the client contacts one of the authoritative servers for `amazon.com`, which returns the IP address for the hostname `www.amazon.com`.

An organization’s authoritative DNS server houses these DNS records. An organization can choose to implement its own authoritative DNS server to hold these records; alternatively, the organization can pay to have these records stored in an authoritative DNS server of some service provider.

**Local DNS server** When a host connects to an ISP, the ISP provides the host with the IP addresses of one or more of its local DNS servers (typically through DHCP).  When a host makes a DNS query, the query is sent to the local DNS server, which acts a proxy, forwarding the query into the DNS server hierarchy.

<img src="https://p.ipic.vip/w9ji26.png" alt="Screenshot 2023-05-01 at 1.01.27 PM" style="zoom:50%;" />

The query sent from `cse.nyu.edu` to `dns.nyu.edu` is a recursive query, sicne the query asks `dns.nyu.edu` to obtain the mapping on its behalf. But the subsequent three queries are iterative since all of the replies are directly returned to `dns.nyu.edu`. In theory, any DNS query can be iterative or recursive.

**The query from the requesting host to the local DNS server is recursive, and the remaining queries are iterative**.

Each time the local DNS server `dns.nyu.edu` receives a reply from some DNS server, it can cache any of the information contained in the reply.

A local DNS server can also cache the IP addresses of TLD servers, thereby allowing the local DNS server to bypass the root DNS servers in a query chain. In fact, because of caching, root servers are bypassed for all but a very small fraction of DNS queries.

## DNS Records

The DNS servers store resource records (RRs) in a **zoom file**, including RRs that provides hostname-to-IP address mappings. Each DNS reply message carries one or more resource records.

A five-tuple `(Name, Value, Type, TTL, Class)` where:

| Record Type | Name       | Value                                       |
| ----------- | ---------- | ------------------------------------------- |
| A           | Hostname   | IPv4 Address                                |
| AAAA        | Hostname   | IPv6 Address                                |
| CNAME       | Alias      | Canonical Name (Hostname)                   |
| MX          | Domain     | Mail Server Hostname, Priority              |
| TXT         | Domain     | Text Information                            |
| NS          | Domain     | Name Server Hostname                        |
| SOA         | Domain     | Primary NS, Responsible Email, Serial, etc. |
| PTR         | IP Address | Pointer to Hostname                         |
| SRV         | Service    | Priority, Weight, Port, Target (Hostname)   |

`TTL` is the time to live of the resource record.

`Class` is `IN` in Internet.

If a DNS server is authoritative for a particular hostname, then the DNS server will contain a Type A record for the hostname. (Even if the DNS server is not authoritative, it may contain a Type A record in its cache.)

**If a server is not authoritative for a hostname, then the server will contain a Type NS record for the domain that includes the hostname; it will also contain a Type A record that provides the IP address of the DNS server in the _Value_ field of the NS record.**

## DNS Messages

Both query and reply messages have the same format. This semantics of the various fields in a DNS message are as follows:

<img src="https://p.ipic.vip/di59ha.png" alt="Screenshot 2023-05-01 at 1.26.17 PM" style="zoom:50%;" />