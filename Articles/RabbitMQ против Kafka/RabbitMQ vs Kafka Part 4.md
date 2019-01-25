RabbitMQ vs Kafka Part 4

Both RabbitMQ and Kafka offer durable messaging guarantees. Both offer at-most-once and at-least-once guarantees but kafka offers exactly-once guarantees in a very limited scenario.

Let's first understand what these guarantees mean:

*   **At-most-once delivery.** This means that a message will never be delivered more than once but messages might be lost.
    
*   **At-least-once delivery.** This means that we'll never lose a message but a message might end up being delivered to a consumer more than once.
    
*   **Exactly-once delivery.** The holy grail of messaging. All messages will be delivered exactly one time.
    

_Delivery_ is probably the wrong word for the above terms, instead _Processing_ might be a better way of putting it. After all what we care about is whether a consumer can process a message and whether that is at-most-once, at-least-once or exactly-once. But using the word _processing_ complicates things, exactly-once _delivery_ makes less sense now as perhaps we need it to be delivered twice in order to be able to successfully _process_ it once. If the consumer dies during processing, then we need that the message be delivered a second time for a new consumer.

Secondly, talking about processing introduces the headache of partial failure. There are multiple steps in the processing of a message. It starts and ends with communication between the application and the messaging platform with the application logic in the middle. Partial failure scenarios of the application logic need to be dealt with by the application. If the operations performed are fully transactional and result in _all or nothing_ then that avoids the partial failure of the application logic. But often multiple steps include different systems where transactional behaviour is not possible. When we include the communications between the messaging technology, the application, a cache and a database, can we really guarantee exactly-once processing? The answer is no.

So exactly-once is limited to the scenario where the only output of the message processing is the messaging platform itself and that messaging platform provides strong transactions. With this limited scenario we can process a message, write a message and signal the message was processed, all in a transaction that results in all or nothing. This is what Kafka Streams offers.

However, if all outputs of message processing are idempotent, we can avoid the need for transaction based exactly-once guarantees. If we make the writing of the output idempotent then we can safely receive duplicates without worry. But not all actions can be made idempotent.

**End-to-End Signalling**

What neither offer, and this is by design in all the messaging systems that I have worked with, is that there is no end-to-end confirmation. In fact considering that a message in RabbitMQ could be routed to multiple queues, that doesn't even make sense. Equally in Kafka, multiple different consumer groups can be reading from the same topic. In my experience end-to-end signalling is something people new to messaging often ask about so its best to state that it is not possible clearly at the beginning.

**Chain of Responsbility**

Essentially, publishers cannot know that their messages get consumed. What they can know is that the messaging system has received their messages and has taken responsbility for storing them safely and delivering them to consumers. There is a chain of responsibility that starts with the publisher, then moves to the messaging system and finally the consumer. Each has to behave correctly when it is their turn to be responsible for a message and during the hand-overs of responsibility. This means that you as the developer must write your applications well, so that you don't lose or misuse messages while they are under your control.

**Message Order**

This post is primarily focused on how each platform provides at-most-once and at-least-once delivery. But there is also message ordering. I have already covered message order and processing order in the previous parts of this series so I recommend you read those posts.

In short, both RabbitMQ and Kafka have FIFO ordering guarantees. RabbitMQ maintains ordering at the queue level and Kafka at the topic partition level. The implications of those design choices have already been covered in the previous posts.

## RabbitMQ Delivery Guarantees

The delivery guarantees are achieved by:

*   Message durability - not losing messages once stored in RabbitMQ
    
*   Message acknowledgements - signalling between RabbitMQ and publishers/subscribers
    

### Durability Primitives

**Queue Mirroring**

Queues can be mirrored (replicated) across multiple nodes (servers). For each queue there is a master queue which resides on a single node. Let's imagine we have 3 nodes, 10 queues with 2 mirrors per queue. The 10 master queues and 20 mirrors will be distributed across the 3 nodes. How masters are distributed across the nodes can be configured. When a node goes down:

*   for each master queue on that lost node, a mirror on another node must be promoted to master
    
*   new mirrors on other nodes are created to replace the lost mirrors on that failed node, thereby maintaining the replication factor
    

More on this in Part 5 where we go into detail about fault tolerance.

**Durable Queues**

