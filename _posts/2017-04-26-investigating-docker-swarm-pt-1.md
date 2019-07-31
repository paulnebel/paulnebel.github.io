---
title: "Investigating Docker Swarm"
meta: "Take a Node.js application designed to exercise Docker Swarm and run it in a multi-host Docker environment."
categories: Node JavaScript Docker
slug: "create-multi-host-docker-app"
--- 

## Creating a Multi-host Docker Application

I have recently become very interested in [Docker Swarm][docker-swarm], in particular with [Swarm mode][swarm-mode], and I wanted to investigate its capabilities further in order to understand how I might justify its use in a production environment (if you're unsure exactly what Swarm is I wrote [a post][blog-what-is-swarm] about it that might help). I even went so far as to [create my own Raspberry Pi Cluster][blog-rpi-cluster] in order to experiment with Swarm.  A search for Docker Swarm will turn up many useful articles explaining how to create a simple swarm cluster but there seem to be relatively few articles that investigate it in greater detail. I decided to create an application that allowed me to exercise all the available features of Swarm in order to become familiar enough with it to use it with confidence in a production context.

This article isn't about Swarm itself but is about the preparatory process of turning a Node.js application into a set of containerised processes that can be distributed across a number of Docker hosts. Once I have achieved this goal I can begin my investigations.

## The Application
I have created an [application][my-app] that I will use to investigate Swarm features.  It is written in [Node.js][node.js] and consists of separate processes that communicate over sockets and tcp using message queues. It isn't intended to solve a particular problem but simply to allow me to learn. There are 3 main elements to this application which I will briefly explain.

### 1. The Web API
The first part of the application is the **web API**, shown below:
![web api][api-pic]{: .center-image .med-image }

This consists of a server written in [Express][express] that listens on port 3000.  The server creates a [Socket][node-net-socket] client that sends and receives messages over the network.

### 2. The Request Cluster
The Socket connected to the web API communicates with the **Request Cluster** which uses [Node.js cluster][node-cluster] to create multiple child processes. The master process runs a [Socket Server][node-net-socket-server] which listens for messages from the web API. The master process is connected to a number of worker processes, each of which runs a [ZeroMQ][zero-mq] requester. ZeroMQ is a highly-scalable, low-latency messaging service.  I used it simply because I wanted to learn more about it and because it allows me to create tcp messages.  The ZeroMQ Requester uses the REQ/REP (request/reply) pattern commonly used in networked programming.  In this pattern a request comes in then a reply goes out.  Additional incoming requests are queued and later dispatched by ZeroMQ.  The application is, however, only aware of one request at a time.

![request cluster][request-cluster-pic]{: .center-image .med-image }

When the Socket Server receives a message it triggers the requester in each worker process to send a message over tcp.  One of the features of Node.js clusters is that child processes share the server ports, meaning that each of the workers can send a request on the same port.  These requesters communicate with the last element of the application, the **Response Cluster**.

### 3. The Response Cluster
Like the **Request Cluster**, the **Response Cluster** consitsts of a Node.js cluster.  In this case, the master process runs a ZeroMQ ROUTER/DEALER pair while the worker processes run ZeroMQ Responsers. The standard REQ/REP socket pair operates sequentially, meaning that a given reqeuster or responder will only ever be aware of one process at a time.  What I want is parallel message processing for which I require a ROUTER/DEALER pair.

The ROUTER can be thought of as a parallel REP socket. Where REPs can only reply to one message at a time the ROUTER can handle many requests simultaneously. It remembers which connection each request came in on and will route reply messages accordingly.

If a ROUTER socket is a parallel REP socket then the DEALER is a parallel REQ socket.  It can send multiple requests in parallel. Incoming requests to the ROUTER will be passed off to the DEALER to send out to its connections.  In this case the DEALER is connected to multiple worker processes each running a ZeroMQ Responder.
![response cluster][response-cluster-pic]{: .center-image .med-image }

These responders can be made to complete asynchronous tasks and report their success. In the first instance, the responders in the **Response Cluster** are reading the text from a file and sending it back to the requester.

## Putting it Together
What I have ended up with is an application in which a web server listens to requests. When a request is received it sends a message over a socket to a client which triggers multiple requesters to send tcp messages.  These messages are received by a ROUTER/DEALER pair which forwards them to multiple responders.  In the first iteration of this application all the responders do is to read the text from a file and return it.  In order to simulate latency each responder implements a random delay of between 1 and 3 seconds before responding. The application is entirely artificial and only exists for the purposes of exercising Docker Swarm.  The number of worker processes in both the request and response clusters is configurable, as is the delay in responding.  Ultimately I'll configure the responders to do more advanced things like write do a database but I'm keeping it simple for now.

I created a very simple web page to run the application. It consists of two buttons, one for connecting the web server to the request cluster socket and one for making a request.  When a request is triggered the page displays information about the messages sent and received.
![simple GUI][web-gui-pic]{: .center-image .med-image }

I used Node.js clusters to send requests in parallel and to respond to requests in parallel.  I could have done this in other ways but I want to compare the behaviour of Node.js clusters to Swarm clusters.

## Transition from single host to multiple hosts
In its initial incarnation my application is running as 3 separate Node.js processes communicating over the localhost network.
![single host node][single-host-node-pic]{: .center-image .med-image }

I can run the 3 separate processes in my application on a single host (my Mac) using [NPM scripts][npm-scripts] to run [Gulp][Gulp] tasks:
{% highlight bash %}
$ npm run gulp:start:requester
$ npm run gulp:start:responder
$ npm run gulp:start:server
{% endhighlight %}

What I want to do is to run each process in its own container, each on a separate Docker host.  I'll do it in two stages.

## Stage 1: Multiple Containers, Single Docker Host
The first stage is to create a container for each process and host them on the same machine, in the same Docker host as shown below. 
![single host docker][single-host-docker-pic]{: .center-image .med-image }

The first thing to do is to create a [`Dockerfile`][Dockerfile] that will generate the containers:
{% highlight bash %}
# Version: 0.0.2

FROM risingstack/alpine:3.4-v6.9.1-4.1.0

MAINTAINER Paul Nebel "paul@nebel.io"
ENV REFRESHED_AT 2016_07_23
LABEL name="Base dev image for zeromq"
LABEL version="1.1"

# Add the relevant packages so that we can 'npm install zmq'
RUN apk update -q &&\
    apk add --no-cache make gcc g++ python zeromq zeromq-dev &&\
    apk upgrade

# Create "dogfish" user
RUN addgroup appuser &&\
    adduser -G appuser -g "App User" -h /home/dogfish -s /bin/ash -D dogfish &&\
    chown -R dogfish:appuser /usr/local

# Set up some semblance of an environment
WORKDIR /home/dogfish
ENV HOME /home/dogfish
RUN mkdir /home/dogfish/app &&\
    mkdir /home/dogfish/.npm-global &&\
    chown -R dogfish:appuser /home/dogfish &&\
    echo "export PATH=~/.npm-global/bin:$PATH" >> /home/dogfish/.profile

USER dogfish

RUN npm config set prefix '~/.npm-global' &&\
    source /home/dogfish/.profile &&\
    npm install -g npm &&\
    npm install -g nodemon &&\
    npm config set python /usr/bin/python &&\
    npm cache clear

WORKDIR /home/dogfish/app
VOLUME /home/dogfish/app
{% endhighlight %}

I used the excellent [Node.js image created by Rising Stack and based on Alpine Docker][risingstack-docker] as the base for my own image. This uses the [ash][ash] shell rather than bash, which requires a little more juggling to get what I wanted. As an example, ash comes without a `sudo` command so I had to change NPMs default directory to one local to the user I created in order to avoid getting [`EACCESS` errors when installing modules][npm-eaccess].

The next thing to do is to create a [`docker-compose.yml`][docker-compose] file to create and run my containers:
{% highlight bash %}
version: '2'

services:
 requester:
  build: .
  volumes:
   - $PWD/..:/home/dogfish/app
  networks:
   - zerobridge
  command: sh ./shell/scripts/startup.sh ${NODE_ENV} ${NODE_PATH} ./services/requesterManager.js target.txt

 responder:
  extends:
   service: requester
  command: sh ./shell/scripts/startup.sh ${NODE_ENV} ${NODE_PATH} ./services/responderManager.js

 server:
  extends:
   service: requester
  ports:
   - "3000:3000"
  depends_on:
   - responder
   - requester
  command: sh ./shell/scripts/startup.sh ${NODE_ENV} ${NODE_PATH} ./services/server.js

networks:
 zerobridge:
{% endhighlight %}

When developing in Docker I map the local directory containing the code to be run to a volume in the container. The Node.js processes run using Gulp communicated over the `localhost` network using an IP address of `127.0.0.1`. In order for the processes running in the containers to communicate with each other I created a [bridge network][docker-networking] called `zerobridge` to achieve the same effect. Containers running on the same bridge network can communicate between themselves directly using the service names they have been created with. This is accounted for in the [configuration file][config-file].  Thus the web server now connects directly to the `requester` service (using `host=requester, port=5678`) while the requester connects directly to the responder (using `tcp://responder:5432`).  The responder uses a wildcard  domain (`tcp://*:5432`).

Note that the version of ZeroMQ installed in the containers is not compatible with the version of ZeroMQ installed on my Mac, so to avoid issues I had to delete my `node_modules` directory before running the services in the containers. This is because of the volume mapping explained above - the containers will install their modules on my local machine in this configuration but if the modules are already installed they will use whatever is there.

Once again, I used NPM scripts to run Gulp tasks to start the containers:
{% highlight bash %}
$ npm run docker:up:requester
$ npm run docker:up:responder
$ npm run docker:up:server
{% endhighlight %}

I can see that the containers are running and the app behaves as expected:
{% highlight bash %}
$ docker ps -a
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                    NAMES
552110a9dfa5        dockerzmq_responder   "sh ./shell/script..."   8 minutes ago       Up 7 minutes                                 dockerzmq_responder_1
fcfc2c0801c3        dockerzmq_server      "sh ./shell/script..."   12 minutes ago      Up 6 minutes        0.0.0.0:3000->3000/tcp   dockerzmq_server_1
f74e9dc68252        dockerzmq_requester   "sh ./shell/script..."   17 minutes ago      Up 7 minutes                                 dockerzmq_requester_1
{% endhighlight %}

## Stage 2: Multiple Containers, Multiple Docker Hosts
The next stage is to create a container for each process and create a separate Docker host for each container on the same machine as shown below.
![multiple host docker][multiple-host-docker-pic]{: .center-image .med-image }

In production mode this is the point where I would create a base image from my `Dockerfile` and add it to [Docker Hub][docker-hub]. I would [configure an automated build][docker-hub-automated-build] triggered by a [Github][github] webhook to re-build the image every time I commit changes to the master branch of the repository.

The only difference between my current `Dockerfile` and that for creating images would be that the new `Dockerfile` would use git to checkout the repository locally to the container, rather than using a mapped volume as in development.  This would have the additional advantage that it would not be necessary to delete the `node_modules` directory as described above.

However, I'm still in development mode so I'm going to map the directory containing my code to a volume on each of the Docker hosts and continue to create containers directly using my local `Dockerfile`.

In order for the containers running in each host to communicate I now need to create a different kind of network.  Whereas containers on the same host can communicate over a *bridge* network, containers in different hosts need an *overlay* network to communicate. Since I am not yet using Docker Swarm mode, I will have to [create an overlay network using a key-value store][docker-overlay-network]. Once I have created an overlay network any containers in any host that are connected to that network will be able to communicate with each other using their service names, as for the bridge network above.

Even though I just said I will not be using Swarm mode, creating this overlay network will involve creating a Swarm. This is because Docker Swarm and Swarm mode are not the same thing, somewhat confusingly! Swarm mode was created in Docker version 1.12 and allows for natively managing a cluster of Docker Engines. Swarm is what was available before Docker version 1.12 and is what I am using in this instance  (one step at a time).

The first thing I need to do is to set up a **key-value store** which will hold information about the network state including discovery, networks, endpoints, IP addresses and more. Docker supports Consul, Etcd, and ZooKeeper key-value stores. This example uses [Consul][consul]. Let's provision a VirtualBox machine called **mh-keystore**:
{% highlight bash %}
$ docker-machine create -d virtualbox mh-keystore
{% endhighlight %}

This will create a new Docker Machine on my Mac:
{% highlight bash %}
$ docker-machine ls
NAME          ACTIVE   DRIVER       STATE     URL                         SWARM               DOCKER    ERRORS
mh-keystore   -        virtualbox   Running   tcp://192.168.99.100:2376                       v1.13.1
{% endhighlight %}

I now need to set the environment to the local **mh-keystore** machine so that I can create a Consul container:
{% highlight bash %}
$  eval "$(docker-machine env mh-keystore)"
{% endhighlight %}

In order to create the Consul container I'll use the official [Consul image from Docker Hub][consul-image]:
{% highlight bash %}
$  docker run -d -p "8500:8500" -h "consul" progrium/consul -server -bootstrap
{% endhighlight %}

I now have a progrium/consul image running in the mh-keystore machine. The server is called consul and is listening on port 8500:
{% highlight bash %}
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                                            NAMES
fda22e403a53        progrium/consul     "/bin/start -serve..."   17 hours ago        Up 17 hours         53/tcp, 53/udp, 8300-8302/tcp, 8400/tcp, 8301-8302/udp, 0.0.0.0:8500->8500/tcp   kickass_bartik
{% endhighlight %}

This image must be running before creating any other Docker hosts otherwise Consul won't be able to register them. The next thing to do is to create a Swarm cluster.  I'll use docker-machine to provision the hosts for my network before creating the network itself. I’ll create several machines in VirtualBox, the first of which will act as the swarm master. As each host is created it will be passed the options that are needed by the overlay network driver. Let's create the Swarm master:
{% highlight bash %}
$ docker-machine create \
 -d virtualbox \
 --swarm --swarm-master \
 --swarm-discovery="consul://$(docker-machine ip mh-keystore):8500" \
 --engine-opt="cluster-store=consul://$(docker-machine ip mh-keystore):8500" \
 --engine-opt="cluster-advertise=eth1:2376" \
 mac-mini
{% endhighlight %}

The `--cluster-store` option tells this Engine the location of the key-value store for the overlay network. The bash expansion `$(docker-machine ip mh-keystore)` resolves to the IP address of the Consul created earlier. The `--cluster-advertise` option advertises the machine on the network. I now need to create a further two Docker hosts and add them to the Swarm cluster:
{% highlight bash %}
$ docker-machine create -d virtualbox \
     --swarm \
     --swarm-discovery="consul://$(docker-machine ip mh-keystore):8500" \
     --engine-opt="cluster-store=consul://$(docker-machine ip mh-keystore):8500" \
     --engine-opt="cluster-advertise=eth1:2376" \
   demo1
$ docker-machine create -d virtualbox \
     --swarm \
     --swarm-discovery="consul://$(docker-machine ip mh-keystore):8500" \
     --engine-opt="cluster-store=consul://$(docker-machine ip mh-keystore):8500" \
     --engine-opt="cluster-advertise=eth1:2376" \
   demo2
{% endhighlight %}

I can now list the machines to see that they're all up and running:
{% highlight bash %}
$ docker-machine ls
NAME          ACTIVE   DRIVER       STATE     URL                         SWARM               DOCKER    ERRORS
demo1         -        virtualbox   Running   tcp://192.168.99.104:2376   mac-mini            v1.13.1
demo2         -        virtualbox   Running   tcp://192.168.99.105:2376   mac-mini            v1.13.1
mac-mini      -        virtualbox   Running   tcp://192.168.99.101:2376   mac-mini (master)   v1.13.1
mh-keystore   *        virtualbox   Running   tcp://192.168.99.100:2376                       v1.13.1
{% endhighlight %}

Now I'm ready to create the overlay network. The usual way to do this is to set the environment to the Swarm master and create the network on the command-line.  However, I'm using docker-compose to create my containers so I'll use that to create the network by expanding the `networks` definition a follows:
{% highlight bash %}
networks:
 zerobridge:
 my-overlay:
  driver: overlay
{% endhighlight %}

Now, when I run docker-compose, the overlay network `my-overlay` will be created. If I also update the definition of the requester service (which is extended by the server and responder services) to use this network then communication between the containers in different Docker hosts should be possible:
{% highlight bash %}
services:
 requester:
  build: .
  volumes:
   - $PWD/..:/home/dogfish/app
  networks:
   - my-overlay
  command: sh ./shell/scripts/startup.sh ${NODE_ENV} ${NODE_PATH} ./services/requesterManager.js target.txt
{% endhighlight %}

If I set my environment to the Swarm master and start a container I shold see that the network is created:
{% highlight bash %}
$ eval "$(docker-machine env mac-mini)"
$ npm run docker:up:server
{% endhighlight %}

This starts the **server** service in the foreground, so I have to open another terminal window and set my environment to the Swarm master to see the created network:
{% highlight bash %}
$ eval "$(docker-machine env mac-mini)"
$ docker network ls
NETWORK ID          NAME                   DRIVER              SCOPE
74a3d55b2f21        bridge                 bridge              local
41703c3c00c1        docker_gwbridge        bridge              local
ba3945515268        dockerzmq_my-overlay   overlay             global
1af37dd52b9c        host                   host                local
bd4d3618c2f0        none                   null                local
{% endhighlight %}

The new network **my-overlay** has been created.  It has been prefixed with the name of the directory containing my docker-compose file. I can now set my environment to each Swarm of the other Docker hosts, **demo1** and **demo2**, and start the services **requester** and **receiver** on them.  I can see that each of them also has access to the overlay network:
{% highlight bash %}
$ eval "$(docker-machine env demo1)"
$ docker network ls
NETWORK ID          NAME                       DRIVER              SCOPE
0b3f0666bf85        bridge                     bridge              local
0fe2171eb4a6        docker_gwbridge            bridge              local
ba3945515268        dockerzmq_my-overlay       overlay             global
6ffe5df43bbc        host                       host                local
59066dba92f3        none                       null                local
$ eval "$(docker-machine env demo2)"
$ docker network ls
NETWORK ID          NAME                   DRIVER              SCOPE
d2886247ddd2        bridge                 bridge              local
0b4bfe957e43        docker_gwbridge        bridge              local
ba3945515268        dockerzmq_my-overlay   overlay             global
4a7aec486c48        host                   host                local
006946905d07        none                   null                local
{% endhighlight %}

### Conclusion
I have successfully run my application 3 different ways on a single machine:
 - As 3 Node.js processes
 - As 3 Docker containers running in a single Docker host with a bridge network
 - As 3 Docker containers each running in a different Docker host with an overlay network

The next step is to run 3 different Docker hosts on 3 different physical machines on my Raspberry Pi cluster, when I can really start playing with Swarm mode.

   [docker-swarm]: <https://docs.docker.com/swarm/>
   [swarm-mode]: <https://docs.docker.com/engine/swarm/>
   [blog-what-is-swarm]: <http://paulnebel.io/api/containers/lean/swarm/2016/08/16/10-stupid-questions-docker-swarm/>
   [blog-rpi-cluster]: <http://paulnebel.io/api/containers/lean/node/raspberry_pi/swarm/2016/08/09/building-a-4-node-raspberry-pi-cluster/>
   [my-app]: <https://github.com/DogFishProductions/swarm-message-queue>
   [node.js]: <http://nodejs.org>
   [api-pic]: <https://paulnebel.github.io/images/cluster-to-swarm/API.png>
   [express]: <http://expressjs.com>
   [node-net-socket]: <https://nodejs.org/api/net.html#net_class_net_socket>
   [request-cluster-pic]: <https://paulnebel.github.io/images/cluster-to-swarm/Request Cluster.png>
   [node-cluster]: <https://nodejs.org/api/cluster.html#cluster_cluster>
   [node-net-socket-server]: <https://nodejs.org/api/net.html#net_net_createserver_options_connectionlistener>
   [zero-mq]: <http://zeromq.org/>
   [response-cluster-pic]: <https://paulnebel.github.io/images/cluster-to-swarm/Response Cluster.png>
   [web-gui-pic]: <http://paulnebel.github.io/images/cluster-to-swarm/web-ui.png>
   [npm-scripts]: <https://docs.npmjs.com/misc/scripts>
   [Gulp]: <http://gulpjs.com>
   [single-host-node-pic]: <https://paulnebel.github.io/images/cluster-to-swarm/Single Host Node.js.png>
   [single-host-docker-pic]: <https://paulnebel.github.io/images/cluster-to-swarm/Single Host Docker.png>
   [dockerfile]: <https://docs.docker.com/engine/reference/builder/>
   [risingstack-docker]: <https://hub.docker.com/r/risingstack/alpine/>
   [ash]: <https://en.wikipedia.org/wiki/Almquist_shell>
   [npm-eaccess]: <https://docs.npmjs.com/getting-started/fixing-npm-permissions>
   [docker-compose]: <https://docs.docker.com/compose/compose-file/>
   [docker-networking]: <https://docs.docker.com/engine/userguide/networking/>
   [config-file]: <https://github.com/DogFishProductions/swarm-message-queue/services/lib/config.js>
   [multiple-host-docker-pic]: <https://paulnebel.github.io/images/cluster-to-swarm/Multiple Host Docker.png>
   [docker-hub]: <https://hub.docker.com/>
   [docker-hub-automated-build]: <https://docs.docker.com/docker-hub/builds/>
   [github]: <https://github.com>
   [docker-overlay-network]: <https://docs.docker.com/engine/userguide/networking/get-started-overlay/>
   [consul]: <https://www.consul.io/>
   [consul-image]: <https://hub.docker.com/r/progrium/consul/>
