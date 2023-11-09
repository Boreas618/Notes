# The Web and HTTP

The World Wide Web is an Internet application.

**Uniform Resource Identifier** (统一资源标识符): Identify resources

**Uniform Resource Location** (统一资源定位符): A common form of URI

![image-20231014141425346](https://p.ipic.vip/3r9vmy.png)

HTTP(Hyper Text Transfer Protocol) is the Web's application-layer protocol. It is implemented in two programs: a client program and a server program. The client and server exchange HTTP messages.

A Web page consists of objects. A HTML file references the other objects in the page with the object's URLs.

**HTTP uses TCP as its underlying transport protocol.**

Because an HTTP server maintains no information about the clients, HTTP is said to be a **stateless protocol**.

## Non-Persistent and Persistent Connections

Although HTTP uses persistent connections in its default mode, HTTP clients and servers can be configured to use non-persistent connections instead.

### HTTP/1.0 Non-Persistant Connections

**Example:**

```
http://www.someSchool.edu/someDepartment/home.index
```

1. The HTTP client process initiates a TCP connection to the server on port number 80. (**1 RTT**)
2. The HTTP client sends an HTTP request message to the server via its socket. The request message includes the path name `/someDepartment/home.index`.
3. The HTTP server process receives the request message via its socket, encapsulates the object in an HTTP response message, and sends the response message to the client via its socket.
4. The HTTP server process tells TCP to close the TCP connection.
5. The HTTP client receives the response message. The TCP connection terminates.
6. The first four steps are then repeated for each of the referenced JPEG objects.

**N + 1 TCP connections are generated** (N is number of resources). The time of reaction is **(N+1)$\times$ 2$\times$RTT + Transmission Time**. (Note that N + 1 counts the web page in)

In their default modes, most browsers open 5 to 10 parallel TCP connections, and each of these connections handles one request-response transaction.

### HTTP/1.1 Persistent Connections

**2 TCP connections are generated** (N is number of resources). The time of reaction is **(2 + N)$\times$RTT + Transmission Time**

Typically, the HTTP server closes a connection when it isn’t used for a certain time (a configurable timeout interval)

**Pipeling**: Subsject to **head of line blocking**.

## HTTP Message Format

```http
GET /somedir/page.html HTTP/1.1
Host: www.someschool.edu
Connection: close
User-agent: Mozilla/5.0
Accept-language: fr

```

Request line and header lines

`GET` is **case sensitive**.

`Connection: close` means non-persistent connection.

<figure><img src="https://p.ipic.vip/95tmpc.png" alt="" width="375"><figcaption></figcaption></figure>

`HEAD` leaves out the requested object. For debugging.

`PUT` upload something to the server.

The real URL accessed is `www.someschool.edu/somedir/page.html`. **When using a proxy, the URI in the request line contains the complete URI, including the authority (host and port number) part.**

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

**HTTP/1.1** specifies that the **Host** filed in the header is mandatory. All other header fileds are optional. 

Status codes:

1XX - informational, not the final response yet

2XX - success

3XX - redirection, retrieve from another location or cache

4XX - client error

5XX - server error

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