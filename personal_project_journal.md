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

# Week 2

## Task 1 - Setup Riak cluster and use dummy value to test for partition

In order to create a network partition on a later stage, I created a private subnet in my "cmpe281" VPC. The private subnet the got automatically generated with VPC creation was for Availability Zone (AZ) us-west-1c. Thus I created my new private subnet in AZ us-west-1a. 
	
	Goto AWS Console. Under Networking & Content Delivery, click on VPC
	From VPC dashboard choose Subnets
	Create Subnet

	Name tag: Private subnet_az_a
	VPC : cmpe281
	Availability Zone: us-west-1a
	IPv4 CIDR block : 10.0.3.0/24

	Create

After this step, I had 2 private subnets
	
	Private subnet : 10.0.1.0/24 : us-west-1c
	Private subnet_az_a : 10.0.3.0/24 : us-west-1a

Next I went through Riak Documentation to understand the ports that has to be opened for inter-node communication within cluster

	http://docs.basho.com/riak/kv/2.2.3/setup/installing/amazon-web-services/

According to the documentation, I created a Riak Security Group in AWS

	Goto AWS Console. Under Compute click on EC2
	From EC2 dashboard choose Security Groups under NETWORK & SECURITY
	Create Security Group

	Security group name : Riak Personal Project
	Description : Riak Personal Project
	VPC : cmpe281

	In Inbound Security group rules, Add new rules

	Port Range : 8099 ; Source : 0.0.0.0/0, ::/0
	Port Range : 8087 ; Source : 0.0.0.0/0, ::/0
	Port Range : 22 ; Source : 0.0.0.0/0, ::/0
	Port Range : 4369 ; Source : <Your security group ID>
	Port Range : 8098 ; Source : <Your security group ID>
	Port Range : 6000 - 7999 ; Source : <Your security group ID>

	Create

Now with this rule, any node in the same security group can communicate with each other.
Next create 5 Riak instances. I created 3 in AZ us-west-1c and 2 in AZ us-west-1a

	Goto AWS Console. Under Compute click in EC2
	From EC2 dashboard choose Instances
	Launch instance

	AMI : Riak KV 2.2 Series
	Type: t2.micro
	Network : vpc - cmpe281
	Subnet : Choose private subnet in us-west-1c or us-west-1a
	Auto-assign Public IP : Disable
	Keep everything else as default in that page. Click Next

	Keep default storage
	No tags
	Configure Security Group
	Choose option : Select an existing security group
	Select Riak Personal Project
	Review and launch

Since all the instances are in private subnet, i created a jumpbox
	
	Goto AWS Console. Under Compute click in EC2
	From EC2 dashboard choose Instances
	Launch instance

	AMI : Amazon Linux AMI 2018.03.0 (HVM)
	Type: t2.micro
	Network : vpc - cmpe281
	Subnet : Public subnet
	Auto-assign Public IP : Enable
	Keep everything else as default in that page. Click Next

	Keep default storage
	No tags
	Configure Security Group
	Choose option : Select an existing security group
	Select cmpe281-dmz (Open Inbound ports 80,22,442)
	Review and launch

ssh into Jumpbox from terminal

	ssh -i <key> ec2-user@<ip address of Jumpbox>
	ssh -i cmpe281-us-west-1.pem ec2-user@13.57.254.158

Copy key file (.pem) into Jumpbox

	scp -i <key> <key> ec2-user@<ip address of Jumpbox> :/tmp
	mv /tmp/<key> . 

From Jumpbox, ssh into all the 5 private riak instances
	
	Riak1 - ssh -i cmpe281-us-west-1.pem ec2-user@10.0.1.218
	Riak2 - ssh -i cmpe281-us-west-1.pem ec2-user@10.0.1.53
	Riak3 - ssh -i cmpe281-us-west-1.pem ec2-user@10.0.1.90
	Riak4 - ssh -i cmpe281-us-west-1.pem ec2-user@10.0.3.50
	Riak5 - ssh -i cmpe281-us-west-1.pem ec2-user@10.0.3.237

Update and Start riak in all the nodes

	sudo yum update
	sudo riak start

Use one node to join the cluster

	sudo riak-admin cluster join riak@10.0.1.218

The nodes were unreachable. Then I searched through Riak documentation and found that we have to change riak.conf and app.config file for successful inter-node communication.

	http://docs.basho.com/riak/kv/2.2.3/using/security/


Change riak.conf

	sudo find / -name "riak.conf"
	Add the following in riak.conf
		erlang.distribution.port_range.minimum = 6000
		erlang.distribution.port_range.maximum = 7999

Change app.config

	sudo find / -name "app.config"
	Add the follwing in the beginning of app.config file
		{ kernel, 
		 [
		  {inet_dist_listen_min, 6000},
		  {inet_dist_listen_max, 7999}
		]}