RabbitMQ has two types of queue: durable and non-durable. Durable queues are persisted to disk and a survive node restart. Durable queues are redeclared on node start-up.

**Persistent Messages**

Just because a queue is durable doesn't mean its messages survive a node restart. Only messages set as _persistent_ by their publisher will be recovered.

With RabbitMQ, the more durable your messages the lower the throughput achievable. So if you have a stream of real-time events and it is no big deal if you lose a few or lose a short period time of that stream, then you might consider not using queue mirroring and publishing all messages as non-persistent. However if you really don't want to lose any messages due to node failure, then use durable queues with mirroring and persistent messages.

### Message Acknowledgements

**Publishing**

Messages can be lost or duplicated during the publishing of a message. It depends on how the publisher behaves. 

**Fire and forget. **Publishers can decide not to use the publisher confirms (message acknowledgements for publishers) and simply send messages in a fire and forget manner. Messages won't get duplicated, but they could easily end up being lost (at-most-once). 

**Publisher Confirms.** When a publisher opens a channel to a broker, it can set the channel to use confirms. The broker will then respond to messages with a:

*   **basic.ack**. Positive acknolwedgement. Messaage received, responsibility for the message now lies with RabbitMQ.
    
*   **basic.nack**. Negative acknowledgement. Something bad happened and the message was not processed. Responsibility for the message remains with the publisher. The publisher could republish for example.
    

In addition to acks and nacks there is the basic.return. Sometimes a publisher needs to know that the message wasn't only received by the RabbitMQ broker but actually persisted to one or more queues. It could happen that a publisher publishes a message to a topic exchange where no subscriber queues have a binding that matches the routing key of that message. In that case the broker simply discards the message. In many scenarios this is fine, in others the publisher wants to know if the message got discarded or not and take action. The **Mandatory** flag can be set to true on a per message basis, and the result is that if the message was not persisted to any queues a **basic.return** is sent to the publisher.

A publisher can choose to wait for an acknowledgement after sending each message, but this seriously reduces throughput. So publishers can instead send a steady stream of messages, limiting themselves to a number of unacknowledged messages. Once they reach the _messages in flight limit_, they pause and wait for all the confirms to come in.

