
#### Structure of Network System

|          Layer           |                                        Function                                        |
| :----------------------: | :------------------------------------------------------------------------------------: |
| Application<br>(Highest) |           Establish an idiom for communicating with a particular application           |
|        Transport         |                       Establish endpoints useful to a programmer                       |
|         Network          |      Given multiple inter-connected LANs, achieve cross-connectivity (Scaling up)      |
|        Data Link         | Manage the channel to enable actual communication, i.e. establish a Local Area Network |
|  Physical <br>(Lowest)   |                  Establish a channel with connectivity and signaling                   |

##### 1. Physical
We have a functioning physical layer once we can send and receive signals. Two concepts:
1. **Bandwidth.** Number of bits transferred per second. Limited at *local* scale; can be improved with parallelism or faster clock rate.
2. **Latency.** A function of signal propagation speed. Limited at *global* scale by speed of light. 
##### 2. Data Link
This layer is about the *metadata* of the physical layer, i.e, who and when we start transmitting signal. We have a functioning link layer once we can build a functioning local area network (LAN) of at least two stations. 
##### 3. Network
We have a functioning network layer once we can connect multiple networks, identify **hosts** among them, and messages can find their way across networks from source to destination.
##### 4. Transport
The transport layer exists once we have the ability to establish communication from end-point to end-point with well-understood properties.
##### 5. Application
Application protocols exist when applications can communicate. For example, HTTP requests `Get` and `POST`.

#### Global IP Internet
The internet can be abstracted as a collection of hosts with several properties. These hosts are associated with other internet concepts, given by the following:
- The set of **Internet hosts** are mapped to a set of 32-bit **IP addresses.** For example, the local host `127.0.0.1`.
- The set of **IP addresses** is mapped to a set of identifiers called **Internet domain names.** They are the things that appear in our browser tab. For example, `128.2.217.3` is mapped to www.cs.cmu.edu.
- A **process** on one Internet host can communicate with a process on another **Internet host** over a **connection.**

Let us first unveil the myth of IP addresses. 
##### 1. IP Address
>[!note] TCP/IP Protocol Family
>- **IP (Internet Protocol)** provides basic ***naming scheme*** and ***unreliable*** delivery capability of **packets** (header + payload like blocks in malloc) from ***host-to-host.***
>- UDP (Unreliable Datagram Protocol) uses IP to provide ***unreliable*** datagram delivery from ***process-to-process.***
>- TCP (Transmission Control Protocol) uses IP to provide ***reliable*** byte streams from ***process-to-process*** over **connections.**
>
>***Reliable*** means that stream of bytes sent by the source is eventually received by the destination in the same order it was sent. An ***unreliable*** connection delivers data faster than a reliable connection, but it does not provide any guarantee for data delivery.

- IP addresses are always stored in memory in big-endian byte order. It is stored in the order we read.
- Each byte in a 32-bit IP address is represented by its decimal value and separated by a period. For example, `0x8002d903` maps to `128.2.194.242`. Checkout this [converter](https://www.browserling.com/tools/ip-to-hex).
##### 2. Internet Domain Names
The Internet maintains a ***mapping*** between IP addresses and domain names in a huge worldwide distributed database called **DNS**.
##### 3. Sockets and Internet Connections
Clients and servers communicate by sending streams of bytes over **connections**. A **socket** is an endpoint of a connection. Each socket is identified by **socket address**, which is an `IPaddress:port` pair. A **port** is a 16-bit integer that identifies a process. There are two types of ports:
- **Ephemeral port.** Assigned automatically by client kernel when client makes a connection request.
- **Well-known port.** Associated with some service provided by a server. For example, port 80 is associated with web servers.

A connection is ***uniquely*** identified by the socket addresses of its endpoints. The 2 endpoints are also referred to as *connection socket pair.* To build a network applications, we need the help of **the sockets interface.**

#### The Sockets Interface
The sockets interface is a set of ***system-level functions*** used in conjunction with Unix I/O to build network applications. Note that in network applications, sockets are **file descriptors** that let the application read/write from/to the network. 
- Therefore, in [proxy lab](https://github.com/cmu15213-s24/proxylab-s24-gpxxlt/compare/8f6f830d3db5a5981f613e9b48554df63c7a9bd0...1ca76c37017bf8b40b3a7b2ccc5c7065d5554319), we have `listenfd` to store proxy's incoming request, `connfd` as an intermediary, and `proxyfd` as the server side file descriptor.

#### Web Servers
##### 1. Web Content
After accepting client's **HTTP request**, web servers return **content** to clients. Content is defined as a sequence of bytes with an associated **MIME** ([Multipurpose Internet Mail Extensions](**http://www.iana.org/assignments/media-types/media-types.xhtml**)) type. For example, HTML document, plain text, png/jpeg image formats. There are two types of content:
- **Static content.** Content stored in files and ***retrieved*** in response to an HTTP request. For example, HTML files, images, audio clips.
- **Dynamic content.** Content produced ***on-the-fly*** in response to an HTTP request. For example, content generated by executing server-side code.

>[!info] Side Note: Serving Dynamic Content with Common Gateway Interface (CGI)
>A typical method of serving dynamic content is as follows:
>1. The server creates a child process and runs the program (called **CGI programs**) identified by the URI in that process.
>2. The child runs and generates the dynamic content.
>3. The server captures the content of the child and forwards it without modification to the client.

The CGI is the original standard for generating dynamic content. It dictates a way to pass the arguments to the child process to run the executable. The arguments are simply appended to the URI following some rules.
- The example URL is `http://add.com/cgi-bin/adder?15213&18213`
	- CGI program on the server `adder` will be executed with the given arguments.
	- Argument list starts with `?`
	- Arguments separated by `&`
	- Spaces represented by `+` or `%20`
##### 2. URL
Web content is identified through **URL** (Universal Resource Locator). It is informative of metadata of the connection and its end points; client and server use it differently. 
- Example URL `http://www.cmu.edu:80/index.html`
- Clients use prefix `http://www.cmu.edu:80` to infer:
	1. Communication protocol `http`
	2. Domain of the server `www.cmu.edu`
	3. Listening port number `80`
- Servers use suffix `/index.html` to:
	1. Determine if request is for static or dynamic content; executable typically in `/cgi-bin` directory
	2. Find file on file system by the path `/index.html`

##### 3.  HTTP Requests and HTTP Responses
Clients and servers communicate using the HyperText Transfer Protocol (HTTP). Clients send HTTP requests to the server. Upon receiving the request (or fail to), the server send back HTTP responses.
- **HTTP request** is a **request line** followed by zero or more **request headers.** 
	- Request Line. Contains HTTP method, URL or URL suffix, and HTTP version of the request. `<method> <uri> <version>` 
	- Request Headers. Provide additional information to the server, like user agent.                        
	  `<header name>: <header data>`
- **HTTP response** is a **response line** followed by zero or more **response headers**, possibly followed by **content**, with blank line `\r\n` separating headers from content.
	- Response Line. Contains HTTP version, status code like `404`, and its English meaning "not found". `<version> <status code> <status msg>`
	- Response Headers. Provide additional information about response, for example, MIME type and length of content.

In my implementation of the [proxy](https://github.com/cmu15213-s24/proxylab-s24-gpxxlt/compare/8f6f830d3db5a5981f613e9b48554df63c7a9bd0...1ca76c37017bf8b40b3a7b2ccc5c7065d5554319), the HTTP requests sent by the client is processed and packaged into `request_msg` and sent to the destination server.