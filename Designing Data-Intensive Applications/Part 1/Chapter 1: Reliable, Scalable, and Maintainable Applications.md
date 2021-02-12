# Part 1

## Chapter 1: Reliable, Scalable, and Maintainable Applications

**Reliability:** The System should continue to work correctly (performing the correct function at the desired level of performance) even in the face of adversity (hardware or software faults, and even human error).

**Scalability:** As the system grows (in data volume, traffic volume, or complexity), there should be reasonable ways of dealing with that growth.

**Maintainability:** Over time, many different people will work on the system (engineering and operations, both maintaining current behaviour and adapting the system to new use cases), and they should be able to work on it productively.

### Reliability (Continuing to work correctly even when things go wrong)

* The Application performs the function that the user expected
* It can tolerate the user making mistakes or using the software in unexpected ways
* Its performance is good enough for the required use cases, under the expected ways.
* The system prevents any unauthorized access and abuse.

**Fault vs Failure**:

* Fault: one component of the system deviating from the spec.
* Failure: System as a whole stops providing the required service to the user.

Best to design fault-resistant systems. Ex: Chaos Monkey: It consistently tries randomized use cases to break the code application. Doing this reveals faults quicky and continuously.

**Security**: Focus on failure prevention over fault-resistant (single failure could release critical information)

#### Hardware Faults

Backup generators, redundant machines etc...
If one machine fails there is unlikely cause to expect others to fail.

#### Software Errors

Can cause larger, cascading problems resulting in more machines crashing.
Usually caused by dormant assumptions about the environment it will run in. There is no quick solution, however lots of small things can help.

* Carefully thinking about assumptions and interactions in a system.
* Thorough testing.
* Process isolation.
* Allowing processes to crash and restart.
* Measuring, Monitoring and analyzing system behaviour in production.
* If a system provides some guarantee, have regular automated checks while it is running and be alerted if there is a descrepency.

#### Human Errors

The most common cause of outages. Usually due to human configs.

To mitigate:

* Design systems to minimize oppurtunity for error: ex: APIs, abstraction etc...
* Decouple places where people make the most mistakes and the places likely to cause failure. ex: use sandbox with fully featured non-prod environments with real data that does not affect users.
* Testing... lots of testing
* Allow quick recovery: ex: fast rollback of changes, gradually release new changes so number of people affected are minimal.
* Telemetry: Monitor performance and error rates (Think telemetry when a rocket has lifted off).
* Good management and training (This needs a whole other book).

#### How Important is Reliability

Huge effect on reputation.

### Scalability

We cannot say 'system X is scalable'. Ask questions like 'If the system grows in a particular way, what are options for coping with the growth?' and 'How to add extra compute power to handle the growth'

#### Describing Load

Twitter example: Reading/Writing tweets is not a scaling problem... but 'fan out' where one tweet posted needs to be sent to many different people, and these people have a collection of many different tweets.

Two proposed solutions:

* Insert tweet into global collection of tweets, when a user goes to their home/timeline, make a query looking up all of the people they follow, and their most recent tweets. (NOTE: This is what my original approach would have been before reading this chapter) These are very heavy time consuming queries on scale.
* Maintain a Cache for each users home/timeline (like a mailbox of tweets for each user) when they access their timeline their cache is loaded. This is much less intensive per user, however more intensive when creating a tweet.

Twitter uses both versions now. Caching for almost all users and global collection for users with an exceptional number of followers.

#### Describing Performance

* Response time: What the client sees.
* Latency: how long a request is waiting to be handled (time in line?).
* Service time: How long it takes to process the request. Network delays and queueing delays.

When measuring response times, better to use percentiles instead of mean response time. Use median response time.

Slowest response times are usually from your most valuable clients as they are processing the most data ex: amazon customer with a very large cart.

It only takes a small number of slow requests to slow down everyone elses because of backlog. Only so much parallel processing can be done. This is why it is important to measure response time for clients.

#### Approaches for Coping with Load

* Vertical scaling: Improving the performance of a single machine to handle more load.
* Horizontal Scaling: Distributing the load across multiple machines.

Good architecture usually uses a combination of the two. (NOTE: I have previously had an intuition toward Horizontal Scaling, this is not always the best approach. Consider a lot of very small VM's compared to one or two more powerful machines that could do the same load. A good balance between the two is needed)

There can be manual or elastic scaling.

Possible future trend toward distributed datasystems but for the time being common wisdom is to scale up until you have to scale out. Keep your data in one system until you should split it up due to scale.

There is no *magic scaling sauce* one size fits all scalable architecture. The architecture for scaling is usually specific for the application.

"An achitecture that scales well for a particular application is built around assumptions of which operations will be common and which will be rare".

Early stage start up, it is more important to be able to iterate quickly on product features than it is to scale to some hypothetical future load.
(Note: Point taken, no point scaling until you can reliably have enough growth that you will have to scale. Step 1 get growth, step to scale.)

### Maintainability

* Operabity: Make it easy for operations team to keep the system running smoothly.
* Simplicty: Make it easy for new engineers to understand the system by removing complexity (this does not mean simple interfaces)
* Evolvability/Extensibility/Modifiability/Plasticity: Make it easy for engineers to make changes to the system in the future, adapting it for unanticipated use cases.

#### Operability: Making Life Easier for Operations

An Operations team is vital to keeping a software system running smoothly.
They are typically responsible for:

* Monitoring the health of the system and quickly restoring it if things go wrong
* Tracking down the cause of problems such as system failures or degrading performance
* Keeping software and platforms up to date, including security patches
* Keeping tabs on how different systems affect eachother, so that a problematic change can be avoided before it causes damage.
* Anticipating future problems and solving them before they occur.
* etc... (Okay I get it they do a lot of super important stuff)

Good operations means making routine tasks easy

Data systems can be used to help make routine tasks easy

#### Simplicity: Managing Complexity

One of the best tools we have for removing accidental complexity is *abstraction*. Using Facade design pattern. ex of abstraction: programming language hides complexity of computers

Abstraction is when you still use something but its implementation saves you from having to think about/understand how it is done.

#### Evolvability: Making Change Easy

Agile working patterns and community provide framework for adapting to change. They have also developed technical tools and patterns for frequently changing environments such as 

* Test-driven development (TDD)
* Refactoring

How to apply Agile working patterns to large scale development? Addressed in later chapters.

The ease of modifying a system is closely linked with its simplicity and abstraction.
