RabbitMQ vs Kafka Part 2

In this part we're going to forget about the low level details in the protocols and concentrate on the higher level patterns and message topologies that can be achieved in RabbitMQ. In [Part 3](https://jack-vanlightly.com/blog/2017/12/8/rabbitmq-vs-kafka-part-3-kafka-messaging-patterns) of the series we'll do the same for Apache Kafka.

First we'll cover the building blocks, or routing primitives, of RabbitMQ:

*   Exchange types and bindings
    
*   Queues
    
*   Dead letter exchanges
    
*   Ephemeral exchanges and queues
    
*   Alternate Exchanges
    
*   Priortity Queues
    

Then we'll combine them all into a set of example patterns.

## Exchanges Types

### Fanout Exchanges

These exchanges provide the typical publish subscribe topology. A message sent to a Fanout exchange will be broadcast to all queues and exchanges that have a binding to the exchange.

![Fig1. Fanout exchange broadcasts to three queues (independent consumers)](../../_resources/74931c88d1064925a1d5d7e4db24a0fb.pngformat750w)

Fig1. Fanout exchange broadcasts to three queues (independent consumers)

In the above diagram each consumer is independent of the others and receives its own copies of all the messages. To scale out the Consumer App 1, more instances of that application would need to be deployed, consuming from the same Queue 1.

Fanout exchanges are the fastest of the exchanges as they do not need to inspect any routing key or message header. Although an exchange may send a single message to multiple queues, in reality it doesn't necesarily duplicate the bits. It can persist the message to the Mnesia database and simply register a pointer to the message in each queue.

### Direct Exchanges and the Default Exchange

Direct exchanges route messages using the Routing Key of the message. Routing keys are set by the publisher of the message. They are multi-word strings separated by dots. Some examples might be "booking.new", "booking.modified" and "booking.cancelled".

A binding between a queue or exchange and a Direct exchange contains a Binding Key. This is an exact match value.

![Fig 2. Direct exchanges routes by exact match Routing Key to Binding Key](../../_resources/22b03289a5574bbdb21f2b6c4fd1ca9f.pngformat750w)

Fig 2. Direct exchanges routes by exact match Routing Key to Binding Key

Direct exchanges are the second fastest exchange as they only perform exact string match operations.

There is a special exchange called the Default exchange which is a Direct exchange. The Default exchange has an implicit binding to all queues in its virtual host. This implicit binding to each queue has as its binding key the name of the queue. This means that you can send a message directly to a specific queue by its name.

![Fig 3. Default exchange has an implicit binding to each queue](../../_resources/973c15df2c1142bb9eb1d9ab50cbb75f.pngformat750w)

Fig 3. Default exchange has an implicit binding to each queue

This can be useful if the publisher wants to choose exactly which consumer it wants to process its message, rather than relying on bindings configured by consumers. Normally we want complete decoupling between the publisher and subscriber, and that is what the other exchanges provide. But in the case when you want point-to-point messaging, the Default exchange provides that capability.

### Topic Exchanges

These exchanges also route using the Routing Key, but Topic exchanges offer the use of two types of wildcard in the Binding Key.

The * wildcard matches a single word in a routing key. For example the routing key "booking.new" has two words. The # wildcard matches any number of words.

For example, let's say we have the following routing keys:

*   booking.new
    
*   booking.modified
    
*   booking.cancelled
    
*   extras.car.new
    
*   extras.car.modified
    
*   extras.car.removed
    
*   extras.hotel.new
    
*   extras.hotel.modified
    
*   extras.hotel.removed
    

We can create bindings with the following binding keys:

*   _booking.new_ \- exact match
    
*   _extras.*.modified_ \- all modifications to extras on the booking (cars or hotels)
    
*   _extras.#_ \- all extras
    
*   _#.new_ \- all new bookings and extras
    

With careful design of routing keys and binding keys, we can add new routing keys without needing to update the existing bindings, making the system robust in the face of change.

Topic exchanges allow you to configure a single exchange for an application to send its messages to, using the routing to ensure that the messages get to the right consumers. This simplifies the configuration and deployment of the publishing and consuming applications.

Note that topic exchanges slow down as the number of bindings increase.

### Header Exchanges

These are the most powerful, but also the slowest of the exchanges. This fact needs to be taken into account as you may have scaling issues with this exchange type. Header exchanges ignore the routing key and instead parse the message headers. Each binding to a Header exchange can include multiple header matchings of which ANY or ALL must match. 

Let's say your application publishes to a single exchange a set of different messages that form a real-time change log that allows for integration with other systems. Each message has the following message headers:

*   entity.type (booking, passenger, baggage, pet)
    
*   change.type (new, modified, cancelled, removed, moved)
    
*   agent.id
    
*   client.id
    

We could create the following bindings:

*   _entity.type=booking, change.type=cancelled, x-match=all._ I want all cancelled booking messages.
    
*   _entity.type=passenger, x-match=all._ I want all passenger messages.
    
*   _entity.type=pet, change.type=new,x-match=all._ I want all newly added pet messages.
    
