---
layout: post
title: "Docker Network File Permissions"
meta: "How running inter-process communication (ipc) with Docker Swarm led to an insight into file permissions on bridge and overlay networks."
categories: Node JavaScript Docker
slug: "docker-network-file-permissions"
---

# Docker Network File Permissions
I have been creating an 'artificial' [Node.js][node] application for the purposes of investigating the properties and behaviour of Docker [Swarm mode][swarm-mode]. I don't mean 'artificial' as in 'intelligence' but rather as in created for no other purpose than my to enable my ongoing investigations.  The application doesn't do anything that would be of use to anyone else but it does enable me to try out some ideas.  Forgive me for being cryptic but I'll explain this application (lets call it the **swarm message queue** app or **SMQ** from now on) in more detail in a later post.  The purpose of this post is to share some findings regarding Docker networks and file permissions.

## Message Queues and Inter-Process Communication (IPC)
As part of the **SMQ** I'm using various types of [ZeroMQ][zero-mq] sockets. Why? Because I want to learn more about ZeroMQ (other message queues are available). It was in the course of using a [Dealer-Router][dealer-router] pair that I discovered something interesting about Docker [overlay networks][overlay-network] and file permissions.

The Dealer-Router pair is used to enable parallel message processing. The Router socket can handle many requests simultaneously. It remembers which connection each request came from and will route reply messages accordingly.  The Dealer, on the other hand, can send multiple requests in parallel.  Incoming requests to the Router will be passed off to the Dealer to send out to its connections.  Likewise, incoming replies from the Dealer will be forwarded back to the Router, which directs each reply back to the connection which requested it.

So far, so good. The issue arises due to the protocol used by the Dealer.  The Dealer connects to its downstream sockets (in the case of **SMQ** these are Reply sockets) using [Inter-Process Communication][ipc] (IPC). IPC is a mechanism used by the operating system to allow processes it manages to share data. The Dealer pair is connected like so:


<script src="https://gist.github.com/paulnebel/40f440bb52616e4d41d00388709b6005.js"></script>


This has the effect of creating the file `filer-dealer.ipc` in the directory the current working directory of the current Node process.

As mentioned above, the **SMQ** exists purely for the purposes of investigating Docker Swarm mode. Note that [Docker Swarm][swarm] and [Swarm mode][swarm-mode] are two different things - Docker Swarm has been available since release 1.6 but Swarm mode was only introduced in release 1.12. Whereas Docker Swarm came in 'kit form' (with the Swarm runtime running as a container on each of the nodes in the Swarm) Swarm mode comes with a core feature-set build in. Docker Swarm required an external discovery service whereas Swarm mode does not.

In order to investigate Swarm mode I want the **SMQ** to run as multiple services on multiple Docker hosts on multiple physical servers. In order to get to this point I have so far achieved the following steps:

 1. Multiple Node.js processes; each process running on the same host; host running on a single physical server; communication via the `localhost` network.
 2. Multiple Node.js processes; each process running in its own Docker container; all containers running in the same Docker host; Docker host running on a single physical server; communication via a [bridge network][bridge-network].
 3. Multiple Node.js processes, each process running in its own Docker container; each container running in its own Docker host; all Docker hosts running Swarm on a single physical server; communication via an [overlay network with an external key-value store][overlay-network].  

It was the transition from step 2 to step 3 that caused me difficulty, for the reasons explained below.

### Multiple Node.js Processes
In particular, it is the interaction between two of the **SMQ** processes that presented a challenge. The first process (`server.js`) consists of a ZeroMQ Request socket which is making multiple requests to the second process.  The second process ('dealer-router.js') is running a Router/Dealer socket pair that, in turn, marshall requests to multiple Reply sockets (currently running in the same process, but eventually to be running in a separate container on a separate host).

The Request socket in `server.js`, being the more 'stable' part of the architecture, binds to an arbitrary TCP port (in this case `5432`) using a wildcard, meaning it's listening on that port on all available networks:

<script src="https://gist.github.com/paulnebel/1411af23aeb603b1cde0b1ffb2ad9f89.js"></script>

The Router socket in `router-dealer.js`, being the more 'transient' part of the architecture, connects to the same TCP port on the `localhost` network:

<script src="https://gist.github.com/paulnebel/282e1b02de1bed0be7232fa1571664c0.js"></script>

There's no strict rule as to which end of a communicating socket pair should `bind` and which should `connect` - both sockets are capable of either type of connection. Nor does it matter which end of the connection is started first. It was relatively trivial to get the Request socket communicating with a Responder socket in a different process using this configuration.

### Multiple Containers Running Node.js Processes on a Single Docker Host




   [node]: <https://nodejs.org/en/>
   [swarm-mode]: <https://docs.docker.com/engine/swarm/>
   [zero-mq]: <http://zeromq.org/>
   [dealer-router]: <http://zeromq.org/tutorials:dealer-and-router>
   [overlay-network]: <https://docs.docker.com/engine/userguide/networking/get-started-overlay/>
   [ipc]: <http://www.itproportal.com/2010/07/03/beginners-guide-inter-process-communication-ipc/>
   [swarm]: <https://www.docker.com/products/docker-swarm>
   [bridge-network]: <https://docs.docker.com/engine/userguide/networking/#/a-bridge-network>
   [overlay-network]: <https://docs.docker.com/engine/userguide/networking/#/an-overlay-network-with-an-external-key-value-store>