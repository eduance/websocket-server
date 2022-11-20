# websocket-server

### Internet protocol (IP)

The internet is basically a network of smaller networks that are woven together. We use IP-addresses to tell
smaller networks how to communicate together. Imagine having three points: each point representing an individual
network. How would you connect these? 
Using wired cables obviously, but how would this look like if you had 6 networks?
Each network would need a cable from and to another network. Point A would need a cable from and to point B, point B needs a cable to point C and so on.

That is where routers come into play. Routers contain a table representing routing information, and each router
knows where it is connected to. If router A has a direct connection to network 1 and network B sends a request
out to router B. Router B will instantly be able to tell: I am not connected with network one, but I know
a router who is connected. So it directly links to router A.

An IP-address is responsible for getting information from point A to point B.

### Transmission control protocol

Well, imagine landing your airplane at a busy airport. Your airplane named KLM A lands at Schiphol and you
are completely free to choose a gate! Now imagine Turkish Airlines doing the same. And American Airlines as well.
This would quickly become a mess, how would baggage handlers know where to bring the right bags? 
Airtraffic control is therefore also responsible for assigning each plane where to park. 

TCP will have the same job. IP is responsible for moving information from a sending machine to a receiving machine.
TCP is responsible for getting information to the right program/service on a machine. The nice thing is: just like
airtraffic controllers, TCP will always guarantee delivery. Now we know that if you combine TCP and IP together,
it will make a very powerful relationship and we know where to route information to and where it can be parked!

### Hyper Text Transfer Protocol

Imagine us both meeting for the first time, what do you do? Well, I would would politely introduce myself
and shake your hand. I would tell you that my name is Brandon, and you would say your name.

Let's think of another set of conventions, in fact, let's make up our own set of conventions. When getting
delivered a pizza, instead of tipping the delivery guy, let's just jump happy up and down, as a rule: the delivery
guy has to also jump happy up and down.

Just like shaking hands and jumping up and down, this is a protocol. It is a set of conventions we do. The official
HTTP protocol is:

```
A client sends a request to a server in form of a request method, URI and protocol version, 
followed by a MIME-like message containing request modifiers, client information, and possible body content over a connection with a server. 
The server responds with a status line, including the message’s protocol version 
and a succes or error code, followed by a MIME-Like version containing server info, 
entity meta info, and possible entity-body content.
```

When HTTP was released back in 1991, most webpages were simply static. As time went by, some attempts were made
to make HTTP work with realtime communication, most described well by: Mobile HTML5: Efficiency and Performance of WebSockets and Server-Sent Events by Eliot Estep from Aalto University
This paper will enlighten you about different subjects such as polling and long-polling and their problems.

Feel free to dive into my notes about these techniques: [click here](https://elated-radius-fa4.notion.site/Networking-9134b43003d240c48a699fc53b1303f8).

### Sockets

Berkeleys sockets, often referred to as sockets are an API for internet sockets and unix domain sockets used for inter-process communication.
Simply put, sockets are an abstraction for an application to bringing data from A to B.

I won't go into the details of Stream sockets and Datagram sockets, but just know that we'll use TCP sockets
as they are reliable and provide byte-stream service.

### The WebSocket Protocol

[RFC6455](https://www.rfc-editor.org/rfc/rfc6455#section-1.1) tells us the need for websockets and explains us that the need for bidirectional communication
has required an abuse of HTTP to poll the server for updates while sending upstream notifications.

This lead to some serious problems:

* The server is forced to use a number of different underlying TCP connections for each
client: one for sending information to client and a new one for each message.

* The wire protocol has a high overhead.

* The client-side script is forced to maintain a mapping from outgoing connections to the incoming connection to track replies.

Websockets solve this, by providing one single TCP connection for traffic in both directions! Combined with the [WebSocket API](https://websockets.spec.whatwg.org//), it
provides an alternative to HTTP polling for two-way communication.

## How do we make a connection?

To set up a successful connection, let's look at what we have to do.

We will first look from the perspective of a server.

1. We first create a socket, as PHP is based on C, we'll mostly look at how C handles this, and then follow up with the PHP implementation which will
be almost identical.

```
int socket (int namespace, int style, int protocol)
```

When creating a socket, we are asked a namespace (php refers to this as domains), a style and a protocol.
A namespace can be both PF_LOCAL (for when we intend to use it locally) or PF_INET when connecting to a IPv4 network.

A style refers to the communication style, which defines the user-level semantics of sending and receiving data on the socket.

What data will we send? Do we send data as a sequence of bytes, or do we group the bytes into records? (packets)
Can data be lost? It sometimes may be worth to lose a bit of data.
Is communication with one partner or will we send it bulk?

AF_ is used for address families and PF refers to a protocol family.
Fun fact is that it doesn't matter if you use AS or PF, as they refer to the same values.

2. We will now bind sockets to an address. Sockets names are called addresses. Socket addresses were named
inconsistently in C, so you'll see "name" and "address" both get mixed up, they are the same for where sockets are involved.

```int bind (int socket, struct sockaddr *addr, socklen_t length)```

We will refer the created socket and bind it, addr and lengths arguments
specify the address  and the length is the length of the address structure which is a custom datatype
for both address and port.

3. We will then listen for connections.

```int status = listen(sockid, queueLimit);```

The sockid is the socket descriptor and the queuelimit is the number of how many
connections that can wait for connections.

4. Connect 

```int status = connect(sockid, &foreignAddr, addrlen);```

We need a socket ID, foreign address (address of participant) and the length.

5. Accept the connection!

```int s = accept(sockid, &clientAddr, &addrLen);```

The server's original socket does not become part of the connection, as this will mean that our initial TCP
request would be lost (this is the problem with polling, no please!), instead, accept makes a new socket which
participates in the connection. The server's original socket will gladly listen for new people joining our crew!

6. Send

```int count = send(sockid, msg, msgLen, flags);```

7. Receive!

```ssize_t recv (int socket, void *buffer, size_t size, int flags)```