These steps will become effective once you stop riak and start again. 
Perform **sudo riak-admin cluster join riak@10.0.1.218** from all the instances. After this step, the node will be in joining state. Status of the cluster or member status can be inspected using following instructions

	sudo riak-admin cluster status
	sudo riak-admin member_status

From the node chosen to join the cluster, perform the following to plan and commit the cluster

	sudo riak-admin cluster plan
	sudo riak-admin cluster commit

This will take few moments to make all the nodes in the cluster valid. To remove a node from a cluster, perform the follwing from the node that has to be removed
	
	sudo riak-admin cluster leave riak@<ip address of the node which was choosen to join>

First I tested whether the nodes are able to talk with each other. So i created a bucket "food" with simple key "pizza" for value "favorite"

	curl -v -XPUT -d "pizza" \
    http://10.0.1.218:8098/buckets/food/keys/favorite

The curl can be tested with the following curl command
	
	curl -i http://<ip address of riak node>:8098/buckets/food/keys/favorite

The test was successfull and I got "pizza" from all the nodes. 
Next task was to create a test network partition. The inter-node communication error I got in the past gave me the idea to change the security rule such the nodes in one private subnet cannot talk with nodes in another subnet

	Goto AWS Console. Under Compute click on EC2
	From EC2 dashboard choose Security Groups under NETWORK & SECURITY
	Change Security Group : Riak Personal Project

	Port Range : 8099 ; Source : 0.0.0.0/0, ::/0
	Port Range : 8087 ; Source : 0.0.0.0/0, ::/0
	Port Range : 22 ; Source : 0.0.0.0/0, ::/0
	Port Range : 4369 ; Source : 10.0.1.0/24 (IPv4 CIDR block for Private subnet in AZ us-west-1c)
	Port Range : 8098 ; Source : 10.0.1.0/24 (IPv4 CIDR block for Private subnet in AZ us-west-1c)
	Port Range : 6000 - 7999 ; Source : 10.0.1.0/24 (IPv4 CIDR block for Private subnet in AZ us-west-1c)

	Create a new Security group

	Goto AWS Console. Under Compute click on EC2
	From EC2 dashboard choose Security Groups under NETWORK & SECURITY
	Create Security Group

	Security group name : Riak Personal Project SG2
	Description : Riak Personal Project SG2
	VPC : cmpe281

	In Inbound Security group rules, Add new rules

	Port Range : 8099 ; Source : 0.0.0.0/0, ::/0
	Port Range : 8087 ; Source : 0.0.0.0/0, ::/0
	Port Range : 22 ; Source : 0.0.0.0/0, ::/0
	Port Range : 4369 ; Source : 10.0.3.0/24 (IPv4 CIDR block for Private subnet in AZ us-west-1a)
	Port Range : 8098 ; Source : 10.0.3.0/24 (IPv4 CIDR block for Private subnet in AZ us-west-1a)
	Port Range : 6000 - 7999 ; Source : 10.0.3.0/24 (IPv4 CIDR block for Private subnet in AZ us-west-1a)

	Create

I changed the security groups of all riak instances in AZ us-west-1c to Riak Personal Project and riak instances in AZ us-west-1a to Riak Personal Project SG2. Thus only instances in the same subnet can talk with each other. I stopped and started my riak nodes again. Upon performing sudo riak-admin cluster status from Riak1 I could see that Riak4 and Riak5 nodes were down. I updated the key for value favorite to "salad" from Riak1.

	curl -v -XPUT -d "salad" \
    http://10.0.1.218:8098/buckets/food/keys/favorite

Upon GET, I got favorite as "salad" from Riak1, Riak2 and Riak3 as they were all in same subnet. Riak4 and Riak5 showed me stale data, it showed "pizza" as favorite. Thus the nodes were Available with inconsistent data.
Next, to check how nodes choose the latest data, I changed the key for favorite as "pasta" from Riak4.

	curl -v -XPUT -d "pasta" \
    http://10.0.3.50:8098/buckets/food/keys/favorite

Upon GET, I got favorite as "pasta" from Riak4 and Riak5 as they were in same subnet and were able to communicate with each other. Riak1, Riak2 and Riak3 still showed "salad". Now I closed the partition by changing both security group rules to:

	Port Range : 8099 ; Source : 0.0.0.0/0, ::/0
	Port Range : 8087 ; Source : 0.0.0.0/0, ::/0
	Port Range : 22 ; Source : 0.0.0.0/0, ::/0
	Port Range : 4369 ; Source : 10.0.3.0/24 , 10.0.1.0/24
	Port Range : 8098 ; Source : 10.0.3.0/24 , 10.0.1.0/24
	Port Range : 6000 - 7999 ; Source : 10.0.3.0/24 , 10.0.1.0/24

After this change, **curl -i http://<ip_address_of_riak_node>:8098/buckets/food/keys/favorite** gave me "pasta" in all the nodes. Thus, during network recovery, all the nodes took the latest key for favorite and became consistent in their answers.
