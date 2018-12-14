---
layout: post
title: Network Sockets
---


### Abstractions All the Way Down


A network socket, like all entities and concepts in software, is an abstraction. A network socket is a software representation of a point in a computer network that is internally exposed to processes, such as a Java application, by a computer's operating system. The OS exposes this endpoint through networking software which is referred to as a protocol stack and provides a set of system calls that allow a user to send and recieve data to other endpoints in the network that have connected to the network socket. 


There are many layers of abstraction between the software we wield as developers, and the eventual physical electrons that get sent across copper or fiber optic wires to enable communcation across a computer network. Operating systems also use sockets to allow different processes within a computer to talk to one another in a process known as inter-process communication.


### Socket Address


A network socket is a representation of a communcations endpoint in a network. This network can be local or remote, and the representation is most often referred to using an _address_. The first stage in establishing communcation to other nodes in a network is the request that the protocol stack creates a socket at a specific address. 


A socket address is composed of both an IP address that identifies the computer it has been created on and a port that represents a single endpoint within that computer. Ports are an abstraction that allow a computer to establish multiple connections to various different networks.


### Socket Descriptor


When the protocol stack creates a socket it provides a process (a running application) with a socket _descriptor_ which is a single unique value that the application can use to identify to the protocol stack which socket it would like to send or receive from. The protocol stack can then identify the IP address and port that the socket descriptor refers to and perform the appropriate action. This leaves applications unaware of the details of where or how they will communicate with a socket, whether it's connected locally or remotely, or to a single client or many. All the application is aware of is the single descriptor.


This degree of seperation also means that the local protocol stack does not have access to foreign socket descriptors, these are internal to that socket representation that is managed by it's own remote protocol stack. 


### Two Way Communication


When a process at a local IP address requests to communicate with a socket at a foreign IP address on a certain port the local protocol stack creates a socket, returns its descriptor to the process, and attempts to establish a connection to the foreign address. That foreign protocol stack receives the request, and subsequently creates it's own socket in order to facilitate the communication. The process of associating a socket to an address is known as _binding_.


This connection is only established if a foreign process(es) has requested to the foreign protocol stack that it would like to listen for connection requests on the address and port that the request has been sent to. The listening process will then receive a socket descriptor from the protocol stack if a connection request has succeded and enable the foreign process to send and recieve data through it's own local socket to the socket managed by the protocol stack on the other side of the network.


Once the local socket has been created and a socket descriptor has been returned, the local process can simply reference the socket descriptor and the protocol stack will handle sending and receiving data to and from the foreign socket at the foreign IP address at the specific port.


A socket exposes both its input and output which represent the ability to ask the protocol stack to send data through the network (write to the socket) or receive data that has been sent to the socket (read from the socket). In order to facilitate two way communication, a process must reference both of these data streams.


### Summary


There is a nuance in differentiating socket addresses, descriptors, and the socket itself. 
To summarize:


1. A socket is the internal (software) representation created by the protocol stack, that allows it to send and receive data across a network.

2. A socket address is made up of an IP address (the address of the host computer) and a port (the specific connection within that host) and is used when a protocol stack wants to communicate with a socket on another network or facilitate communcation between local processes.

3. A socket descriptor is the identifier that allows a process running on an operating system to communicate with that operating system's protocol stack. It only has meaning within the a single computer and cannot be used to reference other network sockets.


Sockets allow network connections to be abstracted away from the processes that wish to speak to other processes. This abstraction applies whether the other process is halfway across the world or running on the same computer. The protocol stack software exposed by the OS handles these details for us and allows higher level protocol's like HTTP to run on these abstractions so we can talk to each other over a variety of networks using a variety of mediums!
