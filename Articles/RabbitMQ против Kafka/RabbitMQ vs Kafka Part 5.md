RabbitMQ vs Kafka Part 5

Fault tolerance and High Availability are big subjects and so we'll tackle RabbitMQ and Kafka in separate posts. In this post we'll look at RabbitMQ and in Part 6 we'll look at Kafka while making comparisons to RabbitMQ. This is a long post, even though we only look at RabbitMQ, so get comfortable.

In this post we'll look at the strategies for fault tolerance, consistency and high availability (HA) and the trade-offs each strategy makes. RabbitMQ can operate as a cluster of nodes and as such can be classed as a distributed system. When it comes to distributed data systems we often speak about consistency and availability.

We talk about consistency and availability with distributed systems because they describe how the system behaves under failure. A network link fails, a server fails, a hard disk fails, a server is temporarily unavailable due to GC or a network link is lossy or slow. All these things can cause outages, data loss or data conflicts. It turns out that it is generally not possible to provide a system that is ultimately consistent (no data loss, no data divergence) and available (will accept reads and writes) under all failure modes.

We'll see that consistency and availability are at two ends of a spectrum and you'll need to choose which of those you'll optimize for. The good news is that with RabbitMQ this is a choice that you can make. It gives you the nerd knobs required to tune it for greater consistency or greater availability.

In this post we'll be paying close attention to what configurations produce data loss of acknowledged writes. There is a chain of responsibility between producers, brokers and consumers. Once a message has been handed off to a broker, it is the broker's job not to lose that message. When the broker acknowledges receipt of a message to the publisher, we don't expect that message to be lost. But we'll see that this indeed can happen depending on your broker and publisher configuration.

## Single Node Durability Primitives

### **Durable Queues/Exchanges**

RabbitMQ has two types of queue: durable and non-durable. All queues are persisted to the Mnesia database. Durable queues are redeclared on node start-up and so survive a restart, system crash or server failure (as long as the data survives). This means as long as you declare your exchanges and queues to be durable, your exchange/queue infrastructure will come back online.

Non-durable queues and exchanges are deleted on start-up.

### **Persistent Messages**

Just because a queue is durable doesn't mean its messages survive a node restart. Only messages set as _persistent_ by their publisher will be recovered. Persistent messages do put more load on the broker, but if message loss is unacceptable then persistent messages are a no-brainer.

![Fig 1. Durability matrix](../../_resources/0199973c3a374b4ab8118c81b8f8c473.pngformat750w)

## Clustering with Queue Mirroring

In order to survive the loss of a broker we need redundancy. We can join multiple RabbitMQ nodes into a cluster and then add additional redundancy by replicating queues across multiple nodes. That way if a single node dies we don't lose data and we stay available. 

A mirrored queue has:

*   one master that receives all reads and writes
    
*   one or more mirrors that receive all messages and meta data from the master. These mirrors do not exist for scaling out the reading of queues but solely for redundancy.
    

![Fig 2. A mirrored queue.](../../_resources/dd8e5456ae1149cba50225371d5381aa.pngformat750w)

You can make a queue mirrored by setting a policy. In that policy you can choose the replication factor and even the nodes that the queue should be hosted on. Examples are:

*   ha-mode: all
    
*   ha-mode: exactly, ha-params: 2 (one master and one mirror)
    
*   ha-mode: nodes, ha-params: rabbit@node1, rabbit@node2
    

## Publisher Confirms

Publisher confirms are necessary for achieving consistent writes. Without publisher confirms it is possible to lose messages. A confirm is sent to a publisher once a message has been written to disk. RabbitMQ does not write messages to disk on receipt but on a periodic basis, in the region of a few hundreds ms. When a queue is mirrored then an ack is only sent once all mirrors have also written their copy of the message to disk. This means that using confirms adds more latency, but if data safety is important to you then they are necessary.

## Queue Fail-Over

When a broker is shutdown or dies, all the queue masters on that node go with it. The cluster then selects the oldest mirror of each master and promotes it to be the new master.

![Fig 3. Multiple mirrored queues and their policies.](../../_resources/4b5c6513c21f4f50aab18815eecfbb9d.pngformat750w)

