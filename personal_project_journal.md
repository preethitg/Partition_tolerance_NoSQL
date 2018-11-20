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

First I tested whether the nodes are able to talk with each other. So i created a bucket "food" with simple key "favorite" for value "pizza"

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

I changed the security groups of all riak instances in AZ us-west-1c to Riak Personal Project and riak instances in AZ us-west-1a to Riak Personal Project SG2. Thus only instances in the same subnet can talk with each other. I stopped and started my riak nodes again. Upon performing sudo riak-admin cluster status from Riak1 I could see that Riak4 and Riak5 nodes were down. I updated the value for key favorite to "salad" from Riak1.

	curl -v -XPUT -d "salad" \
    http://10.0.1.218:8098/buckets/food/keys/favorite

Upon GET, I got favorite as "salad" from Riak1, Riak2 and Riak3 as they were all in same subnet. Riak4 and Riak5 showed me stale data, it showed "pizza" as favorite. 
Next, to check how nodes choose the latest data, I changed the value for favorite as "pasta" from Riak4. Since it was an AP system , Riak4 and Riak5 were available for write. 

	curl -v -XPUT -d "pasta" \
    http://10.0.3.50:8098/buckets/food/keys/favorite

Upon GET, I got favorite as "pasta" from Riak4 and Riak5 as they were in same subnet and were able to communicate with each other. Riak1, Riak2 and Riak3 still showed "salad". Now I closed the partition by changing both security group rules to:

	Port Range : 8099 ; Source : 0.0.0.0/0, ::/0
	Port Range : 8087 ; Source : 0.0.0.0/0, ::/0
	Port Range : 22 ; Source : 0.0.0.0/0, ::/0
	Port Range : 4369 ; Source : 10.0.3.0/24 , 10.0.1.0/24
	Port Range : 8098 ; Source : 10.0.3.0/24 , 10.0.1.0/24
	Port Range : 6000 - 7999 ; Source : 10.0.3.0/24 , 10.0.1.0/24

After this change, **curl -i http://<ip_address_of_riak_node>:8098/buckets/food/keys/favorite** gave me "pasta" in all the nodes. Thus, during network partition recovery, all the nodes took the latest key for favorite ie, the last write and became consistent in their answers.

## Task 2 - Testing VPC Peering

As Professor told us to use VPC Peering for our Team Project for connecting multiple VPCs together, I researched about VPC Peering and tried using that for Personal Project. VPC Peering is a network connection between multiple VPCs across region which enables to route traffic between them using Private IPv4 addresses. AWS allows VPC peering with VPCs in same account or in another AWS account. 

References

	https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html
	https://docs.aws.amazon.com/vpc/latest/peering/vpc-peering-basics.html
	https://docs.aws.amazon.com/vpc/latest/peering/vpc-peering-routing.html
	https://docs.aws.amazon.com/vpc/latest/peering/vpc-peering-basics.html#vpc-peering-limitations

After going through the references, I started to work on my VPC peering connection. From the VPC Peering Limitation, I understood that AWS blocks VPC Peering with overlapping IPv4 CIDR blocks. So I created two VPCs with IPv4 CIDR blocks 11.0.0.0/16 and 12.0.0.0/16. Iam creating VPCs in same region.

	In AWS Management Console, Click on VPC under Networking & Content Delivery.
	Use Launch VPC Wizard from VPC dashboard to create a new VPC

	VPC #1

	Select VPC with Private and Public Subnets as we need both type of subnets for the project.
	IPv4 CIDR block: 11.0.0.0/16
	VPC name: cmpe281-personal-project-1
	Public subnet's IPv4 CIDR: 11.0.0.0/24
	Availability Zone: No Preference
	Private subnet's IPv4 CIDR: 11.0.1.0/24
	Availability Zone: No Preference

	Click on "Use a NAT instance instead" option to create a NAT instance with free tier limit.
	Instance type: t2.micro
	Key pair name: cmpe281-us-west-1

	Keep the rest as default and click in Create VPC.

	VPC #2
	
	Select VPC with Private and Public Subnets as we need both type of subnets for the project.
	IPv4 CIDR block: 12.0.0.0/16
	VPC name: cmpe281-personal-project-peer
	Public subnet's IPv4 CIDR: 12.0.0.0/24
	Availability Zone: No Preference
	Private subnet's IPv4 CIDR: 12.0.1.0/24
	Availability Zone: No Preference

	Click on "Use a NAT instance instead" option to create a NAT instance with free tier limit.
	Instance type: t2.micro
	Key pair name: cmpe281-us-west-1

	Keep the rest as default and click in Create VPC.

Create VPC Peering

	To create a VPC peering connection, choose Peering Connection from VPC Dashboard.
	Click on Create Peering Connection.

	Peering connection name tag : Personal Project
	VPC (Requester) : cmpe281-personal-project-1
	Account : My Account
	Region : This region (us-west-1)
	VPC (Accepter) : cmpe281-personal-project-peer
	Create Peering Connection.

