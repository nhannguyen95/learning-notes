**A router** is an intermediary network device with the role of forwarding (route) data between other devices (even other routers), based on a “geographical” addressing scheme.

---

The Internet looks like a set of intermediary devices delivering the request from your laptop to the website and then the content from the website to your laptop.

---

**A protocol** (in a networking environment) is a set of rules defining how the communication between two devices should happen.

A protocol is implemented with an algorithm (reasoning process on a device) and a set of messages exchanged between the two devices. The messages are the data exchanged between devices, whereas the algorithm is the application that will listen for that messages and that will understand them.

Every communication happening over the Internet is defined by a protocol, and in the path between your computer and the website you will have to use several protocols. Fortunately, your computer and the network devices do all of that for you.

---

The **OSI model** (or **OSI stack**) is a way of categorizing protocols in layers based on their usage. *Other two stacks* used to categorize protocols are the **TCP/IP model** and the **Hybrid model** (that merges OSI with TCP/IP). They're just 3 different ways of categorizing the protocols.

```
    [OSI]                        [TCP/IP]                  [Hybrid]
-------------                 -------------             -------------
 Application             
-------------                  
 Presentation                  Application               Application
-------------      
   Session
-------------                 -------------             -------------
  Transport                     Transport                 Transport
-------------                 -------------             -------------
   Network                       Network                   Network
-------------                 -------------             -------------
  Data Link                      Network                  Data Link
-------------                    access                 -------------
  Physical                                                Physical  
-------------                 -------------             -------------

* TCP/IP stack focuses on Network and Transport layer, it is designed so that you
do not care how you put information on the cable (it's just Network Access) and
you do not care about what are you putting on it (it's just Application data).
What you care is how that data reach the intended application.

* Hybrid stack is used when you do not care about what the application data are,
but you do care about how you access the network.

```

The client packages/encapsulates the request in top-down order, the server unpackages/decapsulates the request in bottom-up order. If the request comes to a router, a router only needs to unpackages 3 last layers so that it can identify where the request should go next.

Next we will discuss each TCP/IP layer.

### Network Access layers

These layers grant the functionalities of hardware and allows its use to upper layers.

#### Physical Layer

**Physical** layer controls how data are put on the cable for transmission and reception between a device and a shared physical medium (physical medium can be Copper, Fibre, or WIFI), it converts information in form of bits into electromagnetic signals (cables and wireless) or light pulses (fiber optic) and vice versa. It makes sure that both devices shared the understanding of the physical medium (both can use this physical medium to send and receive the data).

In order to connect multiple devices together, we can use a layer-1 networking device called Network Hub, a device which wants to join a network needs to connect to a Network Hub in that network.

Note that Physical Layer is pretty dumb:
- There are no individual device addressing, one device can't address traffic directly at another (device-to-device communication), data sent from 1 device is transmitted onto the physical medium and everything else receives it. It's a broadcast medium.
- Only one device can transmit at once on a same Layer 1 shared medium, otherwise collision occurs. Physical Layer has no collision detection and no method of controlling which devices can transmit.
- The more devices added to a Layer 1 network, the higher chance of collisions and data corruption.

Data Link Layer (Layer 2) fixes these problems by adding a lot of intelligence on top of Layer 1.

Learn more in:
- [[07-physical-layer-signals-waves-and-transmission-types]]
- [[08-physical-layer-copper-media-and-ethernet-cable]]
- [[09-physical-layer-fiber-optic-media-data-as-light-pulses]]

#### Data Link Layer

**Data Link** layer defines which cable to use, how much data can be put on the cable at the same time, how it should be formatted and also *detects if something goes wrong during the transmission*. This layer is divided into 2 sub-layers: MAC (Media Access Control) which interfaces with Physical layer and LLC (Logical Link Control) which interfaces with Network layers.

Learn more in [[12-data-link-layer]]
### Network and Transport layers

These layers allow applications to talk to each other.

**Network** layer defines addressing information of the destination device and tells that to Data Link layer so that the Data Link layer can select the right cable to reach that device (This is possible because internet-wide addressing information is processed at this layer). Protocol used is **IP (Internet Protocol)**.

**Transport** layer defines addressing information for application and services, so it can deliver the message to the right application (in the destination device, *addresses of applications are unique within the same computer*). A combination of addresses (of both source and destination device) from transport layer and network layer is called socket, and identifies a unique connection in the Internet. Protocols used are **TCP (Transmission Control Protocol)** and **UDP (User Datagram Protocol)**.

### Application layers

TCP/IP and Hybrid model includes also Presentation and Session layers in it (*This is due to the fact that often time presentation and session-related tasks are handled by the application itself*)

> An application is a web browser, a web server, an online game, an app on a smart phone showing you online content and such.

**Session** layer ensures that data reach the intended application, it implements some mechanisms that allow the **retransmission** of the data in case something gets lost, and *TCP is a transport protocol that handles retransmissions*, therefore it works also at the session layer. UDP has no mechanism to recover data lost in the path (it is considered unreliable), so if we want to use UDP but we also want to know if something gets lost we can use **RTP (Real Time Protocol)** at the session layer.

> This protocol is commonly used for streaming traffic, because streaming is real-time in nature and it needs to be fast, we do not have time for retransmissions.

**Presentation** layer defines information about how to compress and present your content, mostly used for real-time traffic such as voice or video streams, *it is in charge of handling ow data is presented to the application*.

**Application**: We cannot say anything specific about this layer, simply because each application is different and defines its own protocol(s). However, many applications are standard HTTP or HTTPs traffic. 

>  There are tons of other protocols working at this layer, such as **SMTP**, **POP3** and **IMAP** for emails