Fig 3. Multiple mirrored queues and their policies.

Broker 3 dies. Notice that Queue C has its mirror on Broker 2 promoted to master. Also note that a new mirror has been created for Queue C on Broker 1. RabbitMQ will always try to maintain the replication factor set out in your policies.

![Fig 4. Broker 3 dies which causes a fail-over for Queue C.](../../_resources/3e02dcdc7c2c428da6d3949b211008e6.pngformat750w)

Fig 4. Broker 3 dies which causes a fail-over for Queue C.

Next Broker 1 dies! We only have a single broker left. The Queue B mirror get promoted to master.

![RabbitMqQueueFailover3.PNG](../../_resources/06c11bda3ed347d5bfe6e6c9d8174406.pngformat750w)

We bring back up Broker 1. Whether or not the data survived the loss and recovery of the broker, all mirrored queue messages are discarded on start up. This is important to note as it has implications. We'll look at those implications soon. So Broker 1 is now a member of the cluster again and the cluster tries to honor the policies and so creates mirrors on Broker 1.

In this case the loss of Broker 1 was total, the data too, so Queue D that was not mirrored was lost completely.

![Fig 5. Broker 1 comes back online.](../../_resources/29debf4ea3a64cabbf2865f908f34bbd.pngformat750w)

Fig 5. Broker 1 comes back online.

Broker 3 is now brought online and Queue A and B get mirrors created on it in order to satisfy their HA policies. But now all the master queues are on a single node! This is not ideal, we would prefer to have our masters distributed evenly between the nodes. There are no great options for rebalancing masters unfortunately. We'll come back to this problem later on as we need to cover queue synchronization first. 

![Fig 6. Broker 3 comes back online. All masters on one node!](../../_resources/bd79ba8e17c24cb3b11f04aeace9a7bc.pngformat750w)

Fig 6. Broker 3 comes back online. All masters on one node!

So you should now have an idea of how mirrors provide redundancy and perform fail-overs. This gives us availability in the face of node failures and also protection from data loss. But we are not finished yet because in fact it is more complicated than this.

## Synchronization

When a new mirror is created all new messages will always be replicated to that mirror and any others. Regarding the existing data on the master, we can choose to replicate them to the new mirror so that it is a full copy of the master. We can also choose not to replicate the existing messages and let the master and the new mirror converge over time as new messages arrive at the tail and existing messages are drained from the head of the master.

This synchronization is either automatic or manual and is controlled via a queue policy. Let's look at an example.

We have two mirrored queues. Queue A has automatic synchronization and Queue B has manual synchronization. Both queues have 10 messages.

![Fig 7. Two queues with different synchronization modes.](../../_resources/6ac392a767994c20b38fd06431b8dad9.pngformat750w)

Fig 7. Two queues with different synchronization modes.

![Fig 8. Broker 3 is lost.](../../_resources/9904378c93eb464d9cd2e77999b69e40.pngformat750w)

Broker 3 comes back online. The cluster creates a mirror for each queue on the new node and automatically synchronizes the new Queue A mirror with the master. The new Queue B mirror however remains empty. So we have full redundancy of Queue A and only a redundancy of one mirror for the existing messages of Queue B.

![Fig 9. The new Queue A mirror gets all the existing messages and new Queue B mirror does not.](../../_resources/7a127703bbfd4ec2b555d6fc683e87fd.pngformat750w)

Fig 9. The new Queue A mirror gets all the existing messages and new Queue B mirror does not.

Both queues get another 10 messages delivered. Then Broker 2 fails and Queue A fails-over to the oldest mirror which is on Broker 1. No data loss occurs in the fail-over. Queue B has 20 messages in the master and only 10 in the mirror as it never got replicated those original 10 messages.

![Fig 10. Queue A fails-over to Broker 1 without message loss.](../../_resources/477522511645463992aef150b31d7407.pngformat750w)

Fig 10. Queue A fails-over to Broker 1 without message loss.

Another 10 messages arrive at both queues. Now Broker 1 fails. Queue A has no problem failing-over to the mirror on Queue A. It does so without message loss. Queue B however has a problem. At this point we can either optimize for availability or consistency. 