Now that we have multiple messages in flight between the publisher and RabbitMQ, to improve throughput RabbitMQ groups acknowledgements using the **multiple** flag. All messages sent over a channel are assigned a monotonically increasing int value called a Sequence Number. Message acknowledgements include the Sequence Number that the acknowledgement corresponds to. When combined with multiple=true, the publisher needs to keep track of the sequence numbers it has sent so it can know which have been received successully and which ones not. I have written a detailed [post](https://jack-vanlightly.com/blog/2017/3/11/sending-messages-in-bulk-and-tracking-delivery-status-rabbitmq-publishing-part-2) about that subject.

So with confirms we can avoid message loss by either:

*   republishing messages when a nack is received
    
*   persisting the message somewhere when a nack or basic.return is received.
    

**Transactions.** Transactions are not often used in RabbitMQ due to:

*   Unclear guarantees. If messages get routed to mulitple queues or use the mandatory flag then atomicity of transactions is not to be relied upon.
    
*   Poor performance.
    

Frankly, I've never used them, they add no extra guarantees over publisher confirms and just seem to increase uncertainty about how to interpret message acknowledgents that result from a commit.

**Connection/Channel Exceptions. **In addition to message acknolwedgements, publishers need to take into account connection failures and broker failures. Both of which result in the loss of the channel. Losing the channel loses the ability to receive any outstanding message acknowledgments. At this point the publisher can choose to risk message loss or choose to risk message duplication.

If it was a broker failure, may be the failure happened while the message was in an OS buffer, or being parsed and therefore has been lost. Or perhaps the message got persisted to a queue and then just before the broker would have sent the confirm, it died. In this case the message was successfully delivered.

The same goes for connection failure. Did it fail during the transmission of the message? Or after the message was persisted to a queue but before the ack was received?

It is impossible for the publisher to know, so we choose either:

*   to not republish with potential message loss
    
*   to republish with potential message duplication
    

If the publisher has many messages in flight, then the problem is excerbated. One thing that a publisher can do is give consumers a hint by adding a custom message header to indicate that the message was republished. Consumers could choose to check for that header and when it exists perform extra deduplication checks (if they don't already).

**Consumers**

Consumers have two options regarding acknowledgements:

*   No Ack mode.
    
*   Manual Acknowledgment mode.
    

**No Ack mode**, or automatic acknowledgement mode, is dangerous. First of all, as soon as a message is sent to your application, the message is removed from the queue. This can result in message loss when:

*   the connection fails before the message is received
    
*   the message is still in an internal buffer when the application fails
    
*   the message processing fails
    

Secondly, we lose back-pressure as a way of controlling message delivery rate. With manual acknowledgement mode we can specify a prefetch (or QoS) to limit the number of unacknowledged messages that the application has at a time. Without this feature, RabbitMQ sends messages as fast as the connection will allow which might be faster than the consumer can process them, resulting in buffers over filling and memory exception occuring.

**Manual Acknowledgment mode**. The consumer must manually acknowledge each message it receives. The consumer can set the prefetch to a number above 1 and process multiple messages concurrently. It can choose to send an acknowledgement per message, or use the **multiple** flag and acknowledge multiple messages at a time. The batching of acknowledgements improves throughput.

When a consumer opens a channel, messages delivered over channel receive a monotonically increasing **Delivery Tag** int value. This is included in each acknowledgement and acts as a message identifier.

The acknowledgments are:

*   **basic.ack**. RabbitMQ will then remove the message from the queue. The multiple flag can be used here.
    
*   **basic.nack**. The consumer must set a flag on a nack to instruct RabbitMQ whether or not to _requeue_ the message. Requeuing means that the message is placed back in the queue at the head. From there it is delivered to a consumer again (even the same consumer). The basic.nack supports the multiple flag.
    
*   **basic.reject**. The same as the basic.nack but without multiple flag support.
    

So semantically, a basic.ack and a basic.nack with requeue=false are the same, they both result in the message being removed from the queue.

The next question is _when do_ you send an acknowledgement? If the message processing is quick then you would want to send it once the message has been processed (successfully or unsuccessfully). But if you use RabbitMQ as a work queue, and the job takes many minutes to perform then sending an acknowledgement at the end can be problematic. The problem is that if the channel closes, then any unacknowledged messages get requeued immediately resulting in a duplicate delivery.

**Connection/broker failure.** 

If the connection dies, or a broker failure occurs that causes the channel to die, then all unacknowledged messages get requeued and redelivered. This is great for preventing message loss, but bad for duplication.

The longer a consumer has an unacknowledged message, the higher the risk of redelivery. When a message gets redelivered RabbitMQ sets the redelivered flag to true. So at least the consumer has an indication that the message might have already been processed.

**Idempotency**

If you need idempotency and guarantees of no message loss then that means you need to build in some message duplicate detection or other idempotency patterns. If checking for message duplication is costly, then you can use a strategy where publishers always add a custom header on republishing and the consumer checks for this header and the redelivered flag.

### Conclusions

RabbitMQ provides strong reliable, durable messaging guarantees but there are many ways to screw up. 

Here is a list of things to remember:

*   Use queue mirroring, durable queues, persistent messages, publisher confirms, the mandatory flag and manual consumer acknowledgment if you want strong at-least-once guarantees.
    
*   If you go with at-least-once then you may need to add a deduplication or idempotency mechanism due to message delivery duplication.
    
*   If you don't care about message loss so much as you care about low latency and high scaling then look at no mirroring, non-persistent messages and no publisher confirms. I would probably still keep manual consumer acknowledge due to being able to rate limit message delivery via the prefetch limit. Though you would need to batch acknowledgements with the multiple flag.
    

## Kafka Delivery Guarantees

The delivery guarantees are achieved by:

*   Message durability - not losing messages once stored in a topic
    
*   Message acknolwedgements - signalling between Kafka (and possibly ZooKeeper) and publishers/subscribers
    

### A Note About Batching

One difference between RabbitMQ and Kafka is the use of message batches when sending and consuming messages.

With RabbitMQ we can achieve something similar to batching by:

*   pausing publishing every x messages until all acks have been received. RabbitMQ will usually group acknolwedgements using the multiple flag.
    
*   consumers setting a prefetch and grouping acks using the multiple flag
    

But messages are not sent in batches, it is more about allowing a constant stream of message deliveries with the grouping of acks into a single message using the multiple flag. This is what TCP does.

With Kakfa there is more explicit message batching. The reason for batching is performance but there are tradeoffs, the same tradeoffs that RabbitMQ has regarding the number of unacknowledged messages in flight. The more messages that are in flight when a failure occurs means more message duplication or double processing.

Kafka can more effectively work with batching on the consumer side because work is distributed via partitions rather than competing consumers. Each partition has a single consumer, so even the use of large batches does not impact work distribution. However with RabbitMQ, if we use the deprecated pull API to read large batches then we can end up with very unbalanced loads across competing consumers and high effective processing latencies. RabbitMQ by design is unsuited to processing messages in batches.

### Durability Primitives

**Log Replication**

For fault tolerance, Kafka has a master-slave architecture at the log partition level where masters are called leaders and slaves are called followers. Each partition leader can have multiple followers. When a server that hosts the leader fails then a follower gets promoted to be the leader. Kafka gives the producer and the administrator the configuration options to ensure that no loss of stored messages occurs with only a short interruption of service. The more durability you opt for, the more write latency you'll get though.

Kafka has the concept of In Sync Replicas (ISR). Each replica can be in or out of sync. In sync means that they have been up-to-date with the leader within a short period of time (the last 10 seconds by default). Replicas can become out-of-sync when they fall behind. This could happen due to network latency, issues with the host VM etc. Message loss can only occur when a leader fails and no In Sync Replicas exist that are fully up-to-date with the leader. I cover this is much greater detail in [Part 6](https://jack-vanlightly.com/blog/2018/9/2/rabbitmq-vs-kafka-part-6-fault-tolerance-and-high-availability-with-kafka).

**Message Acknowledgements and Offset Tracking**

Due to how Kafka stores messages and how consumers consume messages, Kafka relies on message acknowledgement for producers and offset tracking for consumers. 

**Producer Message Acknowledgement**

When a producer sends a message it tells the Kafka broker what kind of acknowledgment is expects via the Acks property:

*   No acknowledgement, fire and forget. Acks=0.
    
*   Leader has persisted the message. Acks=1
    
*   Leader and all In Sync Replicas have persisted the message. Acks=All
    

Messages can be duplicated during message publishing for the same reasons as with RabbitMQ. If a broker failure occurs or the network connection fails during publishing then the producer will be forced to resend the messages for which it has not received an ack (if it doesn't want to lose the message). But it is quite possible that the message or messages were persisted and replicated.

However, Kafka comes with a nice message deduplication feature that avoids the duplication problem. There are some limitations though:

*   _enable.idempotence_ set to true
    
*   _max.in.flight.requests.per.connection_ set to 5 or less
    
*   _retries_ flag set to 1 or higher
    
*   _acks_ set to all
    

Therefore if you use batches of 6 or more messages or acks=0/1 for throughput then you can't take advantage of this feature.

**Consumer Offset Tracking**

Consumers need to save their offset so that if they fail, a new consumer can continue from where the failed consumer left off. The offset can be stored in ZooKeeper or in another Kafka topic.

Once a consumer reads a batch of messages from a partition, it has options for when it saves its offset:

*   Auto-commit on an interval. As the messages get processed, the client library handles periodic commits of the offset. This makes working with offset commits very easy from the programmers perspective, and can also be good for performance. But it does increase the risk of duplicate delivery if the consumer fails. The consumer might have processed a bunch of messages when it failed, before the corresponding offset could be committed.
    
*   Manual commit on receipt of messages. This corresponds to _at-most-once_ delivery semantics as no matter when the consumer might fail, a message will not be processed twice though it might be left unprocessed. For example if 10 messages are being processed and the consumer fails on message 5 then only 4 messages will have been processed, the others will be skipped as the next consumer will start from the messages after this batch.
    
*   Manual commit at the end, once all messages have been processed. This corresponds to _at-least-once_ delivery semantics. No matter when the consumer might fail, no message will ever end up unprocessed, though messages could end up being processed multiple times. For example if 10 messages are being processed and the consumer fails on message 5 then all 10 will be read again by the new consumer and so 4 messages will be processed twice.
    
*   Manual commit one-by-one. This reduces message duplication but at a large cost of throughput!
    

The _exactly-once_ guarantee is limited to [Kafka Streams](https://kafka.apache.org/documentation/streams/) which is a Java library. If you use Java then I highly recommend looking at it. The main issue with exactly-once is that the output of processing a message and the saving of the offset need to occur in a transaction. For example, if the output were to send an email then we could not make this exactly-once. If we send the email but then the consumer fails before it can save the offset then the new consumer will send that same email again.

A Kafka Streams Java application whose output of processing a message is to write a new message to a different topic can achieve exactly-once processing. This is because we can use Kafka's transactions functionality to write the message and save the offset (writing a message to a topic also) in a transaction. They both succeed or both fail so no matter when the consumer fails, we'll not write the output message twice.

**About Transactions and Isolation Levels**

The main use case for transactions in Kafka are the _read-process-write_ scenarios mentioned above. Transactions can span multiple topics and partitions. A producer opens a transaction, writes a batch of messages and commits the transaction.

When consumers use the default _read uncommited_ isolation level, they see all messages regardless of their transaction status (committed, uncommitted, aborted). When consumers use the _read commited_ isolation level, they do not see uncommitted or aborted messages. They will only be able to consume the messages of a transaction once the transaction has been committed.

You might now be wondering about how the read committed isolation level affects message order guarantees. The answer is that it doesn't. Consumers still read all messages in the correct order and stop at any uncommitted message. So open transactions block read committed consumers. The Last Stable Offset (LSO) is the offset before the first open transaction offset. It is up to the LSO that read committed consumers can read up to.

## Round Up

Both technologies offer mechanisms for reliable, durable messaging, so if reliability is a real concern then you can be sure that both have comparably strong guarantees. However I think that Kafka currently has the edge given the idempotent publishing and the fact that mismanaging the offset doesn't always mean that a message is lost forever as it will still be sitting their in the partition. 

The glaring omission from this post are details of clustering and replication that both RabbitMQ and Kafka provide. So don't draw any conclusions about durability quite yet. Check out parts 5 and 6 to understand better the durability guarantees of messages stored on the brokers.

For now here are some takeaways:

*   Both offer at-most-once and at-least-once delivery guarantees.
    
*   Both offer message replication
    
*   Both have the same tradeoffs regarding throughput and the potential for message duplication. Although Kafka offers idempotent publishing, it only offers that up to a certain volume of traffic.
    
*   Both have controls regarding the number of unacknowledged messages in flight.
    
*   Both have guarantees regarding the order of message delivery.
    
*   Kafka offers real transaction support, with the primary use case being _read-process-write_. Care needs to be taken to avoid impacting throughput.
    
*   With Kafka, if a consumer does not process some messages due to a failure and a poor use of offset tracking, we can rewind the offset back again (given that this is detected). With RabbitMQ those messages are gone.
    
*   Kafka can leverage the performance benefits of batching due to its partitions, whereas RabbitMQ is not built for batching due to the push model with competing consumers.
    

Series links:

*   [Part 1 - Two different takes on messaging (high level design comparison)](https://jack-vanlightly.com/blog/2017/12/4/rabbitmq-vs-kafka-part-1-messaging-topologies)
    
*   [Part 2 - Messaging patterns and topologies with RabbitMQ](https://jack-vanlightly.com/blog/2017/12/5/rabbitmq-vs-kafka-part-2-rabbitmq-messaging-patterns-and-topologies)
    
*   [Part 3 - Messaging patterns and topologies with Kafka](https://jack-vanlightly.com/blog/2017/12/8/rabbitmq-vs-kafka-part-3-kafka-messaging-patterns)
    
*   [Part 4 - Message delivery semantics and guarantees](https://jack-vanlightly.com/blog/2017/12/15/rabbitmq-vs-kafka-part-4-message-delivery-semantics-and-guarantees)
    
*   [Part 5 - Fault tolerance and high availability with RabbitMQ](https://jack-vanlightly.com/blog/2018/8/31/rabbitmq-vs-kafka-part-5-fault-tolerance-and-high-availability-with-rabbitmq)
    
*   [Part 6 - Fault tolerance and high availability with Kafka](https://jack-vanlightly.com/blog/2018/9/2/rabbitmq-vs-kafka-part-6-fault-tolerance-and-high-availability-with-kafka)