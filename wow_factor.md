## Amazon Elastic Beanstalk

Elastic beanstalk helps us to deploy, monitor and scale an application easily in very less time. For testing the uses of Elastic beasnstalk, i created a new application.

	Goto Elastic Beanstalk from AWS console
	Click on "Create a new application"
	Give the application name and description.

After this step, a new application folder gets created with the name of the application

Next, we have to choose the environment for application deployment. 

Elastic beanstalk provides us with 2 type of environment for different types of web applications. Web servers are normal application that uses HTTP requests for processing. Workers are specialized applications that listents for messages from an Amazon SQS queue and post the same to our application.

I selected normal web server as my environment. Once you click on create, you will be asked to choose the url name and type of platform you want, to host the application. Elastic beanstalk is platform as a service. Ive used multi container docker as my platform. I will be using the sample application provided by elastic beanstalk for deployment. 

Next click on create. Elastic beanstalk starts running lot of background processes which includes starting an instance, creating security group, and an Amazon S3 bucket for data storage for the application.

The environment will take 5-10 mins to startup. Once the environment dashboard is created, we can see the health status and running version of the app. Based on the type of app configuration, elastic beanstalk has itself created an Amazon Linux/2.11.5 system. 

Configuration board gives us the configuration overview of the application. In the Instance section , we can see that Elastic beanstalk has itself created an EC2 t2.micro instance. We have full access to change the type of instance here. 
Next lets check the capacity. By default, elastic beanstalk creates a single instance with min and max capacity as 1. 

I have changed this to load balancer to increase the number if instances based on my desire. I gave the min capacity as 1 and max capacity as 4.

Coming back to the configuration now, we can see that a classic load balancer has been automatically created. If we check on the events, we can see that an auto scaling group has also been created. We can do many things from elastci beanstalk : send notification to developer, create custom health checks, integrate a backed DB with the application etc.

Now that ive created an instance, i checked the running instance in my EC2. I could see that a new instance is running.
A new auto scaling group, security group and load balancer has been created.

Next, to check if auto scaling works properly, I stopped the running ec2 instance. Once the instance stopped completely, a new instance got automatically created and terminated the stopped instance.

Thus, we can see that deploying an application is very easy in Elastic beanstalk, we dont have to worry about creating backup, instances, load balancers etc. Elastic beankstalk talkes care of everything.





