---
title: "How to create an effective developer team on a start up budget"
meta: "As a start up you can't afford experienced engineers so you hire recent graduates. Here are some tips for creating an effective development team from a disparate collection of inexperienced junior developers."
description: "As a startup you can't afford experienced engineers so you hire recent graduates. Here are some tips for creating an effective development team from a disparate collection of inexperienced junior developers."
categories: Insights Startup
slug: "create-an-effective-development-team-on-a-budget"
tags: ["teamwork", "mentoring", "coaching"]
header:
    teaser: "https://pn-blog-images.s3.eu-west-2.amazonaws.com/create-an-effective-development-team-on-a-budget/order-from-chaos.png"
    og_image: "https://pn-blog-images.s3.eu-west-2.amazonaws.com/create-an-effective-development-team-on-a-budget/order-from-chaos.png"
---

{% include image.html url="https://pn-blog-images.s3.eu-west-2.amazonaws.com/create-an-effective-development-team-on-a-budget/order-from-chaos.png" description="A picture epitomising order from chaos" caption='<svg class="icon camera-icon" viewBox="0 0 20 20" version="1.1" aria-labelledby="title"><title>camera</title><path d="M1,5H19V16.91H1ZM12,3h4V5H12ZM10,13.27a2.32,2.32,0,0,1,0-4.64h0a2.32,2.32,0,0,1,0,4.64Z"></path></svg><span>Photo by <a href="https://www.istockphoto.com/portfolio/patpitchaya?mediatype=photography">Patpitchaya</a> on <a href="https://istockphoto.com">iStock Photo</a></span>' %}

## TL,DR;

In the current climate hiring engineers can be an expensive business. There is currently a huge demand for experienced developers and even businesses with deep pockets are finding it challenging to attract and retain talent.

As a startup you don't have the resources to compete with these established businesses when it comes to recruitment so you choose to engage newly qualified graduates.

Chances are that you will find enthusiastic and talented individuals who are keen to develop their careers and are excited to work in a startup environment. Unfortunately, however, they are unlikely to have the experience needed to work together in a way that benefits you.

With some careful preparation and mentoring you can forge an effective team from this raw material. You can protect your start up from the issues associated with staff turnover and enhance the prospects of your engineers in a mutually beneficial way.

The tips in this post show you how to achieve this.

## Understand the problems your developers face

I recently conducted workshops with a couple of technical start ups that shared the same problem. They had hired a small group of graduates straight from university who were skilled at coding but not used to working as a team. As a consequence, problems were starting to occur that were related to how they worked together (or, more to the point, how they were failing to work together).

Did I have any ideas that would help turn them from a collection of inexperienced individuals into a crack team of developers? As it turns out I did have some ideas, and I want to share them with you in this article.

## Recognise that start ups are different from established businesses

Start ups are different in a great many ways from established businesses. One of the primary differences is the amount of money they have available to hire employees.

It is extremely common for start ups to be caught in the bind where they need some kind of prototype to attract investment but without that investment they can't afford to hire experienced engineers to build the prototype.

The average salary of a software developer in the UK varies according to which site you look it up on, but generally ranges from approximately £35k for an entry-level developer to around £60k for a senior-level developer. Rates for London-based developers can be much higher.

In my experience (and humble opinion) £60k is the absolute minimum that a senior software engineer would consider as a salary. For really experienced developers a realistic target is £60-80k (and some way above that for specialists). Obviously the rate depends upon the speciality, but we can take these figures as a ball-park. Contractors are even more expensive, charging anywhere from £400-£650 per day.

Most fledgling start ups can't afford anything like £60-80k per engineer, especially when they will probably need at least 3 of them (one for the back-end, one for the front-end and one for the database). What, then, is a start up founder to do when creating their prototype?

It's tempting for technical founders to consider creating the prototype themselves, but to do so is to create a rod for their own backs. There is more than enough 'business stuff' to be done by a founder of any stripe, let alone being responsible for building their prototype too.

The natural recourse, therefore, is to employ graduates straight from university who are hungry for a job and some experience. This is a great idea, but it comes with it's own unique set of challenges which, if not addressed at the beginning, can result in difficulties later on.

The following are some of the typical issues you may face when hiring a 'team' of graduates who have no industry experience.

