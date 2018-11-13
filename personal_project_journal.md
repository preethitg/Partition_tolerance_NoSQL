# Week 1

CAP Theorem

There are three key concepts that has to be balanced while choosing a data management system 
1.Consistency : Every node sees the same data at the same time ie, read from nodes will return the most recent write. 
2.Availability : Availability means that the system should be operational 100% of the time. Every request gets a response  from the node, recent or stale.
3.Partition Tolerance : Partiton Tolerance means that the system should function despite network failure. 

CAP theorem states that distributed data system can only have 2 out of 3: Consistency, Availability, Partition Tolerance[1]. Network partition is not an option, it is a situation that has to be faced. Thus, under network partition, we have to choose between Consistency and Availability based on our usecase.

CP - Under network failure, CP systems try to make the system consistent rather than available. Every transaction should go from one consistent state to another consistent state. It waits for a response from the partitioned node and under timeout, it either makes it unavailable, or throw an error. Once the network partition is resolved, it makes the system available with the most recent data all over the nodes.
Example - MongoDB
	MongoDB chooses to be consistent under network partition. It is an open source document oriented NoSQL system.

AP - Under network partition, AP systems make the system available and give either recent or stale data. It also accepts write during this phase. After network partition recovery, the system resolves all the inconsistent data becomes eventually consistent and gives most recent data in all the nodes.
Example - Riak
	Riak is a NoSQL system with key-value data model. It offers fault tolerant availability. It offers HTTP operations such as GET, PUT, POST, DELETE data manipulation. 

<b>Understand concepts of Riak DB, keywords such as Buckets, Ring, R, W, n_val etc</b>
https://github.com/basho-labs/little_riak_book/blob/master/en/02-concepts.md

<b>To get overview of HTTP operations that can be performed for Riak</b>
http://docs.basho.com/riak/kv/2.2.3/developing/api/http/

<b>MongodB documentation - Install Mongo on Amazon</b>
https://docs.mongodb.com/manual/tutorial/install-mongodb-on-amazon/

<b>Deploy a Highly-Available MongoDB Replica Set on AWS</b>
https://eladnava.com/deploy-a-highly-available-mongodb-replica-set-on-aws/

<h3>Other References:</h3>
<pre>
[1]https://towardsdatascience.com/cap-theorem-and-distributed-database-management-systems-5c2be977950e
[2]http://blog.nahurst.com/visual-guide-to-nosql-systems
[3]http://robertgreiner.com/2014/08/cap-theorem-revisited/
[4]https://queue.acm.org/detail.cfm?id=1394128
[5]https://echidna.co/blog/scalable-commerce-nosql-databases/
</pre>

<h2>Summary</h2>
<pre>
	Got an understanding of CAP theorem
	Got an understanding of MongoDB and its CP properties under network partition
	Got an understanding of Riak and its AP properties under network partition
</pre>


