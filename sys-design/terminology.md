## Terminology

### 1. What is System Design?

System design is the process of defining the architecture, components, and interfaces of a software
system. The design of the system is driven by the functional and non-functional requirements of
users and clients.

The design process addresses these requirements by describing the main characteristics of the system:
performance, scalability, availability, and reliability.

System design is an implementation-oriented process that specifies the interactions between
components, the responsibilities of services, and the unique parts of a system.

A design can include anything from high-level system architecture to the interface of a single
component.

Understanding the advantages and tradeoffs of design choices is critical for scaling a system
and avoiding potential bottlenecks.

#### The Interview Process
A system design interview is often open-ended and unstructured, requiring the candidate, to lead
the discussion. 

A common challenge is to balance the breadth and depth of designs: you will need to both describe
a high-level system architecture and also go into deep dives about concepts and components.

The interview question can be broad with no single correct approach, and part of the interview
is gauging how a candidate handles ambiguity.

Questions may have a broad scope initially, and you should narrow the scope and direct the
discussion to showcase your expertise on specific topics.

While a coding interview is used to determine if a candidate should be hired, the system design
interview is often used to determine the level at which the candidate should be hired.

### 2. Key Terminology
This section introduces the terminology that you are likely to encounter in a system design
interview. While some terminology may have slightly different meanings at different
companies, being able to clearly define a term will let you clarify any differences in
understanding Purthermore,

Being precise in the terminology that you use during your system interview will convey a deeper
sense of understanding and help you explain complex designs.

#### Client, Server, and Machine

The terms client and server are best described in relation to each other: a client is software
that makes requests, and a server is software that responds to those requests.

The terms client- side and server-side are used to describe where software runs. Client-side
software runs on a user's device, such as a laptop, desktop, mobile phone, or tablet. Server-side
software runs on a machine that is managed for the user.

In particular, the term server is overloaded-some readers might associate the term "server"
with physical hardware (eg, "rack server"). However, we'll use the term machine
for hardware: a machine is the physical or virtual computing node that software runs on.

Usually, multiple servers (software) run on a single machine (hardware). Additionally, we'll
further disambiguate between client-side and server-side by using the term device for client-
side hardware and the term machine for server-side hardware.

### Application and Service
An application is software that performs a wide range of operations and targets an entire
problem domain. In other words, it provides a solution to a particular problem or need. An
application can encompass multiple services that each target smaller, more specific parts of
the problem domain.

Another use of the term application is to refer to a client-side application, which is software
installed by and managed by users and often handles user interactions. A client-side
application is also called a client.

A service is software that performs automated tasks or responds to requests from other
software, and it usually runs as server-side software. The methods and messages used to
interact with and invoke a service are called a service interface. While users must install an
application, a service is managed for users and accessed by other software -- the user does not
need to install it.

### Server vs. Service

A service refers to the logical functionality of software but not the physical deployment of this
functionality. Instead, this physical deployment is referred to as a server, which we can now
more precisely define:

* A server is a running software executable (the binary instance) that provides one or more services.

The difference between service and server can sometimes be confusing, but the key point to understand
is that a service refers to logical functionality, whereas a server refers to physical software deployment.

### Service Instance

A service instance is a particular deployment of a service and is deployed in a server. A single
server can run multiple different service instances, and a single service can have instances
across multiple servers.

### Example

Let's clarify these concepts in an example, A server named "server-01" is deployed on a
machine named "us-east-machine-a." This machine is hardware that is physically located in
the United States East region.

"server-01" runs an instance of "Song Service" and an instance of "Playlist Service," two
services that provide functionality for songs and playlists, respectively.

Another instance of the same binary is launched on the same machine and is called "server-02".

Though "server-01" and "server-02" are the same binary, "server-02" is
configured to run an instance of "Song Service" and an instance of "Payment Service", a
separate service that provides functionality for payments.


#### API
The term API stands for Application Programming Interface. An API defines how one
software program can interact with another. Usually, this means defining the methods,
messages, and functions that a software program can interact with or call. When this call is
performed over a network, the software that provides the API is a service

A service interface is a type of API; it is the published interface
used to invoke a service. However, the term API has a broader definition in that it's not
restricted to a service; an API can exist in a library or a local process.



### System, Distributed System, and Scalability

A system is a group of software processes that cooperate to provide services and functions to
other software. The term system is used to denote the complexity and scalability of the
software, that it consists of numerous components and is computationally an order of
magnitude more complex than what is referred to by the term software.

A system can include components other than pure software (eg, databases, load balancers, hardware
and storage) that all interact and cooperate to provide functionality and services.

For example, a multi-threaded software instance running on a single machine can be
considered a system. However, it is typically not what software engineers mean when they refer
to a "system" since it is software limited to running on a single machine

**Scalability** refers to a system's ability to change its capacity and performance in response to
increases or decreases in usage. This property is measured by how effectively a system can add
or remove resources.

While there are no agreed-upon metrics about how to measure scalability, a scalable system can handle
an order of magnitude increase in traffic or users.

For example, if the number of users that access a system increases by 10-fold, a scalable system can
add the resources to manage this increase.

A scalable system usually means that it is a distributed system.

A distributed system is a group of processes that run on different machines and communicate
through a network to provide services and functionality.

Distributed system advantages:
* Scalability: A distributed system can horizontally scale by adding more storage and compute nodes.
* Fault tolerance: A single node failure does not result in a system failure
* Performance: Distributed systems generally have lower latency and higher throughput.

Disadvantages:
* Unreliable network: Networks are inherently unreliable and communication between machines is not guaranteed.
* Data Replication: Replicating data across multiple nodes over an unreliable network can result in lag and write conflicts
* Consistency: Data is spread across multiple machines, and there can be different versions of the data.

It's not easy to scale a system without making it distributed, 
Single machines are typically not powerful enough to handle all application requirements as it scales.
