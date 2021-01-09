---
title: "What is ‘Lean’ software development?"
categories: Lean Agile Containers Microservices
meta: "My definition of what 'lean' and 'agile' software development actually is and how it contributes to a lean operations strategy."
slug: "lean-software-development"
excerpt: "There are, effectively, two types of efficiency available to an organisation: resource efficiency and flow efficiency.  Resource efficiency focuses on efficiently using the resources that add value within an organisation.  Flow efficiency focuses on the ‘unit’ that is processed by the organisation (be it a person as in the case of healthcare, or a new application feature as in the case of software development)."
---
## Introduction
It is very easy to use the words ‘lean’ and ‘agile’ with respect to business in general, and software development in particular, without the reader (or, often, the author) fully understanding what these terms mean. I’m going to define what I mean by lean and agile software development and describe how I can help you ‘innovate at pace’.

The concept of ‘lean’ is an attempt to formalise the approach pioneered by the Toyota Motor Corporation.  Mountains of books have been written about lean so what I am describing here is, necessarily, an absurdly abbreviated version derived from the excellent book [‘This is Lean: Resolving the Efficiency Paradox’](https://www.amazon.co.uk/This-Lean-Resolving-Efficiency-Paradox-ebook/dp/919803930X/ref=sr_1_1?s=books&ie=UTF8&qid=1469784866&sr=1-1&keywords=this+is+lean){:target="_blank"} by Niklas Modig and Par Ahlstrom (I highly recommend reading this). 

## What is efficiency?
There are, effectively, two types of efficiency available to an organisation: resource efficiency and flow efficiency.  Resource efficiency focuses on efficiently using the resources that add value within an organisation.  Flow efficiency focuses on the ‘unit’ that is processed by the organisation (be it a person as in the case of healthcare, or a new application feature as in the case of software development).

Most companies concentrate on resource efficiency if only because this is easier to see, understand and control.  Flow efficiency, being a more abstract concept, tends to be overlooked.  Until, Toyota, that is (actually, lean methodology is nothing new; as the authors point out in ‘This is Lean’ the [Venetian Arsenal](https://en.wikipedia.org/wiki/Venetian_Arsenal){:target="_blank"} in Northern Italy was practising what we now call lean in the sixteenth century to mass-produce warships but let’s not get too hung up on this!).

## What is Lean?
At its most abstract, lean is an *operations* *strategy* intended to achieve the twin objectives of: 
 - Prioritising flow efficiency over resource efficiency, as resource efficiency tends to result in superfluous work and waste at the handover between resources.
 - Increasing both flow efficiency and resource efficiency by eliminating, reducing and managing variation.

 For Toyota, the lean principle was derived by thinking about and defining their company in the following terms: 

 - **Values:** Define what it means to you to be a ‘good’ business. They determine how an organisation should behave, regardless of situation or context.
 - **Principles**: Define how an organisation should think, make decisions and prioritise in order to achieve its values. For Toyota, these principles fall into one of two categories: 
   - **Just-in-time:** This is about how to create flow. Flow is the journey from initiation to completion.
   - **Awareness:** This is about creating a visible and clear picture so that anything that happens to, hinders or disturbs the flow can be identified and rectified immediately.
 - **Methods:** Define what an organisation should do to realise its principles in the best way possible.
 - **Tools & activities:** Define what an organisation should have (tools) and do (activities) in order to carry out its methods.

Values, principles, methods, tools & activities are not themselves ‘lean’. They are simply a means of realising a lean operations strategy.

I hope you agree with Modig and Ahlstrom’s definition.  If you don’t then please take it as a nothing more than a frame of reference for what follows.

## How is 'Agile' Lean?
So, how can software development contribute to a lean operations strategy? I’ll deal with ‘agile’ software development first. ‘Agile’ is a shorthand description for a set of tools and activities that are intended to improve collaboration and respond to unpredictability through ‘incremental, iterative work cadences’ known as sprints.  As far as I am concerned ‘agile’ represents a toolbox.  Some of the tools are useful and some aren’t, it just depends on what you like and what you’re doing.

By definition, when done correctly developing in sprints contributes to the ‘just-in-time’ principle by responding to unpredictability (i.e. managing variation). It can eliminate the build-up of ‘stock’ in the form of completed features and bug fixes that can’t be easily deployed under waterfall development.  It can also increase awareness through scrum meetings, sprint planning sessions & review meetings and [Kanban boards](https://en.wikipedia.org/wiki/Kanban_board){:target="_blank"}.

## What are Lean technologies?
Now for some of the so-called ‘lean’ technologies: 

 - **Minimum Viable Products** – The point of an MVP is not to get your product or service up-and-running as soon as possible. That is only a side-effect.  The point of an MVP is to travel round the ‘[build-measure-learn](http://theleanstartup.com/principles){:target="_blank"}’ loop with the minimum of waste.  In this case, the unit of flow is the question that your product/service feature is designed to answer (e.g. it will increase customer loyalty, it will make an activity faster or smoother, it will justify a higher subscription fee).
 - **Microservices** – They’re not a silver bullet (nothing is in software land) but many products can benefit from splitting an otherwise monolithic application into small, specific, independent units of functionality that can be treated as a black box. Being small they can (and should) be developed quickly making them suitable for sprint development.  They provide a means of applying MVP principles to new features, helping identify those that are worth pursuing and developing further.  By facilitating frequent deployment, they contribute to the flow efficiency of both the organisation (just-in-time creation of features) and the customer (just in time receipt of features)
 - **Containerisation** – This goes hand-in-hand with microservices. The whole point of containerisation is to reduce (ideally, to eliminate) variation in environments.  In addition, the auto-scaling and load-balancing capabilities of technologies like [Docker](https://www.docker.com/){:target="_blank"} contribute directly to increasing the efficiency of hosting and administration resources.
 - **Continuous Delivery** – By eliminating the need for handovers between the various stages of the deployment and delivery process this also eliminates the superfluous waste and variation that often results from these handovers. Together with microservices and containerisation this increases flow *and* resource efficiency by increasing the speed, frequency and reliability of code deployment.  It also makes it easier to recover in the case of error or disaster.

 I hope that this lightening-fast journey has helped explain what I mean by ‘lean’ and ‘agile’ software development.  Of course, none of these tools & activities are any good unless they’re done correctly and appropriately. If you are on your journey towards lean I can help – [just get in touch]({{ site.url }}/contact/).