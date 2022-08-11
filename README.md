# AutoScaling
Maintaining High Availability with Auto Scaling

## Overview

###### Auto Scaling allows you to scale your Amazon EC2 capacity up or down automatically according to conditions you define. With Auto Scaling, you can ensure that the number of Amazon EC2 instances you’re using increases seamlessly during demand spikes to maintain performance and decreases automatically during demand lulls to minimize costs. Auto Scaling is particularly well suited for applications that experience hourly, daily, or weekly variability in usage.

###### But Auto Scaling represents more than a way to add and subtract servers. It is also a mechanism to handle failures similar to the way load balancing handles unresponsive servers. This lab will demonstrate configuring Auto Scaling to automatically launch, monitor, and update the load balancer associated with your Elastic Compute Cloud (EC2) instances.

###### There are two important things to know about Auto Scaling. First, Auto Scaling is a way to set the “cloud temperature.” You use policies to “set the thermostat,” and under the hood, Auto Scaling controls the heat by adding and subtracting Amazon EC2 resources on an as-needed basis in order to maintain the “temperature” (capacity).

###### An Auto Scaling policy consists of:

* A launch configuration that defines the servers that are created in response to increased demand.

* An Auto Scaling group that defines when to use a launch configuration to create new server instances and in which Availability Zone and load balancer context they should be created.

###### Second, Auto Scaling assumes a set of homogeneous servers. That is, Auto Scaling does not know that Server A is a 64-bit extra-large instance and more capable than a 32-bit small instance. In fact, this is a core tenet of cloud computing: scale horizontally using a fleet of fungible resources; individual resources are secondary to the fleet itself.

## Project Pre-requisites

###### To successfully complete this lab, you should be familiar with basic Linux server administration and comfortable using the Linux command-line tools. You should also be proficient at this point with the basics of creating new Amazon EC2 server instances and configuring Elastic Load Balancing.

**Other AWS Services**

###### Other AWS Services than the ones needed for this lab are disabled by IAM policy during your access time in this lab. In addition, the capabilities of the services used in this lab are limited to what’s required by the lab and in some cases are even further limited as an intentional aspect of the lab design. Expect errors when accessing other services or performing actions beyond those provided in this lab guide.

**Key components of Auto Scaling**

###### When you launch a server manually, you provide parameters such as which Amazon Machine Image (AMI), which instance type, and which security group to launch in. Auto scaling calls this a launch configuration. It is simply a set of parameters describing what kind of instances to launch.

###### Auto Scaling groups tell the system what to do with an instance after it is launched. This is where you specify which Availability Zones your instances should be launched in, which load balancers they will receive traffic from, and—most importantly—the minimum and maximum number of instances to run at any given time.

###### You need rules that tell the system when to add or subtract instances. These are known as scaling policies, and have rules such as “scale the fleet out by 10%” and “scale in by 1 instance.”

**Timing matters**

###### There are costs related to using Auto Scaling. There are two important factors that directly affect the cost of AWS and also the manner in which your application scales: cost and time.

###### Amazon EC2 Linux instances are charged per-second

###### This means that you can scale-out the servers when there is a lot of activity, then scale-in to reduce costs when less capacity is required.

**Scaling takes time**

###### Consider the following graph. In most situations, a considerable amount of time passes between when the need for a scaling event occurs and when the scaling event happens.