*   _agent.id=2, client.id=1001, x-match=any._ I want all messages related the specific travel agent or end client.
    

As you can see, Header exchanges are quite powerful.

### Consistent Hashing Exchanges

A Consistent Hashing exchange allows us to partition a single queue into multiple queues and distribute messages between them via a hashing of the routing key, message header or message property.

![Fig 4. Messages are distributed by partitioned hash space](../../_resources/0b6c69eafec84904abdceff05f64788a.pngformat750w)

Fig 4. Messages are distributed by partitioned hash space

This gives us patterns such as ordered processing guarantees and data locality, two pattterns you will see below in the patterns section.

There are some issues with Consistent Hashing exchanges though. Firstly that RabbitMQ doesn't help you to coordinate your consumers across the partitioned queues like Kafka does. So that is down to you to manage somehow. Kafka gives you this out of the box.

Other potential problems are:

*   the thing you hash (routing key, message header or property) doesn't have enough variance to create an even distribution. If you only have four different values then you might get unlucky and all go to a single queue.
    
*   if you have relatively few queues then distribution may be uneven again.
    

Other distributed systems solve this by using the concept of virtual nodes. For example, to prevent imbalanced distribution Riak and Cassandra have virtual nodes which number much greater than the physical nodes and they distribute these virtual nodes across the physical nodes. That way they get better distribution when a cluster has relatively few physical nodes. RabbitMQ doesn't have this concept so pay attention to distribution of messages.

## Dead Letter Exchanges

We can configure a queue to eject a message and send it to a configured exchange upon one of three conditions:

*   The queue has reached the message count limit. The message at the head of the queue (oldest message) is ejected and sent to the configured dead letter exchange (DLX). So when a new message arrives to a full queue it basically kicks out the oldest message and is safely added to the queue.
    
*   The queue has reached its size (bytes) limit. Again, the oldest message is ejected.
    
*   The queue has been configured with a Message Time To Live (TTL) limit and a message has reached that limit.
    
*   A message has been configured with its own TTL, and it has reached that time period in the queue.
    

Messages are only dead lettered from the head of the queue. So messages that have passed their TTL only get forwarded to the DLX when they reach the head of the queue. _This is very important to remember!_