If we want to optimize for availability then we need that the **_ha-promote-on-failure_** policy to **_always_**. This is the default value so we can just not specify the policy at all. This basically means that we allow fail-overs to unsynchronized mirrors. Doing so will result in message loss but keeps the queue available for reads and writes.

![Fig 11. Queue A fails-over to Broker 3 without message loss. Queue B fails-over to Broker 3 losing 10 messages.](../../_resources/9a693be94c314ab5bee74fc5c8be9670.pngformat750w)

Fig 11. Queue A fails-over to Broker 3 without message loss. Queue B fails-over to Broker 3 losing 10 messages.

We could also set ha-promote-on-failure to when-synced. This will prevent a fail-over and wait for Broker 1 to come back online with its data. Once back online, the queue will be available again with the master on Broker 1 with no data loss. Availability is sacrificed for data safety. Except this mode can be dangerous and can even cause total data loss, we'll look at that soon.

![Fig 12. Queue B remains unavailable after the loss of Broker 1.](../../_resources/58dd5e6c3be54e168e1a870fbf8ad647.pngformat750w)

Fig 12. Queue B remains unavailable after the loss of Broker 1.

Right now you may be thinking "Why would I ever not use automatic synchronization?". The answer is that synchronization is a blocking operation. The master cannot perform any reads or writes during synchronization!

Let's look at an example. Now we have very large queues. Why might they be so large? There could be multiple reasons:

*   The queues are not being actively consumed
    
*   They are high velocity queues and consumers are running slow right now
    
*   They are high velocity queues and an outage occurred and consumers are catching up
    

![RabbitMqSynchronizationCost1.PNG](../../_resources/209698f7c4854d1fb46281fc27df035d.pngformat750w)

Fig 12. Two large queues with different synchronization modes.

Now Broker 3 dies.

![Fig 13. Broker 3 dies, leaving a master and one mirror per queue.](../../_resources/10143d1466184301b2154ba7364be6cf.pngformat750w)

Fig 13. Broker 3 dies, leaving a master and one mirror per queue.

Broker 3 comes back online and new mirrors are created. The Queue A master starts replicating its existing messages to the new mirror and during this time the queue is unavailable. It takes two hours to replicate the data causing two hours of downtime for that queue!

However Queue B remains available throughout this period. It has sacrificed some redundancy to remain available.

![Fig 14. Queue A remains unavailable during synchronization.](../../_resources/393869ed453948d792fa068fbcc49e90.pngformat750w)

Fig 14. Queue A remains unavailable during synchronization.

Two hours later, Queue A becomes available and can start accepting reads and writes again.

### Rolling Upgrades

This blocking behaviour during synchronization makes rolling upgrades of clusters with very large queues problematic. At some point the node hosting the master queue will need to be restarted which means either failing-over to a mirror or making the queue unavailable during the server upgrade period. If we choose a fail-over then we’ll lose messages if the mirrors are not synchronized. The default is not to fail-over to an unsynchronized mirror during the shutdown of a broker. This means that once the broker comes back up we did not lose any messages, the only impact was down-time for the queue. You can control the shutdown behaviour with the ha-promote-on-shutdown policy. You can set it to one of two values:

*   always = fail-over to unsynchronized mirrors enabled
    
*   when-synced = only fail-over when a synchronized mirror is available, else make the queue unavailable for reads and writes. When the broker comes online again the queue will also come back online
    

So either way you look at it, with large queues you have choose between data loss and unavailability.

### When Availability Means Better Data Safety

There's one more complication to take into account before you make a decision. While automatic synchronization is better for data redundancy, is it still better for data safety? Sure RabbitMQ itself will be less likely to lose messages as it has better redundancy of existing messages, but what happens to new messages being sent by your publishers?

You need to think about:

*   Can my publisher simply return an error upstream and the upstream service or user can retry later?
    
*   Can my publisher persist the message locally or to a database so it can retry later?
    

If the answer is that your publisher can only drop the message then in fact availability is also better for data safety.

So it is a balancing act and the decision depends on your situation.

## Problems With ha-promote-on-failure=when-synced

