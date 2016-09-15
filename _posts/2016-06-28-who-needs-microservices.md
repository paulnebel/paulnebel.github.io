---
layout: post
title: "Microservices: Who needs them?"
categories: Lean Microservices
meta: "Discover the pros and cons of using Microservices in order to make an informed decision about their use. Be able to argue your case and persuade others."
slug: "who-needs-microservices"
---
# Why aren't more companies using Microservices?

Since the publication of the [Agile Manifesto](http://dimsboiv.uqac.ca/8INF851/web/part1/introduction/The_Agile_Manifesto.pdf){:target="_blank"} there has been a growing belief that agile techniques like microservices are the silver bullet for software development.  The ‘traditional’ development methodologies have, at least among the younger development community, come to be seen as old-fashioned and outmoded.  Despite this, the waterfall method is making a strong comeback (Gartner's IT Key Metrics Data show waterfall methods were employed on 56 per cent of development projects in 2015).

There are many possible reasons for this, not least being the difficulty in overcoming [cultural issues](http://www.theregister.co.uk/2016/06/07/problems_for_agile/){:target="_blank"} within the organisation.  This post will help you justify the use of microservices where appropriate and where justification may be necessary to help overcome organisational objections. 

# Context
The architectural style that is now known as microservices is nothing new.  Their current popularity is partly due to trends such as Containerisation, DevOps and Continuous Delivery enabling this kind of approach.  It is also partly as a result of great work at companies such as Netflix who have applied the pattern to great effect and [advocate the approach](https://www.nginx.com/blog/microservices-at-netflix-architectural-best-practices/){:target="_blank"}.

There’s no precise definition of [what a microservice is](http://martinfowler.com/articles/microservices.html){:target="_blank"} but there are certain common characteristics that can help explain what they can be and how they are useful.  Broadly speaking, microservices are a way of coding that: 
 - Creates components (units of software that are independently replaceable and upgradeable)...
 - ...quickly.   If it takes more than a couple of weeks to implement or a team of more than 5 or 6 people it’s probably not micro enough.
 - Organises code around business capability - e.g. where different teams responsible for different microservices.
 - Encourages smart endpoints and dumb pipes. Each microservice is decoupled and owns its own domain logic.
 - Supports automated deployment, organising and enhancing the ability to deploy discrete chunks of functionality.  This potentially increases the ease, speed and frequency of diverse and incremental changes to an application.
 - Decentralises the control of languages and data. Microservices receive a request, apply their own logic and return a response.  They do not have to be written in the same language or have a single database but can utilise the appropriate technology for their particular function.
 - Evolution rather than revolution.  It is possible to replace sections of a monolithic design piecemeal as and when appropriate.  This reduces the complexity and cost of a complete overhaul.

# Drivers
How, then, do you know if microservices are for you?  The following is not, of course, a fully comprehensive list of reasons to adopt microservices.   If some of these conditions apply to you then it’s worth considering a microservices approach: 
 - There is a team of developers working on the application.
 - New team members must quickly become productive.
 - You want to practice continuous deployment of the application.
 - You must run multiple copies of the application on multiple machines in order to satisfy scalability and availability requirements.
 - You want to test the introduction of a new function or service and measure its effectiveness before rolling it out to all customers.
 - You want to take advantage of emerging technologies (frameworks, programming languages, etc).
 - You want to reduce the risk and limitations of committing to a particular technology stack.
 - You want to embrace containerisation of your software stack.

Not surprisingly, the last three points above are anathema to highly risk-averse organisations (e.g. finance). If you work for such a company then microservices aren’t for you.

# Examples 
As Martin Fowler [points out](http://martinfowler.com/articles/microservices.html){:target="_blank"}, Netflix, eBay, Amazon, the UK Government Digital Service, realestate.com.au, Forward, Twitter, PayPal, Gilt, Bluemix, Soundcloud, The Guardian, and many other large-scale websites and applications have all evolved from monolithic to microservices architecture. 

# Issues 
So, just like anything else, it’s not all milk and honey.  For example:  
  - It can be hard to encompass a monolithic application, but it can be done. Replacing a monolith with a complex system of related microservices may result in:
   - Interactions between services that exhibit unexpected behaviours. 
   - Cascading failure (e.g. loss of network - each microservice may appear to be working properly but they can’t speak to each other). However, applications will be more tolerant to individual weakness (e.g. a memory leak only affects that service running in its own process) 
  - Applications must be designed to tolerate the failure of services. 
   - There is a temptation to retain the services you have rather than drop unnecessary services and build new ones. 
   - There is a transfer of complexity from the development team to the DevOps team who must now manage many more processes, load balancers and messaging layers. 

# Summary 
I believe that agile techniques such as microservices are nothing more than a tools.  Like any other tools, you get the best results when you use the right tool for the job.  If you’re a [Lean startup](http://theleanstartup.com/){:target="_blank"} they’re likely to be very useful but if you’re a highly risk averse organisation they’re probably not for you.  Anything in between and you’ll have to decide for yourself.  Hopefully this post will have helped you decide which side of the microservices fence you are on.  

# Further Reading 
 - [https://www.thoughtworks.com/insights/blog/microservices-nutshell](https://www.thoughtworks.com/insights/blog/microservices-nutshell){:target="_blank"}
 - [http://martinfowler.com/articles/microservices.html#CharacteristicsOfAMicroserviceArchitecture](http://martinfowler.com/articles/microservices.html#CharacteristicsOfAMicroserviceArchitecture){:target="_blank"}