Dead letter exchanges are just regular exchanges, you can create one as any of the four types and bind any queues or other exchanges to it. I previously [documented](https://jack-vanlightly.com/blog/2017/6/11/improving-reliability-and-incident-response-via-a-message-lifecycle) a topology that uses a centralised dead letter exchange. 

RabbitMQ's dead letter functionality provides for more than just an escape route for messages in peril of being lost. It can be used for retry and delay queues as we'll see in the patterns section.

## Ephemeral Exchanges and Queues

Exchanges can be configured to **auto-delete** themselves onces all queue bindings have been removed. Queue bindings can be removed by either just removing the binding itself or removing the queue.

Queues can be configured to **auto-delete** once all consumers have stopped using the queue. This can be because a consumer cancelled its subscription or that the channel closed.

Queues can be configured to be **Exclusive Queues**. This means that only the consumer that declared the queue is able to consume it and once the consumer cancels or closes the channel the queue auto-deletes.

Queues can be configured with a **Queue TTL.** Once the queue has been unused for the TTL period, it will be deleted. Unused means no active consumers subscribed.

Ephemeral exchanges and queues can be used for patterns such as delay queues, retry queues and reply-to queues, as we'll see in the patterns section.

## Alternate Exchanges

Each exchange can be configured with an alternate exchange. When the exchange cannot route a message because either there are no bindings or no bindings match the message, then the exchange will route the message to its alternate exchange.

This gives us a way of not losing messages that might be lost because of a bad routing key or bad routing topology. But it also enables new routing patterns that the four exchange types do not provide for. We'll see in the patterns section how Alternate Exchanges can be used for different routing scenarios. 

## Priority Queues

Messages can be configured with different priority levels, up to 10 levels is recommended. When a queue is declared, it can be declared as a priority queue. If the publisher sets a priority on a message then its position in a priority queue will be determined by that priority. Higher priority messages get shunted further forward than lower priority ones. So if a queue has 1000 low priority messages and a high priority message arrives it will be placed at the head of the queue immediately.

There are two important considerations to take into account with priority queues. If a priority queue becomes full and has a high priority message at the head of the queue, when a low priority message arrives it will kick out the high priority message. The low priority message will be safely persisted to the queue and the high priority message will be sent to the DLX.

Likewise, lower priority messages can get stuck because they may be perpetually behind higher priority messages. Even setting a message TTL would not help in this case as dead lettering always occurs at the head of the queue.

So use priority queues with care.

## CC and BCC

Publishers of messages can add additional routing keys in two string array message headers (CC and BCC). They behave just like email. Routing keys in the CC and BCC headers route the message just like the standard message routing key. The BCC header will be stripped out of the message before delivery.

## Declaring Exchanges, Queues and Bindings

Applications themselves can declare the exchanges, queues and bindings they want. Kafka requires some centralised management due to decisions about partitions that affect all consumers. RabbitMQ is more flexible in this regard and applications can manage their own RabbitMQ artefacts without worrying about affecting other applications. Convention based topolgies are great as we can use simple conventions within applications and build sophisticated routing topologies that self-manage themselves. However, if scaling and performance become a concern you may need to carefully architect your routing topology into a more specialised snowflake. 

## Virtual Hosts

A virtual host is a logical container of exchanges and queues. They can be used to control access to exchanges and queues. Cross virtual host routing is not possible so all examples in this section assume an encompassing virtual host.

## #1 - Simple Broadcast with Fanout Exchange

Use a Fanout exchange to broadcast all messages to all consumers.

![Fig 5 - Simple pub sub pattern with Fanout exchange](../../_resources/108f5648be5a417484e9c082724318c6.pngformat750w)

Fig 5 - Simple pub sub pattern with Fanout exchange

## \# 2 - Multi Layered Exchanges

In order to reduce the cost of routing, a layered approach can be applied. In the example below we initially route based on a small finite number of routing keys to other exchanges. Each binding to a topic increases the overhead of that exchange. In this case we can route all the booking messages to a fanout exchange where it can be efficiently broadcast to all interested consumers.

Likewise, those consumers that want to filter based on message headers of a given type of message can route messages efficiently to the more costly Header exchange where message header based routing is performed over a subset of the messages that come in through the entry point Topic exchange.

![Fig 6. Optimise routing overhead through layered exchanges](../../_resources/65aacbdb66a7436287e00630a7e08ffe.pngformat750w)

Fig 6. Optimise routing overhead through layered exchanges

If all downstream exchanges want to be able to route all messages the entry point can be a Fanout instead.

![Fig 7. Fanout broadcasts to more specialised exchanges](../../_resources/e5089f94f3e8437eb0b1a5dd4bfaf1ab.pngformat750w)

Fig 7. Fanout broadcasts to more specialised exchanges

## #3 - Email Routing System with Header Exchanges

Emailing routing is not a general pattern but it does demonstrate the power of Header exchanges. This example also demonstrates how you can reduce your dependency on schedulers.

Let's say we are an airline and we work with a partner called ABC that performs aircraft maintenance on our aircraft. Every day their systems send us emails which contain information inside the email or within attachments. Yes welcome to the world of integration via SMTP, it's real.

You have five applications that all need to update internal systems with the various data files that ABC sends us daily. For example the finance department needs to know aircraft component status in order to create predictive models on future costs so the required budget is provisioned and accounted for.

When all five applications read from the mailbox directly, we can no longer rely on read status. Each application needs to track what they have read before, skip the emails they are not interested in and be scheduled to run every X minutes or hours. We have to implement mailbox read logic over and over.

![Fig 8. Applications reading directly from a mailbox](../../_resources/2bd6c0e8cdae4d5facbef2da00d677f2.pngformat500w)

Fig 8. Applications reading directly from a mailbox

Instead we could have a single application be responsible for reading the mailbox that writes all emails and their attachments to a database. Then each application needs to read from that database. Again each application must keep track of what emails it has read, which is repetitive code (see my [Taskling](https://github.com/Vanlightly/Taskling.NET) project on Github for a way of tracking data processed by batch jobs). These applications also need to be scheduled by a scheduler.

![Fig 9. Applications reading from a database](../../_resources/bf6db210b4844cfca6d1641b15250f7e.pngformat750w)

Fig 9. Applications reading from a database

A better option is to have a single scheduled application that reads from the mailbox and sends the emails as messages on RabbitMQ, to a Header exchange. Attachments can be persisted to a database or a cloud service like S3, with just the attachment key in the message. The email properties such as sender address, recipient address, cc, subject are all added as message headers. Then each application need only create bindings which match the emails they want to consume. The applications need no scheduler as they get pushed the email messages from RabbitMQ.

![Fig 10. Applications receive emails they want over RabbitMQ](../../_resources/851d9a8ae13341e38055ee05ec21e386.pngformat750w)

Fig 10. Applications receive emails they want over RabbitMQ

The limitation of Header exchanges is that you can only do exact matches. This does rule out Header exchanges quite often unfortunately. I would love for Pivotal to invest more into Header exchanges but for now that is how it is.

## #4 - Public Message Exchange, Private Consumer Exchange

This is a flexible, convention based routing pattern. Unique snow flake topologies can be difficult to manage as they grow larger. I tend to prefer convention based topologies as they tend to manage themselves. In this pattern, publishers of messages declare a Fanout exchange based on the message type name. Consumers on start-up declare their own queue and for each message they consume they declare their own private exchange and bind it to the message exchange they want to subscribe to. Using this simple logic, publishers and consumers create all the queuing infrastructure automatically without knowing about each other, or impacting each other in any way.

![Fig 11. Public message exchanges, private consumer exchanges](../../_resources/b7f41d05b800489cb18448723b4faf9b.pngformat750w)

Fig 11. Public message exchanges, private consumer exchanges

In the above diagram the publisher publishes two message types: new bookings and modified bookings. For flexible routing, it sets the sales channel as the Routing Key (the sales channel could be the main website, comparison sites, travel agencies etc) and adds some other interesting data in the message headers.

The publisher simply publishes each message to its corresponding message exchange. Each consumer has its own queue and private exchange. It binds its private exchange to the message exchange. In our example Consumer App 1 wants all new bookings. Consumer App 2 wants alll new bookings of a specific very important client. Consumer App 3 wants all new and modified bookings related to MyTravel.com which sells bookings as a 3rd party seller.

This pattern makes for a self-managing topology where the only clean-up required is when consumers are removed permanently from the system. Deployments and development is simplified as all applications create the necessary RabbitMQ exchanges, queues and bindings they need reducing the burden on the operations team and deployment pipelines.

Another benefit of giving each consumer its own private exchange is that support teams can put in "wire taps" to view all messages consumed by an app. You can create a queue and bind it to the private exchange of a consumer application and get copies of all the messages it receives. This can also be used for adding on-demand audit logging of messages of a given consumer.

## #5 - Point-to-Point Messaging

We can bypass the various routing options and send messages directly to a queue by name. Send a message to the default exchange, with the name of the queue as the Routing Key and it will be directed straight to the queue.

![Fig 12. Point-to-point message via the Default exchange](../../_resources/234cde03a7174a1dacb41b3f0bbeea23.comstatic56894e581c1210fead06f878t5a29885153450a6ffb92ebca1512671320691format750w)

Fig 12. Point-to-point message via the Default exchange

This is useful when the publisher wants control over which consumer processes a message, rather than relying on routing where 0 to many queues might receive the message. NServiceBus uses the default exchange for the sending of commands. NServiceBus splits messages into two categories: events and commands. Events get published to exchanges where any consumer can subscribe to the event. Applications send commands directly to specific consumers by using their queue name.

## #6 - Processing Order Sensitive Applications

Sometimes you need to be scale out your consumers and maintain ordered processing of messages. While RabbitMQ guarantees the FIFO ordering of a queue, if there are multiple competing consumers each consuming multiple messages in parallel, the processing order is lost.

![Fig 13. Multiple competing consumers consume a single queue and lose processing order guarantees](../../_resources/c2a19bf7f7644874954db33dbdccf5a0.pngformat750w)

Fig 13. Multiple competing consumers consume a single queue and lose processing order guarantees

One way of getting around this problem is to use a Consistent Hashing exchange and partition the queue into multiple queues and route messages to these queues based on hashing the routing key or a message header. Normally the global order of all messages is not necessary, just the order of related messages. For example, all messages related to a given booking must be processed in the correct order. So if we set the booking id as the routing key or as a message header we can guarantee that all the messages of a given booking id always goes to the same queue. Then if we only have a single consumer consuming from that queue we get the processing order guarantees we need.

![Fig 14. Messages distributed by hashing function, each queue consumed by one consumer](../../_resources/ffa87ed53ab740a8b4cedc280fc786b4.pngformat750w)

Fig 14. Messages distributed by hashing function, each queue consumed by one consumer

But, RabbitMQ doesn't help you coordinate your consumers to match one consumer to one queue. That is down to you to do somehow.

## #7 - Data Locality

By using the Consistent Hashing exchange like in the previous pattern we also get data locality. For example all of user 1001's events always go to consumer 2 because we hash a message header that contains the user id.

This means that consumer 2 can do some operations that would not be feasible if network round-trips were needed. We can implement counters, real-time aggregations and the such in **memory**. 

However, while this sounds great there are dangers involved. If you increase the number of queues then the distribution of messages changes. So now user 1001's messages go to consumer 4. But consumer 2 doesn't know that, it just stops seeing user 1001 messages. So now you have two consumers with in memory counters and aggregations. You can avoid this by hand-rolling some way of notifying consumers of a change in message distribution and get them to write out their in memory values to a data store and expect a new slice of user messages.

## #8 - Hierarchical Routing

This is an extension of the Public Message Exchange, Private Consumer Exchange pattern and allows for Topic exchange like routing using Fanout exchanges.

Imagine we have split our business into domains, sub domains and actions. We could construct a message namespace in the format _domain.sub-domain.action_:

*   finance.invoicing.invoice-requested
    
*   finance.invoicing.invoice-generated
    
*   finance.fraud.alert
    
*   finance.fraud.check
    

We create three extra message exchanges:

*   finance.invoicing
    
*   finance.fraud
    
*   finance
    

We create bindings to these exchanges according to the namespace as below:

![Fig 15. Hierarchical routing](../../_resources/b0f70fd1bf534217a53a63d85a09321a.pngformat750w)

Fig 15. Hierarchical routing

This pattern can be useful when you want to capture the messages of large groups of related exchanges without having to create large numbers of bindings. When publishers declare the message exchange of the message they publish, they also declare the exchanges in the hierarchy and the necessary bindings. This means that once you subscribe to the parent exchange, when new child exchanges are added their messages automatically get routed to you.

## #9 - Class of Service with Priority Queues

Some instances of a given message type may carry greater priority than others. Perhaps some clients are more important than others, or messages can be flagged as higher priority. One way of processing higher priority messages before lower priority ones is to configure a queue as a priority queue, and send all messages with a priority level.

![Fig 16. Priority Queue](../../_resources/6ac940aa3f2f48eaa16685f9240a99b2.pngformat750w)

In general though, priority queues are little too simple. Lower priority messages can get stuck behind higher priority messages and dead lettering can eject high priority messages in favour of low priority messages.

A better approach to class of service is using a topic exchange, see the next pattern.

## #10 - Class of Service with Topic Exchanges

Using a Topic with the priority set in the Routing Key has neither of the gotchas of priority queues. Messages are routed to physcially different queues and processed by different application instances. The high priority queue may offer lower latency solely by the fact that messages don't have to wait behind lower priority messages, but also higher priority messages may be consumed by a more scaled out set of consumers on bigger VMs. 

![Fig 17. Routing by priority](../../_resources/7f81c991252b4ce3914bf4b23c3b2eff.pngformat750w)

Fig 17. Routing by priority

## #11 - If/Else Routing with Alternate Exchanges

Imagine we have a handful of super important clients that require custom behaviour for each message and each of these messages need go to a dedicated consumer for that client. Messages related to the hundreds of lesser important clients should get handled by a generic consumer.

We can achieve this by putting a client identifier as the routing key and sending the messages to a Direct exchange configured with an Alternate exchange.

![Fig 18. If/Else routing with alternate exchange](../../_resources/d42f564c4973451bb177fc64a29476d0.pngformat750w)

Fig 18. If/Else routing with alternate exchange

The alternate exchange is just a regular exchange, of any of the four exchange types. You can even chain alternate exchanges together and make if, else if, else if, else logic structures.

A Topic exchange could not deliver this routing as it could not do OR, it can only do AND. If we used a Topic exchange and used the # wildcard to capture all messages, we would end up processing the very important client messages as well as the lesser important clients.

## #12 - NOT Routing with Alternate Exchanges

Sometimes you want to consume all messages **except** one specific type. None of the four exchange types offer negative matching. Instead we can create a "waste bin" queue and binding for the message you do **not** want. This queue is set up with a very short queue based message TTL so the messages get discarded almost immediately upon arriving.

You configure the exchange with an alternate exchange and wire up your consumer with a queue bound to that alternate exchange. Now you consume all the messages except the one type you don't want.

![Fig 19. Consumer App 2 gets all booking messages except mytravel.com ones](../../_resources/5082ee49429441f5b8af17ef19cbba20.pngformat750w)

Fig 19. Consumer App 2 gets all booking messages except mytravel.com ones

This topology is similar to the one in the Public Message Exchange, Private Consumer Exchange pattern. Consumer 2 sets up a private topic exchange and routes all bookings made from the mytravel.com sales channel to a waste bin queue and then consumes the rest.

Obviously you could just let your consumer consume every message and just discard the messages you don't want. It depends on your specific scenario.

## #13 - Delayed Retry with Cascading Exchanges and Queues

There is a lot of bad advice out there regarding delayed retry routing in RabbitMQ. All delayed retry methods rely on message TTL expiry and dead letter exchanges. Many people do not take into account that only messages at the **head** of the queue get dead lettered. This means you cannot mix delay times in one queue as shorter delayed messages get stuck behind longer delayed messages. Now that warning is said, let's look at this very inventive pattern.

This pattern is taken from NServiceBus. It uses cascading topic exchanges which chain together via dead letter configuration and topic routing. The idea is that we create multiple levels of delay, where each level is responsible for its own fixed delay period. These periods increase to the power of 2. Level 1 as 1 minute, level 2 as 2 minutes, level 3 as 4 minutes, level 4 as 8 minutes etc. Then using binary style routing and binding keys we can move a message between delay queues to achieve any delay period at 1 minute resolution. You could use 20 or so queues, with level 1 at 1 second and achieve second resolution delay times up to a period of years!

![Fig 20. Cascading delay exchanges and queues](../../_resources/a949e1b2ac7143438b6a6a971ff22dc9.pngformat750w)

Fig 20. Cascading delay exchanges and queues

The above diagram shows this cascading, shared delayed retry infrastructure with just three levels. With three levels, and level 1 being 1 minute, we can achieve any delay up to 7 minutes with a 1 minute resolution. Not much but this is a super simplified version.

Each delay queue has configured as its dead letter exchange the exchange of the level below. In this example we see that consumer 1 sends a message for retry with a delay of 5 minutes. Let's see how the message flows through the exchanges and queues:

*   The level 3 exchange has two bindings. The routing key 1.0.1.consumer.app.1 matches the binding key *.*.1.# only. So it gets routed to the Level 3 queue. That queue has a 4 minute message TTL configured. After waiting for 4 minutes it gets dead lettered to the Level 2 exchange.
    
*   The Level 2 exchange has two bindings. The routing key 1.0.1.consumer.app.1 matches the binding key *.0.# only, so the message is routed to the Level 1 exchange.
    
*   The Level 1 exchange has two bindings. The routing key 1.0.1.consumer.app.1 matches the binding key 1.# only. So it gets routed to the Level 1 queue. That queue has a 1 minute message TTL configured. After waiting for 1 minute it gets dead lettered to the Level 0 exchange.
    
*   Both consumers have created bindings from their queues to this exchange. The message matches the #.consumer.app.1 binding and so is routed to the Consumer 1 queue where it gets consumed by Consumer 1 again, 5 minutes after it sent the message to the retry infrastructure.
    

I love this pattern as it shows how the routing primitives that RabbitMQ offers can be combined so inventively.

Some things to remember about this pattern:

*   This is a shared infrastructure approach and the wait times may not be exact under load.
    
*   If the original message had a routing key, it was removed for this pattern to work. I get around this by putting the routing key as a message header upon sending the message for retry, then the consumer can retrieve the original routing key if it needs it.
    
*   If the original message had a TTL and you want that to be respected then you'll need to add that as a message header when you send it for retry and get your application to check it and discard the message if the time period has passed. The reason for this is that when the consumer application sends the message to the retry exchanges, it shouldn't set the message TTL as that might interfere with the retry timing. In any case, when a message is dead lettered any message TTL is stripped from the message.
    
*   Retries and message ordering are fundamentally opposed. If message order is important then delayed retries are probably not a good idea unless you add some logic that detects older messages.
    

## #14 - Delayed Retry with Ephemeral Exchanges and Queues

We can achieve similar results to the previous pattern with ephemeral exchanges and queues. When an application wants to send a message for a delayed retry, it creates a one-off exchange and queue with a guaranteed to be unique name, such as a GUID/UUID.

The ephemeral exchange is configured as follows:

*   Fanout
    
*   Auto-delete
    

The ephemeral queue is configured as follows:

*   a message TTL corresponding to the delay you want.
    
*   its dead letter exchange is the Default exchange.
    
*   a queue TTL that expires a few seconds after the message TTL.
    
*   a binding to the ephemeral exchange.
    

The consumer declares the exchange and queue, then sends the message for delayed retry with the routing key as the name of its own queue. The next series of events occur:

1.  Ephemeral exchange routes the message to the ephemeral queue
    
2.  The message sits in the queue for the message TTL time period
    
3.  The message is dead lettered to the default exchange
    
4.  The default exchange routes the message to the queue that matches the routing key
    
5.  The ephemeral queue reaches its queue TTL period and auto-deletes itself
    
6.  The ephemeral exchange sees that no queues are bound to it and it auto-deletes itself.
    

![Fig 21. Ephemeral exchanges and queues for delayed routing](../../_resources/31d8c815331740bcb9f91f1af4b283a9.pngformat750w)

Fig 21. Ephemeral exchanges and queues for delayed routing

Considerations to take into account:

*   Creating exchanges, queues and bindings is relatively expensive. If you generate high loads with retries then this might put too much pressure on your cluster.
    
*   Like the cascading exchange pattern, the original routing key and message TTL are removed to make this work. See that pattern for more details on that.
    

If you use the Public Message Exchange, Private Consumer Exchange pattern you don't need to rely on the default exchange at all and can configure the consumer's private exchange as the dead letter exchange. This removes the need to set a custom routing key.

![Fig 22. Routing to private consumer exchange instead of Default exchange](../../_resources/6f470df4727e482a8c378f57f159b71f.pngformat750w)

Fig 22. Routing to private consumer exchange instead of Default exchange

## #15 - Delayed Delivery with the Delayed Message Exchange Plug-In

[https://github.com/rabbitmq/rabbitmq-delayed-message-exchange](https://github.com/rabbitmq/rabbitmq-delayed-message-exchange)

This plug-in gives you a new exchange type, the Delayed Message exchange. This exchange can mimic the normal exchange types with the twist that if you put the header "x-delay" on a message, the exchange will delay delivery of the message for the number of milliseconds in your header value.

The downside of this plug-in is that it does not support high availability of messages that are in the delay period. So the loss a node will result in the loss of delayed messages on that node. Another downside is not supporting the "mandatory" flag. We haven't covered that yet as it falls under the [Part 4 Delivery Guarantees](https://jack-vanlightly.com/blog/2017/12/15/rabbitmq-vs-kafka-part-4-message-delivery-semantics-and-guarantees) part of this series, but it plays an important role in message delivery guarantees.

## #16 - Delayed Delivery with Cascading Exchanges and Queues

If the Delayed Message exchange plug-in isn't for you then you could try the casacading exchanges and queues pattern. This is the same pattern as the Delayed Retry with Cascading Exchanges and Queues, except that publishers send directly to the delay exchanges. Because we use a binary like routing keys, this places a burden on the publisher to be able to create the correct routing key. This can work if you create a code library for managing the creation of delayed messages and have control of the publisher.

The benefits of this approach over the plug-in are that we do not lose RabbitMQ HA capabilities.

## #17 - Delayed Delivery with Private Publisher Exchange

If the publisher always wants the **same** delay on all messages then they can declare their own exchange and queue for the purposes of adding the delay.

Let's say that the third publisher of the diagram below needs to delay messages by 5 minutes, for business reason X.

![Fig 23. Three publishers send to a topic exchange](../../_resources/44b07bc6bff44485b32eff070667e31d.pngformat750w)

Fig 23. Three publishers send to a topic exchange

The third publisher declares its own exchange and queue, which means it does not affect other publishers. Because we always want a 5 minute delay we can use a single queue with its dead letter exchange as the main topic exchange.

![Fig 24. One publisher adds a delay with its own exchange and queue](../../_resources/e01390dee6a0456cac2724ddeef3dc0c.pngformat750w)

Fig 24. One publisher adds a delay with its own exchange and queue

This is the simplest method of all but only works with a standard delay time. As stated earlier, messages are only dead lettered from the head of the queue so shorter TTL messages get stuck behind longer TTL messages. You cannot mix various length TTL messages.

## #18 - RPC and Reply-To Queues

In order to do Remote Procedure Call (RPC) style messaging, in general you must use Reply-To queues that the recipient of your message can reply to. This can be tricky to get right and you should really ask yourself if you really need to use a messaging system for RPC. Messaging systems are built with asynchrony and durability in mind. Most of the reasons for using a messaging system are not there when it comes to RPC. But if you still want RPC then you have a few options on RabbitMQ.

But first let's speak more generally of the complexities of RPC over a messaging system. In our case RabbitMQ has a really neat feature that side steps these problems which other messaging systems suffer from. Let's first understand those issues so that we can appreciate the functionality provided by RabbitMQ (feel free to skip to the end of this pattern to view RabbitMQ's nice feature for RPC).

An important question is: do you want to make the call in code just like any other method invocation? If the answer is yes then you need an RPC architecture that is compatible with a stateful context. That is, there is state in an active thread on a particular host that needs the response message to get back to it. If we have a scaled out service of 10 instances, with hundreds of threads per instance, we need that the response message gets delivered back to that same thread on that same host. This rules out some patterns.

However, if we have a stateless context then we are free to choose any of the RPC patterns. If upon making the RPC call the context of that execution ends and all state is either lost or cached with a correlation Id in Redis or something, then we have more freedom to choose different patterns. However this stateless context is not always possible or desired.

Let's review each option, whether it works in a stateful or stateless context and what other trade-offs it might have. We'll explore them using RabbitMQ's exchanges and queues. 

### Fixed Reply-To Queue and Correlation ID

![Fig 25. Single reply-to queue](../../_resources/eaa33f8a8bbe447d9bff3ea509aa1702.pngformat750w)

Fig 25. Single reply-to queue

The outgoing message includes two message headers:

*   reply-to-queue
    
*   correlation-id
    

The recipient sends the message to the Default exchange with the reply-to queue name as the routing key. It also includes the same correlation-id header and value. The original sender consumes this reply to queue and can retrieve any state from a state store using the correlation id.

Obviously this is not compatible with our stateful context. Any of our 10 instances of the application could consume the message.

### Queue Per Application Instance and Correlation Id

![Fig 26. Reply-To queue per application host](../../_resources/514cbdcdacab4943b73610adcaa428c4.pngformat750w)

Fig 26. Reply-To queue per application host

This is the same as the Fixed Queue pattern except that each application instance has a separate queue. This can be compatible with the stateful context if you go through some hoops to get there. First of all if your application is single-threaded, with only one active request at a time then you can be sure that the thread that waits for a response will be the one that receives the response message. But these applications are not common. More likely it is a web application and therefore multi-threaded with multiple active requests. In this case it depends on the capabilities of the language.

C# for example has the ability to store a waiting Task as a TaskCompletionSource object in memory and resume the task at any time. You can use a singleton class to store TaskCompletionSource objects by their correlation ids in a dictionary. The singleton is the only object that consumes messages. When it consumes a message it retrieves the correlation Id from the message, and retrieves the TaskCompletionSource object from a dictionary and resumes it, passing it the message.

This is not trivial code to write however and could be a risky choice depending on your concurrency programming skill levels. Bugs would be difficult to diagnose and fix. If you can find a library to do that for you then this could be a decent option.

### Ephemeral Reply-To Queue with No Correlation Id

![Fig 27. Ephemeral reply-to queue per message](../../_resources/5abfad72bf6e4fa2ad58a60a61153307.pngformat750w)

Fig 27. Ephemeral reply-to queue per message

This is totally compatible with the stateful context and is the simplest pattern. The caller declares a queue with a GUID/UUID as a name and passes that as a message header in the request message. The queue should be made an auto-delete queue so that it automatically deletes itself once the sender has stopped using it. The recipient sends a response message to this queue as indicated in the request message header. Once the sender has consumed the response message it cancels its subscription to the queue and the queue auto-deletes itself.

So what is the trade-off with this simple and elegant design? Performance. Creation of queues can be expensive. If you have a decent amount of traffic then you would need to do trials to see if the constant creation and auto-deletion of queues puts too much burden on the cluster. Other messaging systems don't even have ephemeral exchanges and queues to help with this problem! 

### RabbitMQ's Killer RPC Feature - Direct Reply-To

Finally we get to RabbitMQ's cool RPC feature. Direct Reply-To side steps all these issues by giving you something very similar to the ephemeral pattern but without the performance issues. This is how it works:

*   The sender puts the name of a "pseudo queue" called _amq.rabbitmq.reply-to_ in the reply-to message header. It is a pseudo queue because it is not really a queue at all, but it can be treated as one.
    
*   The sender consumes this _amq.rabbitmq.reply-to_ queue in no-ack mode (more on that in [Part 4 Message Delivery Guarantees](https://jack-vanlightly.com/blog/2017/12/15/rabbitmq-vs-kafka-part-4-message-delivery-semantics-and-guarantees) part of the series).
    
*   The recipient sends the response message to the Default exchange with the routing key as the name of this pseudo queue.
    
*   The consumer gets pushed the message from the RabbitMQ node directly, without it having ever been written to a queue.
    

Obviously we lose high availability guarantees because we cannot use queue mirroring to replicate the message across nodes, but with RPC we don't really need that. Also if the sender gets disconnected then the pseudo queue goes away and no response can be sent. But that is no different than RPC over HTTP.

Basically, just like with HTTP if something goes wrong you just retry the request again and even use patterns such as circuit breaker if need be.

So RabbitMQ has a special custom behaviour for doing RPC that avoids all the pain of real reply-to queues. 

We can create a full message lifecycle that handles transient and non-transient errors in a way that guarantees that we don't lose messages and that we can respond to processing failures in a controlled and managed way.

What does a lifecycle consist of? Basically it is a workflow of possible paths that a message can take. It starts at the publishing of a message, then includes the successful processing of a message, or retries in case of transient failure, a place for unprocessable messages to go to, a place to archive failed messages, the option to discard, returning failed messages to the original consumers etc.

Core to this concept is also that failed messages carry with them all the information practically possible to diagnose the failure. This means:

*   The consumer application
    
*   The exception or error message
    
*   The server
    
*   The time
    
*   Who published the message
    
*   How many times has this message been processed?
    

We build the lifecycle into our messaging code library with a super simple code API for developers to use. Additionally, we ensure that the code API **forces** them to think about the nature of the failure - is it transient or persistent? Do we do immediate retries, delayed retries, discard the message or send it to a failed message queue?

Read my [full article](https://jack-vanlightly.com/blog/2017/6/11/improving-reliability-and-incident-response-via-a-message-lifecycle) on building a message lifecycle with RabbitMQ.

There are more patterns that are related to messaging in general that I have not covered such as:

*   Sagas
    
*   Commands/Events
    
*   Message Processing Audit Feeds
    
*   Change Data Capture (CDC) Feeds
    

RabbitMQ doesn't necessarily have specific features regarding the above patterns which is why I have not explored them in the example patterns section. NServiceBus has good support for Sagas, Commands/Events and Message Processing Audit Feeds. I have not used NServiceBus in production but have played with it a fair amount and analysed its messaging topologies on RabbitMQ. I would recommend looking into it if you are looking at an event driven architecture and are on the .NET stack.

RabbitMQ has more than I have covered here, plug-ins and other functionality I have not mentioned. If I have missed any patterns or topologies then please leave a comment!

*   [Series Introduction](https://jack-vanlightly.com/blog/2017/12/3/rabbitmq-vs-kafka-series-introduction)
    
*   [Part 1 - Two different takes on messaging (high level design comparison)](https://jack-vanlightly.com/blog/2017/12/4/rabbitmq-vs-kafka-part-1-messaging-topologies)
    
*   [Part 2 - Messaging patterns and topologies with RabbitMQ](https://jack-vanlightly.com/blog/2017/12/5/rabbitmq-vs-kafka-part-2-rabbitmq-messaging-patterns-and-topologies)
    
*   [Part 3 - Messaging patterns and topologies with Kafka](https://jack-vanlightly.com/blog/2017/12/8/rabbitmq-vs-kafka-part-3-kafka-messaging-patterns)
    
*   [Part 4 - Message delivery semantics and guarantees](https://jack-vanlightly.com/blog/2017/12/15/rabbitmq-vs-kafka-part-4-message-delivery-semantics-and-guarantees)
    
*   [Part 5 - Fault tolerance and high availability with RabbitMQ](https://jack-vanlightly.com/blog/2018/8/31/rabbitmq-vs-kafka-part-5-fault-tolerance-and-high-availability-with-rabbitmq)
    
*   Part 6 - Fault tolerance and high availability with Kafka