### 1. You are the first rung on a very high ladder

More established companies tend to have established teams with established roles, so the scope for doing something outside your defined role is significantly limited.

As a start up your team is small enough that, although you may have some defined roles, the reality is that everyone has to chip in to get stuff done.

This is is wonderful from the developer's point of view because it gives them access to a range of experience they are unlikely to get elsewhere.

It's not so good from your point of view.

The reason is that once they have gained just enough of this almost unlimited experience they will look for a role that utilises that experience, usually with a significantly larger salary than you can offer.

After a year or so with you they are ready to move on, potentially leaving you with a hole in your team that will cost you more time and money than you want to spend to fill it.

### 2. The temptation to use the latest technology is almost impossible to refuse

Being given a 'green field' to start creating your new product or service is intoxicating for a young engineer. Imagine being able to have a say in, or even choose, the shiny new technologies that command big salaries to work with straight out of college.

Once again, this is great for the engineer but potentially really bad news for you.

One of the problems with shiny new technologies is that you're never standing on firm ground for long. Being new, it is likely that you will experience major version updates that introduce breaking changes on a regular basis.

Each time this happens you will have to fix part of the code you have already written for no reason other than that you can't be left with an outdated version for too long. To do so risks a wholesale re-write of your app at a point in the not-too-distant future when your old version is no longer supported in any way (this is referred to as *technical debt*).

Not only that, but virtually no technology gets used on its own. Just as the only certainties in life are death and taxes, so a certainty in a developer's life is that software will be updated. The newer it is, the more often it will be updated (see above).

It is highly unlikely that your shiny new technology will be used on its own. It will typically be integrated into a codebase that uses other third-party modules.

These third-party modules will have their own dependencies and update schedules. If you stay on an outdated version of one technology for too long you'll start to get issues with other necessary modules requiring different, incompatible, versions of common dependencies. This will force you to either update both or not update either. The newer the technology the more likely it is to be updated and the more often you will face this dilemma.

The more established a technology is the less likely it is that major updates will occur and even when they do they are much more likely to be backwards-compatible. Established technology is simply not as exciting to your average engineer as the new stuff, however.

The final significant issue with new tech is that the documentation, if it exists, is likely to be incomplete or out of date or both. There are plenty of existing technologies for which this is also the case, but at least there will be a large number of other developers with experience of that technology to help you out when you get stuck. The newer it is, the less people know how to use it so the likelihood of you not knowing how to do something and getting stuck is much higher.

### 3. The temptation to build everything yourself is almost impossible to refuse

You don't build a drill from scratch every time you want to put a shelf up. Why, then, do so many start ups feel the need to build every element of their product from scratch when so much of the functionality they need can be bought off the shelf?

I've written before about the power of [service design blueprints](https://paulnebel.io/insights/startup/blueprint-for-success/) in finding out what you need to build and, more importantly, what you *do not* need to build.

### 4. When you work with other people you need ground rules

It's a long time since I was at university and it would not surprise me to find that students need to do at least some of their work in groups. However, working as an ad-hoc group at a university is very different from working as a team in a business.

As the founder, you need consistency and continuity. For that you need ground rules, and these rules need to be obeyed if you are not to get into difficulty later.

One of the complaints I've heard from founders is that each of their engineers is doing things in a different way. This is no good for your business continuity.

### 5. When you work with other people you need to communicate

As I mentioned above, it is common to see developer teams comprised of a back-end, a front-end and a database engineer. Each of these roles has its own specific skills and aptitudes so it makes sense to have some kind of differentiation between them.

The problem comes when there is no *communication* between them. Your front-end developer doesn't need to know the minutiae of the database, and your database engineer doesn't need to know exactly how the token authentication system is set up in the back-end.

They do, however, need to know the generalities of these things and the principals that have been applied.

Why?

Because twice in the last month I've been approached by a start up owner whose back-end engineer has accepted a new job after being with them for a year (see point #1 above). No-one else knows anything about the backend and they are panicking about what to do.

This is a problem that could be headed off at the pass by communication.

## Define your expectations

I've already mentioned that start ups are different from established businesses. It is important that the whole team understands what the difference is and the role they need to play to make the business as success.

