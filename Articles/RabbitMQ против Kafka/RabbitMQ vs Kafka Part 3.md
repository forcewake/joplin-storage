RabbitMQ vs Kafka Part 3

In [Part 2](https://jack-vanlightly.com/blog/2017/12/5/rabbitmq-vs-kafka-part-2-rabbitmq-messaging-patterns-and-topologies) we covered the patterns and topologies that RabbitMQ enables. In this part we'll look at Kafka and contrast it against RabbitMQ to get some perspective on their differences. Remember that this comparison is within the context of an event-driven application architecture rather than data processing pipelines, although the line between them can be a bit grey. Perhaps it is more like a continuum and this comparison focuses on the event-driven applications end of that continuum.

The first difference that comes to mind is that the retry and delay patterns do not make sense with Kafka. Messages in RabbitMQ are transitory, they move around and disappear. So re-adding them for a second round of consumption is a real use case. But at the center of Kafka is a log. Adding a duplicate message for a second round of consumption makes no sense at all and would just corrupt the log. One of the strengths of Kafka is its ordering guarantees inside a log partition, adding duplicates messes this up. In RabbitMQ you can re-add a message to a queue that a single consumer consumes, but Kafka is a unified log that all consumers consume from. Delays are not so contrary to the concept of a log but Kafka offers no in-built delay mechanism.

We'll look at how retries might be achieved with Kafka in the patterns section.

A second big difference that affects the patterns made possible by RabbitMQ and Kafka are that messages are ephemeral in RabbitMQ and more persistent in Kafka. In RabbitMQ once a message is consumed it is gone, with no trace of it ever having existed. In Kafka, each message is persisted to the log and remains there until cleaned up. That data cleaning really depends on the amount of data you have, how much space you can give it and the patterns you want to achieve. We can go for a window in time approach where we store the last few days/weeks/months of messages, or we can use Log Compaction to give us a most recent state per message key approach.

Either way we allow consumers to rewind and reconsume older messages. That sounds like retry behaviour though not in the same way as RabbitMQ.

Where RabbitMQ is about moving messages around and giving us powerful primitives to make creative routing patterns, Kafka is about storing the current and historical state of a system. It can be used as a source of truth in way that RabbitMQ cannot. 

## #1 Messaging with Simple Publish Subscribe plus Rewind Option

The simplest use case for both RabbitMQ and Kafka is simply as a publish subscribe infrastructure. A publisher or multiple publishers simply write messages to the partitioned log and a single or multiple consumer groups consume those messages.

Apart from the details of managing how producers send messages to the right partition and how consumer groups coordinate themselves, this is no different than a Fanout exchange topology in RabbitMQ.

If you read Part 2 where we explored all the patterns and topologies of RabbitMQ and found yourself thinking "_I just need basic pub sub_", then the fact that you can rewind and consume older messages will probably swing you towards Kafka.

I got so used to queueing systems that the fact that you can rewind the clock and travel back in time just blew my mind when I first saw Kafka a few years ago. This feature (by using a log instead of a queue) is very useful for recovering from failure. I started working for my current client 4 years ago now, as the tech lead of the backend systems support team. We had over 50 applications that consumed real-time business events over MSMQ and it was a regular thing that an application would get deployed with a bug and process a day's worth of messages before the problem was detected. In many cases the messages were gone and that was that but usually we were able to source the original data from a third party system and republish the messages, only to the single subscriber that had the problem. This required us to build custom republishing infrastructure. If we had had Kafka it would have been as simple as changing the offset for that application.

Now we use RabbitMQ and the support team use the same republishing infrastructure only now targeting the default exchange to send messages to the queue of the application that had the bug.

## #2 Event-Driven Data Integration

This pattern is essentially event sourcing though not at the scope of a single application. There are two types of event sourcing, application level and system level and this pattern relates to the latter.

### Application Level Event Sourcing

The application manages it's own state as an immutable sequence of change events. These events get stored in an event store. To get the current state of an entity means replaying or combining the events of that entity in the correct order. It is naturally combined with CQRS. For some applications this might be the right architecture but it is often misunderstood and misapplied, adding unnecessary complexity when a traditional CRUD app would have been fine.

I am not against this architecture, I see all the great things it enables, but it has its costs too and I have never chosen to use it so I will not go further into it in this post.

### System Level Event Sourcing

Applications can manage their **own** state in any way they want. If they can store their state in a relational database and get immediate transactional consistency that comes with that then great - we should go for that. Not necessarily that it must be relational but immediate consistency with ACID guarantees really is the holy grail for OLTP systems. We only move to eventually consistent architectures when immediate consistency doesn't scale.

But applications often need each other's data and that need can lead to sub-optimal architectures like shared databases, non existent domain boundaries or reliance on awkward REST APIs. 

I listened to a [Software Engineering Daily podcast](https://softwareengineeringdaily.com/2016/10/14/kafka-event-sourcing-with-neha-narkhede/) where they described an event sourcing scenario with a profile service in a social network. There are a bunch of other related services such as search, social graph, recommendation service etc. All need to know about changes to the profile. In my experience as an architect in an airline we have a couple of big software systems with a myriad of smaller services that orbit those larger systems. The auxillary services need bookings data and flight operations data. Each time a booking is made, modified, a flight delayed or cancelled these services need to go into action.

This is where event sourcing comes in. But first let's look at some common problems that arise in large software systems and then look at how event sourcing can solve that.

Large complex enterprise systems usually grow organically; there are migrations to new technologies and new architectures that might not reach 100% of the system. Data is spread across the enterprise in various locations, applications share databases for quick win integrations and no-one is quite sure how it all fits together.

### Messy Data Distribution

Data is distributed and managed in multiple places making it difficult to understand:

*   how data flows through your business
    
*   how changes in one part of the system can affect other parts
    
*   multiple copies of the data exist which slowly diverge over time with conflicts between them
    

Without clear domain boundaries, changes are costly and risky as it means touching multiple systems.

### Centralised Shared Databases

The shared database pattern can cause a few headaches:

*   Not really optimised for any application. It is probably the full data set and fully normalised. Some applications have to write mega queries to get the data they need.
    
*   Applications can affect each other's performance.
    
*   Changes to the schema mean large scale migration projects where development is halted for two months in order to bring all applications up to date, all at the same time.
    
*   No changes to the schema are made. That change that everyone wants never gets done because it is too painful.
    

### Reliance on Awkward REST APIs

Getting data out of other systems via REST APIs is hit and miss. Each REST API might have a different style and different conventions. To get the data you need might involve many HTTP requests and the whole thing can be quite clunky. GraphQL can help with that but it's still no subsitite for having all the data you want locally in a database where you can slice and dice the data with SQL.

We are moving more and more to API centric architectures and there are many benefits to that architecture, especially when these APIs are outside of our control. There are so many APIs out there these days that we don't have to build so much software as we used to. However, it is not the only tool in the toolbox and for an internal backend architecture there are other alternatives.

### Kafka as an Event Store

Let's take a simple example. We have a booking system that manages our bookings in a relational database. It uses all the ACID guarantees offered by the database to manage its state effectively and in general everyone is happy with it. No CQRS, no event sourcing, no microservices, just a reasonably well built traditional application. However there are a myriad of auxillary services (perhaps microservices) related to bookings: push notifications, emails, anti-fraud, integrations with third parties, the loyalty scheme, invoicing, cancellations, flight operations etc. The list can go on and on. They all need booking data and there are upteen different ways they can get it. They also produce their own data which in turn is useful for other applications.

An alternative architecture is put Kafka at the center. On every new booking or booking modification, the booking system publishes the full current state of that booking to Kafka. We use Log Compaction to reduce the messages to only the latest state of the bookings, keeping the size of the log under control.

As far as all other applications are concerned this is the source of truth and they consume that single data source. Suddenly we go from a complex web of dependencies and technologies to producing and consuming to/from Kafka topics.  

Kafka as an event store:

*   If size is not a problem, Kafka can store the entire history of events, which means that a new application can be deployed and bootstrap itself from the Kafka log. Events that are complete representations of the state of the entity can be compacted with Log Compaction making this approach more feasible in many scenarios.
    
*   Events need to be replayed in the correct order. As long as messages are partitioned correctly we can replay the messages in order and apply the filters, transforms etc so that we always end up with the right data at the end. Depending on the "partitionability" of the data we can get highly parallelised processing of the data, in the right order.
    
*   The data model may need to change. May be we develop a new filter/transform function and want that replayed over all the events, or the events from the last week.
    

Kafka can be fed not only by applications in your organisation that publish all their state changes (or the result of state changes) but also from integrations with third party systems:

*   Periodic ETLs extract data from third party systems and dump it in Kafka
    
*   Some third party services offer web hooks and your triggered REST web services can write to Kafka.
    
*   Some third party services expose messaging system interfaces that can be subscribed to and written to Kafka.
    
*   Some third party system deliver data via CSV, which can get written to Kafka.
    

Going back to the issues I explained earlier. With a Kafka centric architecture we simplify data distribution. We know where the source of truth is, we know what the data sources are of that source of truth are and all the downstream applications work with **derived** copies of the data. Data flows from producers to consumers. Master data is owned by the producer alone but others can work on projections of that data freely. They can filter it, transform it, augment it with other data sources and store it in their own databases.

Each application that needs bookings and flight operations data now has it locally because it subscribes to the Kafka topics that contain that data. Applications can use the power of SQL, Cypher, JSON or whatever the query language is. The data can be sliced and diced efficiently because it is stored in a structure and a data store optimised for its needs. We can change the schema without worrying about impacting other applications.

Not all applications need to store that data and can simply react to the data contained in each event. That is fine, event sourcing supports both models.

At this point you might be asking: Why can't I do this with RabbitMQ? The answer is that you could use RabbitMQ for real-time event processing, but not as an event sourcing platform. RabbitMQ is only a complete solution for reacting to events that are happening **now**. When you add a new application that needs its own slice of all the bookings data, in a format optimised for the problem it solves, then RabbitMQ won't help you. With RabbitMQ we're back to either a shared database or using a REST web service provided by the data source.

Secondly, event processing order is important. With RabbitMQ as soon as you add a second consumer to a queue you lose any ordering guarantees. So you could limit RabbitMQ to a single consumer and get correct message order, but this does not scale.

Kafka on the other hand can provide all the data this new application needs to build its own copy of the data and stay up to date, with message processing order guarantees. 

Now think again about API centric architectures. Does APIs for everything still sound like the best idea? I think that when it comes to needing to share read-only data then I prefer the event sourcing architecture. We avoid cascading failures and the lower levels of uptime that comes with larger numbers of dependencies on other services. We increase our ability to slice and dice the data creatively and efficiently. But sometimes you need to change data in another system synchronously and then APIs make sense. Some prefer APIs over asynchronous methods also and I think that comes down to taste. 

This event sourcing approach to data integration fits perfectly with microservices and Self-Contained Systems (SCS). I listened to a [Software Engineering Daily podcast](https://softwareengineeringdaily.com/2017/01/03/self-contained-systems-with-eberhard-wolff/) on SCS earlier this year and the idea is really interesting. I may not go to the lengths to build a fully complient SCS but the idea of _self-containment with asynchronous communication with other systems preferred_ is basically what I have described in this event sourcing pattern.

## #3 - High Traffic, Processing Order Sensitive Applications

A while back I encountered a problem with one of our RabbitMQ consumers. It was consuming a RabbitMQ queue that delivered a feed of files from a third party. The volume of files was high and naturally the application was scaled out to handle the message volume. The problem was that the data in the database was inconsistent and it was causing problems in the business.

It turned out that the problem was that sometimes two files pertained to the same entity and they arrived within a few seconds of each other. They would both get processed and due to load on one server, the second message ended up getting written to the database first and the first message would then overwrite the second message. So we ended up with bad data. RabbitMQ did its job, it delivered the messages in the right order, but we still ended up inserting the data in the wrong order.

We solved it by simply checking the timestamp on the existing record and not inserting if the message was older. We could also have solved this by using the Consistent Hashing exchange and partitioned the queue in the same way Kafka uses partitions.

Kafka stores messages in a partition in the order that they were delivered to it. Message order exists at the level of the partition only. In my example above, with Kafka we would have used the hashing function on the entity id to choose the partition. We would have had a bunch of partitions, more than enough to scale out our consumers. Processing order would have been achieved because each partition is only consumed by one consumer. Simple and effective.

Kafka has some advantages over RabbitMQ regarding partitioning via hashing function. With RabbitMQ there is nothing stopping you from having competing consumers on a single queue that is fed by a Consistent Hashing exchange. RabbitMQ does not help coordinate consumers to ensure that only one consumer consumes from each queue. Kafka does that for you with consumer groups and a coordinator node. So we can rest assured that with Kafka we have a guarantee of one consumer per partition and have the guarantee of processing order. 

## #4 Data Locality

By using the hashing function for routing messages to partitions, Kafka gives us data locality. For example, messages related to user id 1001 always go to consumer 3. Because user 1001's events always go to consumer 3 means that consumer 3 can performantly do some operations that would not be feasible if network round-trips were needed. We can implement counters, real-time aggregations and the like all in memory. This is where the line between event-driven applications and stream processing pipelines start to merge. 

This reminds me of sticky sessions of a bygone era when people used to store web session data in memory and use stateful load balancing to route requests of a specific user to the same server each time. That caused all kinds of problems such as scaling problems and inbalanced loads.

Sticky sessions meant that if our 10 servers were overloaded and we added another 10, then we couldn't move over existing users to other servers because their sessions existed on the original 10 servers. We could only move new users onto the newly deployed instances. This meant that our original 10 servers were still overloaded and the new 10 underutilised. These days we tend to avoid sessions all together or put them in Redis.

So how does Kafka avoid the same pitfalls as sticky sessions? With Kafka we don't elastically scale out and in our partitions. First of all you can't "scale in" the number of partitions; once you have 10 partitions you can't go down to 9. But secondly there's no need to. Each consumer can consume 1 to many partitions so there's little reason to want to reduce the number of partitions. Adding partitions in Kafka introduces latency spikes while rebalancing occurs, so we tend to scale the partitions according to the peak loads and scaling out needs of the consumers.

But if we do need to increase the numbers of partitions and consumers for scaling purposes then we only need pay a momentary latency cost while rebalancing occurs. Note that the data in the partitions stay put if we add new ones. No data is rebalanced, only the consumers. But new messages coming in will now be routed differently and the new partitions will start receiving messages. This also means that the messages of user 1001 might now go to consumer 4 (meaning that user 1001 is now in two partitions).

So we'll need to be able to persist that state somewhere as soon as rebalancing occurs because all bets are off concerning who gets what partitions after the rebalancing. Once rebalancing is finished we can retrieve the state of the new messages coming in and continue our in memory operations.

## #5 Failures, Retries and a Message Lifecycle

In [Part 2](https://jack-vanlightly.com/blog/2017/12/5/rabbitmq-vs-kafka-part-2-rabbitmq-messaging-patterns-and-topologies) I ended with a complete message lifecycle that dealt with transient and non-transient errors through a combination of:

*   immediate retries
    
*   pausing of consumption for a configurable time period
    
*   delayed retries
    
*   message discard
    
*   and the wrapping of messages as Failed Events that get routed to the support engineers for triage and response.
    

With Kafka we can take a similar approach. The difference is that the messages themselves don't move anywhere, they stay put in their log partition. They could get cleaned up though, so we're going to have to take both of those scenarios into account. The delayed retry pattern doesn't make so much sense with Kafka. Instead we can rely on immediate retries, pausing consumption and sending messages to a Failed Event topic.

Immediate retries should resolve problems like very short lived availability issues with other services and things like database deadlocks.

If the issue is longer lived than that then the consumer can pause itself for a minute. It could retry the message every 1 minute, or some kind of backoff could be implemented. If the problem is that an API is unavailable for a few minutes then this strategy should resolve the problem.

If the problem persists past a certain time period then notifications should go out to support engineers so they can start investigating the problem.

Sometimes the issue is not transient and only affects a handful of messages. Perhaps the message has bad data caused by a bug in the producer of the message. Perhaps there is a bug in the consumer that only happens on processing a small subset of messages. In this case we may prefer to move on to the next messages and continue consuming. We can wrap the failed message in a Failed Event message and send it to the Failed Event topic.

These Failed Events contain:

*   The consumer application
    
*   The exception or error message
    
*   The server
    
*   The time
    
*   Who published the message
    
*   How many times has this message been tried
    

This makes diagnosis much much simpler than correlating error logs with the message timestamp. From that topic, support engineers can use a message triage and response application and choose to:

*   Do nothing
    
*   If the problem is resolved, and the consumer application exposes a REST web service, the triage app could call the service passing the offset of the message to be retried.
    
*   If the problem is resolved, and the original message has been cleaned up, the triage app could unpack the original message from the Failed Event and send it back to the original topic.
    

When things go wrong, how we react depends entirely on that specific domain. If it was a notification that your flight is leaving in 30 minutes then does it make sense to retry the message 4 hours later? Or may be it is a type of message that under no circumstance can be lost. Allowing support teams to see what messages cannot be processed and let them make decisions based on their knowledge of the domain is sometimes the only option. Support engineers can automate responses to specific incidents over time which can reduce the burden. But application developers can also automate this decision making process. Using the same "30 minutes till your flight notification" example, the developer can create the logic that makes the consumer discard the message, log an error and move on.

That's it for Kafka messaging patterns. I know there are more patterns but this series is restricted to using Kafka as a general purpose messaging technology for an event-driven application architecture. If you think I missed anything then please let me know.

In the next part we'll compare and contrast the message delivery semantics and guarantees offered by both systems.

*   [Series Introduction](https://jack-vanlightly.com/blog/2017/12/3/rabbitmq-vs-kafka-series-introduction)
    
*   [Part 1 - Two different takes on messaging (high level design comparison)](https://jack-vanlightly.com/blog/2017/12/4/rabbitmq-vs-kafka-part-1-messaging-topologies)
    
*   [Part 2 - Messaging patterns and topologies with RabbitMQ](https://jack-vanlightly.com/blog/2017/12/5/rabbitmq-vs-kafka-part-2-rabbitmq-messaging-patterns-and-topologies)
    
*   [Part 3 - Messaging patterns and topologies with Kafka](https://jack-vanlightly.com/blog/2017/12/8/rabbitmq-vs-kafka-part-3-kafka-messaging-patterns)
    
*   [Part 4 - Message delivery semantics and guarantees](https://jack-vanlightly.com/blog/2017/12/15/rabbitmq-vs-kafka-part-4-message-delivery-semantics-and-guarantees)
    
*   [Part 5 - Fault tolerance and high availability with RabbitMQ](https://jack-vanlightly.com/blog/2018/8/31/rabbitmq-vs-kafka-part-5-fault-tolerance-and-high-availability-with-rabbitmq)
    
*   Part 6 - Fault tolerance and high availability with Kafka