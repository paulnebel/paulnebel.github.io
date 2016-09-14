---
layout: post
title: "Using Neo4j with Seneca: Part II"
categories: Lean Microservices Node Seneca
meta: "How to use the Neo4j Store plugin I built for the Seneca microservices framework.  An example of use with the User plugin."
slug: "using-seneca-neo4j-pt-2"
---
# Description
In a [previous blog post]({% post_url 2016-07-12-using-seneca-neo4j %}){:target="_blank"} I introduced the [Seneca](http://senecajs.org/){:target="_blank"} microservices framework and my [seneca-neo4j-store](https://github.com/DogFishProductions/seneca-neo4j-store){:target="_blank"} plugin for persisting data using a graph database. This post describes [a project](https://github.com/DogFishProductions/seneca-user-service-example){:target="_blank"} I've created to demonstrate how to use this plugin within the Seneca ecosystem.

In this project the standard Seneca [user plugin](https://github.com/senecajs/seneca-user){:target="_blank"} has been extended to accommodate a graph model and is started as a listening microservice. The test client is a separate microservice that exists simply to exercise the extended user plugin. 
# Background
The [seneca-neo4j-store](https://github.com/DogFishProductions/seneca-neo4j-store){:target="_blank"} plugin works like any other Seneca [store plugin](https://github.com/senecajs/seneca/blob/master/doc/data-store.md){:target="_blank"}. It allows us to persist [data entities](http://senecajs.org/tutorials/understanding-data-entities.html){:target="_blank"} without the need to know anything about how or where it is done. This leads to a problem, however - if we use it as a simple replacement for, say, the MySQL plugin then all we've done is to exchange nodes in the graph database for rows in a SQL database. We're not using the power of the graph in any way, so why bother? 
## What are we missing?
Graph databases offer a number of advantages over relational databases due to the nature of the data they describe. Whereas relational databases consist of tables with rows as entities, graph databases represent entities as nodes (sometimes called vertices). In addition to this they define relationships (or edges) between nodes. Both nodes and relationships can have properties (key-value pairs).

Nodes can be labelled with one or more labels. Relationships are named and directed and always have a start and end node. The beauty of the graph is that the relationships between entities are built into the graph itself (i.e. there is no need for expensive join tables). Once you've found the start node you can traverse a small, related sub-set of the overall graph to find the results you need which leads to blisteringly fast performance. There are other advantages, but let's limit ourselves to this for now.

The beauty of Seneca data stores is that they provide the simplest model for data in the form of basic Create-Read-Update-Delete data operations. These map onto a set of conventional message patterns of the form of `role:entity, cmd:save|load|remove|list`. The trade-off to this is that these messages represent the lowest common denominator. It enables what is essentially a key-value store with reasonable, if limited, query capabilities. Data must be normalised because you don't get table joins.

This is clearly an issue with respect to graph databases. Using a graph database without relationships makes no sense. A consequence of this is that the seneca-neo4j-store introduces two new messages, `saveRelationship` and `updateRelationship` which don't conform to the standard Seneca data store metaphor. 
## What's going on here?
In [Part I]({% post_url 2016-07-12-using-seneca-neo4j %}){:target="_blank"} of this blog post I explained why these two new messages have been introduced and how to use them. The purpose of this project is to show some examples of how you might use the Neo4j store plugin in your microservices to engage the power of graphs. 
# Neo4j User Service example
The user plugin provides business logic for user management, including: 
 - registration
 - activation
 - login
 - logout
 - password management, including resets

There are two core concepts: user and login. A user, storing the user account details and encrypted passwords, and a login, representing an instance of a user that has been authenticated. A user can have multiple logins.

If we use this plugin as-is then, as described above, all we will be doing is creating nodes instead of table rows. We will not be making use of the power of graphs. In this example we will create a user microservice that extends the basic user functionality so as to make the most of Neo4j. This is not intended as a recommendation for how to architect your own user service, merely one way of extending seneca-user which provides an example of how to use my seneca-neo4j-store. 

## Key concepts
I will be focussing on user registration and login in this example. It struck me that in a typical application there will be relatively few users and relatively many logins. This means that graphs could be used effectively for, say, audit purposes. Something which kills the performance of graph databases is the existence 'supernodes'. A supernode is a node with a large number of relationships (typically 100k or more) which quickly becomes a bottleneck in graph traversal. It is unlikely that such a node would occur in the context of user management but this is the inspriation for the model contained in this example. 

### Realm
Any company would love to have 100k users registered with one of their applications, so let's start here. When the Neo4j User Service example is started it creates a realm node with a 'scope' property of 'UK' (note also the 'reset', 'login' and 'user' nodes created by the seneca-user plugin - these act as templates for other nodes of the same type based on the respective default values supplied when initialising the plugin):

![realm graph](https://github.com/DogFishProductions/seneca-user-service-example/raw/master/docs/graph1.png){: .center-image }

If your business is as successful as you hope it will be you'll have many hundreds of thousands of users. Using a concept like 'realm' to group them will dramatically improve your ability to retrieve an individual user and traverse her graph. You could, for example, use the top level domain to determine the realm a user belongs to. If you wanted to provide finer-grained control over user activations you could introduce further nodes between the realm and the user, e.g.: 
 - applcation - managed through domain
 - organisation - managed through sub-domain
 - region - managed through top-level path

 These are merely suggestions, but you get the idea. For now we'll limit ourselves to a single realm. Now that we've segmented users into bite-sized chunks, let's deal with user registration. 

### Registration
The essence of this example is logging the time at which specific events occur for audit purposes. With this in mind I decided to use the free  
[neo4j-timetree](https://github.com/graphaware/neo4j-timetree/blob/master/README.md){:target="_blank"} library provided by [GraphAware](http://graphaware.com/) as a mechanism for registering and retrieving event times in a graph-friendly manner. GraphAware TimeTree is a simple library for representing time in Neo4j as a tree of time instants. The tree is built on-demand, supports resolutions of one year down to one millisecond and has time zone support. It also supports attaching event nodes to time instants (created on demand). Since I'm noting the time at which logins occur I might as well extend the `register` action to note the time at which registrations occur, right? Note that I'm using a resolution of `SECOND` in the following examples:

![registration graph](https://github.com/DogFishProductions/seneca-user-service-example/raw/master/docs/graph2.png){: .center-image }

### Active logins
Although any given user may have multiple active logins (e.g. login via desktop, laptop and mobile device at the same time) it is highly unlikely that an individual user will have many (i.e. > 100) concurrent active logins. In this case we can extend the default login behaviour (by calling `prior` on the extended action) to add an `ACTIVE_LOGIN` relationship between a user and any active logins associated with them. In addition, we can use TimeTree to record the time at which login occurred. This is done by adding a `LOGGED_IN_AT` event relationship from the login to the TimeTree when the login is created. Note that, unfortunately, you don't seem to be able to add properties to TimeTree event relationships, so we can't distinguish between active and inactive logins via the `LOGGED_IN_AT` relationship. This isn't a problem as such but it would be nice to have:

![active logins graph](https://github.com/DogFishProductions/seneca-user-service-example/raw/master/docs/graph3.png){: .center-image }

### Inactive logins
Where things start to get interesting is when we consider inactive logins (or 'logouts', if you prefer). While a given user may only have a limited number of concurrent active logins they may well generate a very large number of inactive logins over time. In order to accommodate this I've extended the logout action to add logins to a linked list of 
`INACTIVE_LOGINS` in the order in which they are made inactive:

![inactive logins graph](https://github.com/DogFishProductions/seneca-user-service-example/raw/master/docs/graph4.png){: .center-image }

I don't explicitly create a 'LOGGED_OUT_AT' relationship because I don't expect that I'll need to know that information in detail. Simply maintaining the order of logouts is sufficient for now. One of the beauties of graph databases is that, if I decide that it is important to add a 'LOGGED_OUT_AT' relationship in future I can do so without having to update a schema and potentially break my domain logic. 
### Deactivation
As a final element of this train of thought I've extended the `deactivate` action such that if a user with an active login is deactivated then that login is deactivated and added to the list of `INACTIVE_LOGINS`:

![deactivation graph](https://github.com/DogFishProductions/seneca-user-service-example/raw/master/docs/graph5.png){: .center-image }

# Conclusion
I hope this example gives you ideas for how to use my seneca-neo4j-store plugin in your own projects. It's slightly contrived and there are many other ways of achieving the same thing but it shows how it's now possible to very quickly create and interrogate the audit log using the event retrieval functionality built into TimeTree.