In a great many ways, a start up development team is better for its members than most other teams they could join. This is because the scope they enjoy for learning new things and taking on responsibilities is generally much greater than being a member of an established team. As discussed above, though, this comes with its own challenges.

As a start up founder you have a chance to break away from the norm and establish something much better. The vast majority of technical businesses fall into the trap of building for the sake of building. This is a great way to spend a lot of money for very little return. You have the opportunity to create a team that builds for the sake of learning, or for the sake of thinking, which is much more powerful. However, it is up to you to make that happen.

You and your team both need to be clear that you are not building the final product. You are, instead, building a working prototype. Your developers need to develop the mindset of *make it work, then make it right* (as espoused by one of the founders of modern coding, [Kent Beck](https://en.wikipedia.org/wiki/Kent_Beck)).

Remember that *perfection is the enemy of execution*. Your team should aim to create code that is not sloppy but is also not perfect. You are at the stage where you are still establishing problem/solution fit. You need to *love the problem more than the solution* which may mean throwing things away and moving in a different direction that the evidence leads you down.

Your team needs to understand that the code they are working on now may be deleted in a month, or may end up in the final version of the product.

## Create your infrastructure

I've [written previously](https://paulnebel.io/insights/startup/be-prepared/) about the basic things you need to do as a founder to create a solid environment for your technical development.

Even if you are not yourself technical you need to understand at least enough to decide for yourself on the technologies to be used for the front- and back-end of your product. This doesn't mean that you need to be able to write a line of code, but you do need to understand the trade-offs you are making when choosing how to build your prototype.

As mentioned above, you should steer well clear of shiny new technologies unless you have absolutely no choice in the matter (i.e. you cannot achieve your goal any other way). For this first version of your product it is far better to use a well established technology like Javascript of Python in the first instance rather than [Haxe](https://haxe.org/), [ClojureScript](https://clojurescript.org/), [Kotlin](https://kotlinlang.org/docs/tutorials/javascript/kotlin-to-javascript/kotlin-to-javascript.html) or any other of the new players on the block.

Don't get me wrong - I'm not saying that these technologies are not superb in their own right, just that a junior development team is likely to get its knickers in a twist quite quickly with one of these new languages with no-one to help them out of it unlike Javascript of Python which have millions of helpful users, tutorials and code examples.

Both Javascript and Python have mature, stable frameworks that support both front- and back-end development ([Node.js](https://nodejs.org/) in the case of Javascript, [Django](https://www.djangoproject.com/) in the case of Python). This means that you can choose engineers with the same skills for both front- and back-end development who can support each other.

It is highly unlikely that you will need lightning fast performance at this stage (or, indeed, at any stage - this is usually the exclusive domain of fintech where nanoseconds can equate to profits). It is much better to go for something simple, with a view to building it better/faster once you have established both what it should do and how it should do it.

For example, [React](https://reactjs.org/docs/getting-started.html) is a great framework but it can become very complex very quickly (and I can pretty much guarantee that your developers will be using their experience building your product to get themselves a better React job as quickly as possible). Think about plain javascript with as few libraries as possible to start. Choose technology with plenty of support and at least half-decent documentation.

This may be a harder sell to your new team members but it will be much easier to replace those that leave if you take this approach.

## Set your baseline standards

Choose whichever [coding standards](https://www.geeksforgeeks.org/coding-standards-and-guidelines/) you prefer. There are plenty of them for any given language produced by the likes of Google and Facebook who know what they're talking about. Just make sure to choose a standard and stick with it.

Most modern development environments like [VisualStudio Code](https://code.visualstudio.com/), [Sublime Text Editor](https://www.sublimetext.com/) or [Atom](https://atom.io/) will have modules developed for them that highlight exceptions to pre-defined code standards, making it easier for your team to stick to the rules.

For the front-end use a framework like [Bulma](https://bulma.io/) together with [Material Design](https://material.io/design) to create the interface without wasting time worrying whether a button should be 2 pixels to the left or not. Use tools that allow you to concentrate on what you are building, and building quickly, not on how to build it.

## Choose the processes that work best for you

One of the reasons you chose to start your own business was to be in charge. So be in charge.

Choose processes because they *work for you*, not because everyone else is using them. Don't be afraid to stop doing things that don't work and try something else.

It is easy to forget that [Agile Software Development](https://www.agilealliance.org/agile101/) is a toolkit, not a shopping list. Start ups are not like established businesses. Don't use Scrum because everyone else is using it, use it because it works for you (or don't if it doesn't).

### Plan to achieve business objectives, not product functionality

Your development team is likely to be much more effective if they understand that they, like you, are trying to achieve a business outcome. The only reason they are building anything at all is so that you can get your business to the place that you want it to be in.

Make sure that you explain to your team what your business objectives are so that they know why they are doing what they are doing. The aim isn't to build a piece of software, the aim is to have a revenue of eleventy squillion pounds in 3 years time. The software they are developing is hopefully the best way of achieving this goal, but if they or you can think of a better way of achieving that goal then do it.

### Think in terms of achievements, not development tasks

In my experience your horizon should extend no further than the current week. At the very least you should have a team meeting on Monday to discuss what you want to achieve in that week. You can record this any way you like (e.g. [Trello](https://trello.com/), [Jira](https://www.atlassian.com/software/jira), [Notion](https://clickup.com/blog/notion-project-management/)). Remember that it is your responsibility to set the goals, so be *clear* about what you want your development team to achieve.

### Organise your time and make your tasks visible

You need to be *realistic*. Developers (myself included) are *terrible* at estimating how long it takes to do anything because you can never be sure that you won't hit an unexpected problem. You can minimise this by putting tasks into a calendar, not just allocating them 'points' in the planning meeting.

This makes it much easier to see what you can get done and helps account for all those other tasks that have been booked in to your week.

Share your calendars so that everyone in the team can see what everyone else is doing. This is especially important in this time of remote working. It is far easier and more effective to operate as a team if you know who is doing what at any given moment. This way, you can see how what you are doing affects, and is affected by, what others are doing.

### Plan ahead, but not too far ahead

Make sure your developers are delivering every week - no month-long sprints. You need to show progress, to test changes and to raise revenue. You need at least one other meeting per week in addition to your Monday planning meeting. Friday is a good day for this extra meeting, and the point of it is to assess what got done, what went well (to be repeated) and what did not go well (to be fixed for the future). We normally refer to this as a *retrospective*.

### Share knowledge at source, make the best use of your teams talents

You are a small team so try to be *one* team until you get too big for that to work. Your engineers and designers are smart people, let them join in customer conversations (even if only as silent listeners) so that they understand the problem and can come up with the best solution.

### Measure achievements, not nonsense

Avoid at all costs judging team performance by vanity metrics like number of lines of code written. The only thing that matters is how many new customers you have acquired and the number of existing customers you have lost. Everything else is just noise.

## Trust your team

Set them up and watch them go. Even if you are technical, as the founder you will have much more important things to do than to monitor and manage the technical team. This means you need to put them in a position where they can manage themselves as far as possible.

### Share responsibility, share knowledge

Despite what I said about Agile development being a toolkit, it is vital that your team use a methodology like [gitflow](https://nvie.com/posts/a-successful-git-branching-model/) for managing the codebase. Whenever a developer wants to add changes they have made to the codebase they need to issue a [pull request](http://oss-watch.ac.uk/resources/pullrequest), which should be reviewed before being committed.

Let your team members take turns in being responsible for code reviews and committing pull requests. The person who wrote the code explains what it is doing to the person who will commit the code *before* it is committed to the main or master branch of the repository. This way you have a chance of spreading knowledge across the whole team and not being hamstrung if your back-end developer decides to leave.

## Coach your team

If you want to turn a group of inexperienced developers into an effective team you need to show them how to work. If you can't, or don't want, to do that you need to bring someone into the team to do it for you.  You can either hire another team member to do this or you can consider engaging with a Fractional CTO for the time it takes to establish the rigor and discipline your putative team needs.

## The take-away

You've got to sell your start up to employees just like you sell it to customers and investors. This means that they have to believe in your idea, the path to get your goals, and in you.

To do this, you need to make them understand what their contributions will be and how their talents will make a positive impact on reaching those goals.

They need to believe that the ride is going to be rewarding and fun.

You need to give them the right environment for this to happen which suits both their objectives and yours.

**Make sure everyone wins.**