The idea of the **_ha-promote-on-failure_** = **_when-synced_** is that we prevent failing-over to an unsynchronized mirror and thereby avoid data loss. The queue remains unavailable for reads or writes. Instead we bring try back the lost broker with its data intact so that it can resume being the master, with no data loss. 

But, and this is a big but, if the broker lost its data, then we have a big problem, the queue has been lost! All data gone! Even if you have mirrors that are mostly caught up with the master, those mirrors are discarded too.

To readd a node with the same name we tell the cluster to forget the lost node (using _rabbitmqctl forget\_cluster\_node_ command) and start a new broker with the same hostname. While the cluster still remembers our lost node, it remembers that our queue existed and has unsynchronized mirrors. When the cluster is told to forget the lost node, that queue is forgotten as well. We must now redeclare it. We lost all data even though we had mirrors that had a partial set of the data. It would have been better to have failed-over to an unsynchronized mirror!

So using manual synchronization (and not performing synchronization) coupled with _ha-promote-on-failure=when-synced_ is in my opinion pretty dangerous. The docs say it exists for data safety but it is a double edged sword.

## Rebalancing Masters

As promised, we come back to our problem of having masters concentrated on a single node or a couple of nodes. This can even happen as the result of a rolling upgrade of your cluster. In a three node cluster you'll end up with all your masters on one or two nodes.

Rebalancing masters can be difficult for two reasons:

*   There are no great tools for doing rebalancing
    
*   Queue synchronization
    