A VPC peering comes to effect only if both the Accepter VPC accepts the Peering Connection instantiated by Requester VPC. 

	To accept VPC peering, Goto Peering Connection from VPC Dashboard.
	Select the new peering connection and Click on Actions -> Accept Request.

This creates a Peering Connection tag. Only creating a VPC peering connection will not enable traffic between the instances in VPCs. We have to change the routing table of the various subnets involved in the peering connection to accept traffic from other VPC CIDR blocks. From the above VPC creation, we have a Public subnet and a Private subnet in each VPC. The MongoDB cluster which has to created as part of the personal project will reside in the private subnets of these 2 VPCs. Therefore, we have to change the routing tables of all the 4 subnets involved to enable traffic routes from the private subnets in other VPC.

Goto VPC Dashboard and choose Route table option.
Add the following in the routing tables.

<b>cmpe281-personal-project-peer : Public and Private subnets</b>
	
	Destination: 11.0.0.0/16. (CIDR block of VPC 1)
	Target : pcx-********* (Peering Connection tag)

<b>cmpe281-personal-project-1 : Public and Private subnets</b>	

	Destination: 12.0.0.0/16. (CIDR block of VPC 1)
	Target : pcx-********* (Peering Connection tag)

After the routing tables are changed, create a public instance (jumpbox) and a private instance in each VPC.

<b>Configuration of Public instances</b>

	Goto AWS Console. Under Compute click in EC2
	From EC2 dashboard choose Instances
	Launch instance

	AMI : Amazon Linux AMI 2018.03.0 (HVM)
	Type: t2.micro
	Network : cmpe281-personal-project-1 / cmpe281-personal-project-peer
	Subnet : Public subnet
	Auto-assign Public IP : Enable
	Keep everything else as default in that page. Click Next

	Keep default storage
	No tags
	Configure Security Group
	Choose option : Create a Security group
	Security group name : Jumpbox
	Description : SG for Jumbox
	VPC : cmpe281-personal-project-1 / cmpe281-personal-project-peer

	In Inbound Security group rules, Add new rules
	Type : SSH ; Port : 22 ; Source : Anywhere
	Type : HTTP ; Port : 80 ; Source : Anywhere
	Type : HTTPS ; Port : 443 ; Source : Anywhere

Initially, while creating private instances, I opened only SSH port and HTTP port for inbound traffic. But the private instances were not able to talk with each other. I went through some AWS documentation and youtube videos and found out that we have to enable "All ICMP - IPv4" from the other CIDR block of other private instance we want to connect with. Thus the final configuration for private instances are:

<b>Configuration of Private instances</b>

	Goto AWS Console. Under Compute click in EC2
	From EC2 dashboard choose Instances
	Launch instance

	AMI : Amazon Linux AMI 2018.03.0 (HVM)
	Type: t2.micro
	Network : cmpe281-personal-project-1 / cmpe281-personal-project-peer
	Subnet : Private subnet
	Auto-assign Public IP : Disable

	Keep default storage
	No tags
	Configure Security Group
	Choose option : Create a Security group
	Security group name : Private-SG
	Description : SG for private instances
	VPC : cmpe281-personal-project-1 / cmpe281-personal-project-peer

	For security group in cmpe281-personal-project-1,
	In Inbound Security group rules, Add new rules
	Type : SSH ; Port : 22 ; Source : 11.0.0.0/16 , 12.0.0.0/16
	Type : All ICMP - IPv4 ; Source : 12.0.1.0/24

	For security group in cmpe281-personal-project-peer,
	In Inbound Security group rules, Add new rules
	Type : SSH ; Port : 22 ; Source : 11.0.0.0/16 , 12.0.0.0/16
	Type : All ICMP - IPv4 ; Source : 11.0.1.0/24	

After creating instances, ssh into both the jumpbox public instances

	ssh -i cmpe281-us-west-1.pem ec2-user@54.153.127.96
	ssh -i cmpe281-us-west-1.pem ec2-user@52.53.125.40

In order to connect to private instances, move the key-pair to jumpbox

	scp -i cmpe281-us-west-1.pem cmpe281-us-west-1.pem ec2-user@<public IP>:/tmp
	mv /tmp/cmpe281-us-west-1.pem .

Connect to respective private instances

	ssh -i cmpe281-us-west-1.pem ec2-user@11.0.1.156
	ssh -i cmpe281-us-west-1.pem ec2-user@12.0.1.160

From one private instance, ping the other private instance to check if the private instances from both VPCs are able to talk with each other

	ping 11.0.1.156
	ping 12.0.1.160

<h2>Summary</h2>

	Setup Riak cluster and performed network partitioning to test the Partition Tolerance of AP system
	Setup VPC peering connection to use the same to test the Partition Tolerance of CP system next week.
