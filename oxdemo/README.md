# DX Docker Con Demo

## Demo setup

Make sure these things are done before you run this demo for the very first time. 

3. Clone `https://github.com/mikegcoleman/dcus-booth-demo-2017.git` onto the demo machine in the current users home directory

		$ git clone https://github.com/mikegcoleman/dcus-booth-demo-2017.git 
             
5. Navigate in a web browser to [https://dockersupportlab.signin.aws.amazon.com/console/](https://dockersupportlab.signin.aws.amazon.com/console/)

		Account: dockersupportlab
		User Name: dockerdemo
		Password: <see sticker>

	> **Note** This is a live Docker-owned AWS account. DO NOT allow attendees to access it. DO NOT access any resources that are not necessary for this demo, you could be wrecking someone else's day doing so!
	
6. Open the following web pages in their own tab
	
	[Cloud Formation](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stack/detail?stackId=arn:aws:cloudformation:us-east-1:698275814562:stack%2FDockerCon-Swarm%2F17e2ec00-2213-11e7-a89e-50d5cd1ea8d2)
	
	[Log Group](https://console.aws.amazon.com/cloudwatch/home?region=us-east-1#logStream:group=DockerCon-Swarm-lg)
	
	[Auto Scaling Groups](https://console.aws.amazon.com/ec2/autoscaling/home?region=us-east-1#AutoScalingGroups:view=details)
	
	[Docker EE from Amazon Marketplace](https://aws.amazon.com/marketplace/fulfillment?fulfillmentOptionId=abdd18e9-f511-452d-a9cb-c9ffe72383df&productId=542db327-b437-4851-8f65-f15d5c80b4ef&ref_=dtl_psb_continue&region=us-east-1&showManualLaunchTab=CLOUD_FORMATION)
	



# Running the Demo

## Creating a New EE Swarm Cluster

We will start by showing how you can use Cloud Formation to create a new Swarm cluster. Since this takes a bit of time, we won't actually create it, but you'll get a sense of how the process works. 

1. Navigate in your web browser to the EE listing on the Amazon Marketplace ([Docker EE from Amazon Marketplace](https://aws.amazon.com/marketplace/fulfillment?fulfillmentOptionId=abdd18e9-f511-452d-a9cb-c9ffe72383df&productId=542db327-b437-4851-8f65-f15d5c80b4ef&ref_=dtl_psb_continue&region=us-east-1&showManualLaunchTab=CLOUD_FORMATION))

2. Review the various options on the page (it doesn't matter what you select as you are not deploying a cluster due to time)

3. Click `Launch with CloudFormation Console`

4. Point out the CFN template is prepopluated and click `Next`

5. Point out the various parameters and click `Next`

6. Click `Next`

7. Point out the Summary and the Create button - and then click `Cancel`

## Looking at an existing Cluster

After you run through the cloudformation template AWS deploys a swarm cluster built on an AWS infrastrucgture that inclues the VPC, subnets, load balancer, and network security group. 

Let's take a look at how an already deployed cluster. 

1. Navigate to the `Dockercon-Swarm` in cloudformation. [Cloud Formation](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stack/detail?stackId=arn:aws:cloudformation:us-east-1:698275814562:stack%2FDockerCon-Swarm%2F17e2ec00-2213-11e7-a89e-50d5cd1ea8d2)

2. The `Events` list is the log output for all the resources created during the swarm provisioning proces. You can actually watch these events in real time while a cluster is provisioned. 

3. The `Resources` group shows all the actual resources that were created. You can see the instances, load balancers, vpc's, subtnets, etc. here. 

4. The `Outputs` group shows key items that you'll need to know to manage and use your swarm.  Two the items are of particular interest. 

	The `DefaultDNSTarget` is the URL of a load balancer that sits in front of your deployed apps. When you deploy an app on the  Swarm ou will access it via this URL (and usually a port number). Ports are dynamically allocated as apps are created. 

	`Managers` will bring up the autoscaling group for the swarm manager nodes.  From that view you can find the public IP you can use to SSH into those nodes. Both manager and worker nodes are deployed as part of an autoscaling group - if you need to increase or decrease the number of nodes you can do so via the autoscaling group - we'll look at this in a bit more detail. 
	
## Autoscaling Groups

1. Navigate to he EC2 Autoscaling Groups page:  [Auto Scaling Groups](https://console.aws.amazon.com/ec2/autoscaling/home?region=us-east-1#AutoScalingGroups:view=details)

	There are a few ASG's listed here, but the two we care about are `DockerCon-Swarm-ManagerAsg-1U1CH2YQV4R7Y` (the ASG for the manager nodes) and `DockerCon-Swarm-NodeAsg-1QM0RAETRB2E9` (the ASG for the workernodes)
	
2. Click the check box next to the Manager nodes

	> **Note** makre sure only one check box is checked
	
	At the bottom of the web page, on the left side ou can see we have 1 desired node. 
	
3. Click the `Instances` tab, and you can see the actual EC2 instance that is serving as our Manager node

4. Uncheck the box next to the manager asg, and click the one next to the node ASG. 

	Look at the number of desired nodes and the actual instances. 
	
5. Move back to the `Details` tab, and click `Edit` on the top righ side of hte lower part of the web page. 

6. Here you an add (or remove) more nodes simply by adjusting the value for the `Desired` field. 

	When you do this nodes will be provisioned or deprovisioned, added (or removed) from the Swarm, and the load balancer adjusted appropriately. 
	
Now that we've looked at how to create and scale the Swarm, let's take a look at the actual swarm itself. We'll start by SSHing into the manager node. 

1. `$ cd ~/dcus-booth-demo-2017/oxdemo`

2. `ssh -i swarm.pem docker@52.91.232.31`

3. Let's take a look at the nodes in the swarm

		~ $ docker node ls
		ID                           HOSTNAME                       STATUS  AVAILABILITY  MANAGER STATUS
		1bgcerq0qfnmlvt67l20aq0tc    ip-172-31-2-220.ec2.internal   Ready   Active
		dc1cq4oa0nts6mhy7r16si38u    ip-172-31-37-194.ec2.internal  Ready   Active
		yo9quc3pea4i9fi333gpwehcq    ip-172-31-29-104.ec2.internal  Ready   Active
		yva8ulea9zqhk9nnhycnzmbrz *  ip-172-31-17-132.ec2.internal  Ready   Active        Leader

4. From here you can deploy services or stacks just like you would normally. In fact there should be a couple services running

		~ $ docker service ls
		ID            NAME    MODE        REPLICAS  IMAGE
		f4euparg11vi  web     replicated  1/1       nginx:latest
		l5wq8wiktxsl  catweb  replicated  1/1       mikegcoleman/catweb:latest
		
Once you have services running, you may need to take at their logs from time to time. This is another point of integration with Docker EE for AWS. Docker Logs can be aggregated into a Log Group under Cloudwatch. 

1. Navigate to the cloudformation log group: [Log Group](https://console.aws.amazon.com/cloudwatch/home?region=us-east-1#logStream:group=DockerCon-Swarm-lg)

2. Click on an of the listed logs. These logs represent the various services that have been deployed onto the swarm. 


	
	