There is a 3rd party [plugin](https://github.com/Ayanda-D/rabbitmq-queue-master-balancer) for rebalancing masters that is not supported. Pivotal says "The plugin has some additional configuration and reporting tools, but is not supported or verified by the RabbitMQ team. Use at your own risk."

There is also a trick of using HA policies to move a master. Pivotal provide a script that uses this trick [here](https://github.com/rabbitmq/support-tools/blob/master/scripts/rebalance-queue-masters). It works by:

*   Removing all mirrors via a temporary policy that has higher priority than the existing HA policy
    
*   Changing the temporary HA policy to use "nodes" mode specifying the node where you want the master to be migrated to.
    
*   Synchronizing the queue to force the migration
    
*   Once migration is complete, removing the temporary policy. The original HA policy now takes precendent and the desired number of mirrors are created.
    

The downside of this approach is that is might not be feasible if you have large queues or strict redundancy requirements.

Now let's look at how RabbitMQ clusters deal with network partitions.

## Network Partitions

Distributed system nodes are separated by network links and network links can and will go down. How often depends on your on-premise infrastructure or the reliability of your chosen cloud. Either way, distributed systems need to be able to cope with them. Again we are left with the choice of choosing availability or consistency, and again the good news is that RabbitMQ gives both options (just not at the same time).

With RabbitMQ we have two primary options:

*   Allow split-brain. This provides availability but can provoke data loss when the split-brain is resolved.
    
*   Disallow split-brain. This can cause some short-lived disruption to availability depending on how your clients connect to the cluster. It can also cause complete unavailability in a two node cluster.
    

But what is a split-brain? It is where a cluster is divided in two because the network links cut-off part of the cluster. On each side of the partition, mirrors get promoted to master. This means we end up with more than one master per queue.

![Fig 15. A master and two mirrors, each on a separate node. Then a network partition occurs and one mirror is separated. The separated node, seeing that the other two nodes have gone away promotes its mirrors to master. Now we have two masters and both can be written to and read from.](../../_resources/3b1d6e4af82443069ca5cf8d537e1b08.pngformat750w)

Fig 15. A master and two mirrors, each on a separate node. Then a network partition occurs and one mirror is separated. The separated node, seeing that the other two nodes have gone away promotes its mirrors to master. Now we have two masters and both can be written to and read from.

So if publishers end up writing to both masters we'll have two diverging copies of the queue.

RabbitMQ provides different partition modes which favor either availability or consistency.

### **Ignore Mode (Default)**

This mode opts for availability. When a partition happens, split-brain occurs. When the partition is resolved, the administrator has to decide which side of the partition wins. The losing side needs to be restarted and any data that only existing on that side of the partition is lost.

![Fig 16. Three publishers are connected to three brokers. Internally the cluster routes all requests to the master queue on Broker 2.](../../_resources/43e21010a4c841c38b83683596875caf.pngformat750w)

Fig 16. Three publishers are connected to three brokers. Internally the cluster routes all requests to the master queue on Broker 2.

Now we lose Broker 3. Broker 3 seeing that the other brokers have gone, promotes its mirror to master. We now have split-brain.

![Fig 17. Split-brain. Writes go to two masters and the two copies diverge.](../../_resources/1517708cd0a741b9abc58d2799aee626.pngformat750w)

Fig 17. Split-brain. Writes go to two masters and the two copies diverge.

The network partition gets resolved but the split-brain continues. The administrator must manually resolve the split-brain by choosing a losing side of the partition. In the below case the administrator shutdown Broker 3 and brings it back. Any messages not consumed from Broker 3 are lost when the Broker rejoins the cluster.

![Fig 18. Administrator shutdown Broker 3](../../_resources/5b10d99861184f62855c2213b72c0e92.pngformat750w)

Fig 18. Administrator shutdown Broker 3

![Fig 19. Administrator starts-up Broker 3 and it rejoins the cluster, losing any messages that remained on that Broker.](../../_resources/013c5c08f36a4785be7dca03efcb5d58.pngformat750w)

Fig 19. Administrator starts-up Broker 3 and it rejoins the cluster, losing any messages that remained on that Broker.

Throughout the network partition and afterward the cluster and that queue were available for reads and write.

### Autoheal Mode

This is exactly the same as Ignore mode except that the cluster itself will automatically choose a losing side of the partition. The losing side rejoins the cluster empty, losing any messages that were not consumed and only sent to that side of the partition.

### Pause Minority

If we don't want to allow a split-brain scenario, then our only option is to refuse reads and writes on the minority side of a partition. When a Broker sees that it is on the minority side of a partition it pauses itself. This means that it closes down any existing connections and refuses any new connections. It checks once per second to see if the partition has resolved itself. Once the partition has resolved itself it will unpause itself and rejoin the cluster.

![Fig 20. Three publishers are connected to three brokers. Internally the cluster routes all requests to the master queue on Broker 2.](../../_resources/fc4f39a5b87644e3a98c0fef9dfbd4d9.pngformat750w)

Fig 20. Three publishers are connected to three brokers. Internally the cluster routes all requests to the master queue on Broker 2.

Then Broker 1 and 2 are separated from Broker 3 by a network partition. Rather than promote its mirror to a master, Broker 3 pauses itself becoming unavailable.

![Fig 21. Broker 3 pauses itself, disconnected any clients and refusing connection requests.](../../_resources/af1ad9da67a049bebc4059e7ffc4efef.pngformat750w)

Fig 21. Broker 3 pauses itself, disconnected any clients and refusing connection requests.

Once the partition is resolved it rejoins the cluster.

Let's look at another example where the master is on Broker 3.

![Fig 22. Master on Broker 3](../../_resources/703ddc8018f24acab4f605500a0f979c.pngformat750w)

Fig 22. Master on Broker 3

Then the same network partition occurs. Broker 3 pauses itself as it is on the minority side. On the majority side the nodes see that Broker 3 is gone and the oldest mirror between Broker 1 and 2 is promoted to master.

![Fig 23. Fail-over to Broker 2 while Broker 3 is unavailable.](../../_resources/ef180e714da74a569f3d9e6e4ee5e01d.pngformat750w)

Fig 23. Fail-over to Broker 2 while Broker 3 is unavailable.

When the partition is resolved, Broker 3 rejoins the cluster.

![Fig 24. The cluster is back to normal again.](../../_resources/ec9fe981c5d94a819a06b4ca464e7c00.pngformat750w)

Fig 24. The cluster is back to normal again.

What is important to realize here is that we get consistency, but we can also get availability **_if_** we can successfully route clients to the majority side of the partition. I personally would choose Pause Minority for most situations but it really depends on your particular use case.

Making sure clients can successfully connect to a node is important for availability. Let's look at our options.

## Ensuring Client Connectivity

We have a few options for directing clients to the majority side of a partition or to the nodes that are live (after one node has failed). First let's remember that a given queue is hosted on a specific node, but the exchanges and policies are replicated across all nodes. Clients can connect to any node and internal routing will make sure the clients get connected to the right node. But when a node is paused it refuses connections, so clients must connect to a different node. When a node is down it cannot do much either.

Our options are:

*   Access the cluster via a load balancer that does simple round robin and clients perform connection retries until successful. If a node is down or paused then the connection attempts to that node will fail, but subsequent attempts will be made to other servers (in a round robin way). This would work for short lived network partitions, or a downed server that will be brought back quickly.
    
*   Access the cluster via a load balancer and remove the paused/downed nodes from the list as soon as detected. If you can make that change quickly and if clients can perform connection retry attempts then we should get continued availability.
    
*   Give each client the list of all nodes and the client randomly chooses one node when they connect. If they get a connection refused error then they switch to another node in the list until they can connect.
    
*   Use DNS to switch to accesses away from a downed/paused node. This would require a short TTL.
    

## Conclusions

RabbitMQ clustering is a mixed bag. The most serious deficiencies are that:

*   nodes that rejoin a cluster throw away their data
    
*   synchronization is blocking and causes queue unavailability.
    

All the hard decisions stem from these two design decisions. If RabbitMQ could keep data around when rejoining a cluster then synchronization would be faster. If it could make synchronization non-blocking then it would better support large queues. Fixing those two problems would make RabbitMQ a much better choice for a fault tolerant and highly available messaging technology. I would hesitate to recommend RabbitMQ with clustering when:

*   The network is not rock solid
    
*   Storage is not rock solid
    
*   You have very large queues
    

Regarding settings, for high availability think about:

*   ha-promote-on-failure=always
    
*   ha-sync-mode=manual
    
*   cluster\_partition\_handling=ignore or autoheal
    
*   Persistent messages
    
*   Ensure your clients can still connect to a live node when a node goes down
    

For consistency (data safety) think about:

*   use Publisher Confirms and Manual Acknowledgements on the consumer side
    
*   ha-promote-on-failure=when-synced if publishers are able to retry later on AND if you have VERY reliable storage! Else go for _always_.
    
*   ha-sync-mode=automatic (but for large inactive queues you may need to consider manual mode, also think about whether unavailability might cause message loss)
    
*   Pause Minority mode
    
*   Persistent messages
    

There is still more to fault tolerance and high availability, like how to safely perform administrative operations like rolling upgrades. There is also the shovel and federation which I still need to write about.

If I missed anything else out then please let me know.

See my post where I use Docker and Blockade to wreak havoc on a RabbitMQ cluster to demonstrate some message loss scenarios, based on the theory we covered in this post: [How to lose messages on a RabbitMQ cluster](https://jack-vanlightly.com/blog/2018/9/10/how-to-lose-messages-on-a-rabbitmq-cluster).

Series links:

*   [Part 1 - Two different takes on messaging (high level design comparison)](https://jack-vanlightly.com/blog/2017/12/4/rabbitmq-vs-kafka-part-1-messaging-topologies)
    
*   [Part 2 - Messaging patterns and topologies with RabbitMQ](https://jack-vanlightly.com/blog/2017/12/5/rabbitmq-vs-kafka-part-2-rabbitmq-messaging-patterns-and-topologies)
    
*   [Part 3 - Messaging patterns and topologies with Kafka](https://jack-vanlightly.com/blog/2017/12/8/rabbitmq-vs-kafka-part-3-kafka-messaging-patterns)
    
*   [Part 4 - Message delivery semantics and guarantees](https://jack-vanlightly.com/blog/2017/12/15/rabbitmq-vs-kafka-part-4-message-delivery-semantics-and-guarantees)
    
*   [Part 5 - Fault tolerance and high availability with RabbitMQ Clustering](https://jack-vanlightly.com/blog/2018/8/31/rabbitmq-vs-kafka-part-5-fault-tolerance-and-high-availability-with-rabbitmq)
    
*   [Part 6 - Fault tolerance and high availability with Kafka](https://jack-vanlightly.com/blog/2018/9/2/rabbitmq-vs-kafka-part-6-fault-tolerance-and-high-availability-with-kafka)