# Network Programming

It is important to realize that clients and servers are **processes** and not machines, or *hosts* as they are often called in this context.

## Networks

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
// inet stands for internet, p stands for presentation, n stands for network
// returns: 1 if OK, 0 if src is invalid dotted decimal, −1 on error

const char *inet_ntop(AF_INET, const void *src, char *dst, socklen_t size);
// returns: pointer to a dotted-decimal string if OK, NULL on error
```

`AE_INET` for IPv4, `AE_INET6` for IPv6.

### Internet Domain Names

<img src="https://p.ipic.vip/gx2rzp.png" alt="Screenshot 2023-05-03 at 12.09.24 PM" style="zoom:50%;" />

### Internet Connections

Full duplex: the data can flow in both directions

A **socket** is an end point of a connection. Each socket is identified by the socket address `address:port`.

The port in the server’s socket address is typically some *well-known port* that is permanently associated with the service. The mapping between well-known names (smtp, http) and well-known ports (25, 80) is contained in a file called `/etc/services`.

By default, HTTP requests use port 80 for non-secure (HTTP) connections and port 443 for secure (HTTPS) connections. However, if the server is running on a different port, you can specify the port number in the request URL, like `http://fduhole.com:8080/`.

A socket tuple: `(cliaddr:cliport, servaddr:servport)`



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

### Sockets With Protection

We folk a child process to server the client in the server.

<img src="https://p.ipic.vip/kn2yke.png" alt="Screenshot 2023-05-27 at 7.27.47 AM" style="zoom:50%;" />

```c
while(1) {
  // Accept a new client connection, obtaining a new socket
  int conn_socket = accept(server_scoket, NULL, NULL);
  pid_t pid = fork();
  if (pid == 0) {
    // when we fork, the child gets all of the parents' file descriptors
    // but the server_socket is not needed in child
    close(server_socket);
    serve_client(conn_socket);
    close(conn_socket);
  } else {
    // same as above, the conn_socket is not needed in parent
    close(conn_socket);
    wait(NULL);
  }
}
close(server_socket);
```

### Concurrency

```c
while(1) {
  // Accept a new client connection, obtaining a new socket
  int conn_socket = accept(server_scoket, NULL, NULL);
  pid_t pid = fork();
  if (pid == 0) {
    // when we fork, the child gets all of the parents' file descriptors
    // but the server_socket is not needed in child
    close(server_socket);
    serve_client(conn_socket);
    close(conn_socket);
  } else {
    // same as above, the conn_socket is not needed in parent
    close(conn_socket);
    // THE ONLY CHANGE IS HERE!
    //wait(NULL);
  }
}
close(server_socket);
```

Use threads, without Protection:

![Screenshot 2023-05-27 at 7.40.23 AM](https://p.ipic.vip/ekskk3.png)

Problem with this: unbounded threads. When the website becomes too popular, there's throughput sink.

Instead, we allocate a bounded "pool" of worker threads, representing the maximum level of multiprogramming.

```rust
extern crate threadpool;

use threadpool::ThreadPool;
use std::sync::mpsc::channel;

fn main() {
    // Create a thread pool with 4 workers
    let n_workers = 4;
    let n_jobs = 8;
    let pool = ThreadPool::new(n_workers);

    // Create a channel for transmitting job results
    let (tx, rx) = channel();

    for _ in 0..n_jobs {
        let tx = tx.clone();
        pool.execute(move|| {
            // Do some work, send result to channel
            let result = 2 * 2;  // example of some work
            tx.send(result).expect("channel will be there waiting for the pool");
        });
    }

    // Collect job results
    for _ in 0..n_jobs {
        println!("{}", rx.recv().unwrap());
    }
}
```



