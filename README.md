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

<img width="661" alt="Screen Shot 2022-08-10 at 8 54 48 PM" src="https://user-images.githubusercontent.com/67527927/184047371-f1eaa489-6e08-4332-8afb-6b2cf03a32a1.png">

* In this example, the rule says that you must be in a particular condition for at least two minutes.
* CloudWatch is the underlying data collection system that monitors statistics such as CPU utilization. It is a polling protocol, and in general takes 60 seconds to aggregate new data.
* Auto Scaling is also a polling system, and it takes another 60 seconds.
* Then there is boot time for your server. A large, complex server may take many minutes to launch.
* Finally, the load balancer needs to poll the server for a few cycles before it is comfortable that the server is healthy and accepting requests.

### Connect to your EC2 Instance
<img width="197" alt="Screen Shot 2022-08-10 at 8 58 21 PM" src="https://user-images.githubusercontent.com/67527927/184047650-f8be961c-23f4-46cd-b4e3-35be5ed4c25d.png">
###### These instructions are for Mac/Linux users only. If you are a Windows user, skip ahead to the next task.

###### To the left of the instructions you are currently reading, click  Download PEM.

###### Save the file to the directory of your choice.

###### Copy this command to a text editor:
```
chmod 400 KEYPAIR.pem

ssh -i KEYPAIR.pem ec2-user@Instance
```

###### Replace KEYPAIR.pem with the path to the PEM file you downloaded.

###### Replace Instance with the values of Instance shown to the left of these instructions.

###### Paste the updated command into the Terminal window and run it.

###### Type yes when prompted to allow a first connection to this remote SSH server.

###### Because you are using a key pair for authentication, you will not be prompted for a password.

### Configure AWS CLI
###### In this section, you’ll configure the AWS Command Line Interface (CLI) tool to execute the commands.

###### Enter the following command in your SSH session:
```
aws configure
```
* AWS Access Key ID: Press Enter
* AWS Secret Access Key: Press Enter
* Default region name: Enter the value of REGION located to the left of these instructions
Note: If you have a letter at the end of your region code (e.g. us-west-2c) subsequent commands will fail. Please ensure the region code ends in a number.
* Default output format [None]: Press Enter

###### You omitted the Access Key and Secret Key values as the credentials are being passed to the instance using an IAM role created during setup. See http://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html for more information about IAM roles and policies.

###### The default output format if not mentioned is json.

### Create a Launch Configuration

###### This launch configuration specifies a machine image (Amazon Machine Image or AMI) that will launch when Auto Scaling adds new servers. You will use the AMI that was used to create your current running instance.

###### Copy the following command into your text editor.
```
aws autoscaling create-launch-configuration --image-id AMIID --instance-type t3.micro --key-name KEYNAME --security-groups EC2SECURITYGROUPID --user-data file:///home/ec2-user/as-bootstrap.sh --launch-configuration-name lab-lc
```
###### Replace the variables in the command that you copied with the values of the variables located to the left of these instructions.

###### Copy the final command from your text editor and paste it into your SSH session.

###### Press Enter.

### Create an Auto Scaling Group
###### In this section, you will create a new Auto Scaling group in your current region and Availability Zone. The group will ensure that there is always one server running by establishing a minimum Auto Scaling group size of one. You will also specify that the maximum number of servers in this group must not exceed four.

###### Copy the following command into your text editor.
```
aws autoscaling create-auto-scaling-group --auto-scaling-group-name lab-as-group --launch-configuration-name lab-lc --load-balancer-names LOADBALANCER --max-size 4 --min-size 1 --vpc-zone-identifier SUBNET1,SUBNET2
```
###### Replace the variables in the command that you copied with the values of the variables located to the left of these instructions.

###### Copy the edited command from your text editor and paste it into your SSH session.

###### Press Enter.

### Verifying Auto Scaling
###### In this section of the lab, you verify your lab environment and test your Auto Scaling group.

### Verify that the Auto Scaling servers were launched
###### Use the AWS Management Console to inspect your instance count. You should see two instances in your fleet because you set the minimum size to one (you may need wait a few minutes before it appears), and you had one instance already running.

###### In the AWS Management Console, on the Services menu, click EC2.

###### In the navigation pane, click Instances.

###### Select the instance without a name (not the instance named “Command-Line Tools”).

###### Verify that the Status Checks column has changed from Initializing to 2/2 checks. You can click the Refresh icon to refresh the status.

###### When the status is 2/2 checks, click the Details tab in the lower pane.

###### Copy the Public DNS (IPv4) name of your new instance to your Clipboard. It will be similar to ec2-54-324-142-12.us-west-2.compute.amazonaws.com. (do not click open address)

###### Paste the Public DNS value into the address bar of a new browser window to verify that the instance is running.

###### You should see a page that looks like this:

<img width="168" alt="Screen Shot 2022-08-10 at 9 11 23 PM" src="https://user-images.githubusercontent.com/67527927/184048555-6274e1f9-bf0d-4c35-8b87-e1748f5f1783.png">

### VERIFY THAT AUTO SCALING WORKS

###### Right-click the instance without a name, click Terminate instance (do not terminate the Command-Line Tools instance).
###### Click Terminate

###### In a few minutes a new t3.micro instance will appear because Auto Scaling will detect that the fleet size is below the minimum size.

###### Refresh the console to see the new instance.
###### Note It may take a few minutes for the new instance to appear.

###### After the Instance State of the new instance is running, right-click the new instance, click Stop Instance
###### Click Stop
###### Auto Scaling will detect that the instance is non-responsive after a few minutes and will automatically terminate it and launch a replacement instance for you.

### TAG AUTO SCALING RESOURCES
###### Notice that the Auto Scaling instances are launched without names. There are two ways to help you better identify these instances. The first is by adding a new column to the Management Console.

###### Click the Gear icon in the navigation bar of the AWS Management Console to display the Preferences dialog box.

###### If it is not already selected, click the aws:autoscaling:groupName option in the Tag columns.

###### Click Confirm.

###### In the AWS Management Console, click the Refresh button.

###### Auto Scaling automatically creates and populates a tag called aws:autoscaling:groupName for your Auto Scaling instances.

###### The second way you can better identify your Auto Scaling instances is to modify your Auto Scaling group to populate the Name tag for you. You could have created a Name tag for the Auto Scaling group when you created it by using the --tags “Key=Name, Value=AS-Web-Server” option. Instead, because the Auto Scaling group already exists, modify the existing tags.

###### Copy the following command into your SSH session, then press Enter.
```
aws autoscaling create-or-update-tags --tags "ResourceId=lab-as-group, ResourceType=auto-scaling-group, Key=Name, Value=AS-Web-Server, PropagateAtLaunch=true"
```
###### If the command executed successfully, you’ll be returned to the shell prompt.

###### Now you will verify that the configuration was updated.

###### Right-click the running instance without a name, click Stop Instance and then click Stop.
###### In a few minutes, a new instance will be created.

###### Verify that the new instance is named AS-Web-Server.
###### Note You may need to refresh the AWS Management Console.

### AUTO SCALING INTEGRATION WITH YOUR LOAD BALANCER
###### Auto Scaling instances are being added to your load balancer. This was configured with the –load-balancer-names NameOfYourELB option when the Auto Scaling group was created. Now confirm that the instances have been added.

###### Return to the EC2 Dashboard.
###### In the navigation pane, click Load Balancers.
###### Select the only load balancer by clicking on it.
###### Click the Instances tab in the lower pane.
###### Your instance should be listed by name.

###### Create Auto Scaling Notifications
###### All of these Auto Scaling activities are occurring transparently. You can configure Auto Scaling to notify you when it automatically creates or terminates instances. Auto Scaling has been integrated with Amazon Simple Notification Service (SNS) for precisely this purpose. SNS is a web service that makes it easy to set up, operate, and send notifications from the cloud. It provides developers with a highly scalable, flexible, and cost-effective ability to publish messages from an application and immediately deliver them to subscribers or other applications.

###### CREATE AN AMAZON SNS TOPIC
###### First, create an Amazon SNS topic used to send notifications.

###### In the AWS Management Console, on the Services menu, click Simple Notification Service.

###### If prompted, click Get Started.

###### In the left navigation pane, click Topics.

###### You may need to click the  icon first, then click Topics.

###### Click Create topic.

###### In the Name box, type 

###### lab-as-topic

###### In the Display name box, type a name (e.g., lab-as).

###### Click Standard

###### Click Create topic.

###### On your topic page, click Create subscription, then configure:
* Protocol: Email
* Endpoint: Enter the email address used to receive email notifications
* Click Create subscription

###### Note You will need to validate your SNS subscription request, so the email address must be a valid one that you can access.

###### Check your email and click the appropriate link to confirm your subscription to the topic.
###### Now that SNS is set up, you need the Amazon Resource Number (ARN) for the SNS topic to use with Auto Scaling.

###### In the AWS Management Console, return to the SNS configuration page.

###### In the left navigation pane, click Topics.

###### Copy the ARN for your topic lab-as-topic to your text editor.

###### Your ARN should look similar to: arn:aws:sns:us-west-2:574764847664:lab-as-topic

### CREATE AUTO SCALING NOTIFICATIONS

###### You can use the aws autoscaling describe-auto-scaling-notification-types command on your server’s Linux command line to determine the types of Auto Scaling notifications that are supported. For example:

```
aws autoscaling describe-auto-scaling-notification-types

“AutoScalingNotificationTypes”: [

“autoscaling:EC2_INSTANCE_LAUNCH”,

“autoscaling:EC2_INSTANCE_LAUNCH_ERROR”,

“autoscaling:EC2_INSTANCE_TERMINATE”,

“autoscaling:EC2_INSTANCE_TERMINATE_ERROR”,

“autoscaling:TEST_NOTIFICATION”

]
```
###### Copy the following command to your text file:
```
aws autoscaling put-notification-configuration --auto-scaling-group-name lab-as-group --topic-arn SNSARN --notification-types autoscaling:EC2_INSTANCE_LAUNCH autoscaling:EC2_INSTANCE_TERMINATE
```
###### Modify the script and replace SNSARN with your ARN.

###### Copy the edited command from your text editor into your SSH session, then press Enter.

###### If the command executed successfully, you’ll be returned to the shell prompt.

###### Check your email to verify that you received a test notification email confirming the configuration.

### Create Auto Scaling Policies
###### Currently you have an Auto Scaling group that will verify that you have at least one running server. In this section, you configure the Auto Scaling group to automatically scale up whenever the average CPU of the web server fleet is ≥ 40%.

###### You will use the aws autoscaling put-scaling-policy command to create two scaling policies that increase the number of servers by one for scale-up events and decrease the number of servers by one for scale-down events. In this exercise, you will also specify a “cooldown” period of 300 seconds to let the environment settle before initiating additional scale-up/down events.

### CREATE A SCALE-UP POLICY
###### Copy the following command into your SSH session, then press Enter.
###### This will create a scale-up policy.
```
aws autoscaling put-scaling-policy --policy-name lab-scale-up-policy --auto-scaling-group-name lab-as-group --scaling-adjustment 1 --adjustment-type ChangeInCapacity --cooldown 300 --query 'PolicyARN' --output text
```
###### An ARN should be returned similar to this: arn:aws:autoscaling:us-west-2:327756672493:scalingPolicy:875526fd-5e5d-4fa6-a23d-ba9818d5d397:autoScalingGroupName/lab-as-group:policyName/lab-scale-up-policy

### CREATE A SCALE-DOWN POLICY
###### Copy the following command into your SSH session, then press Enter.
###### This will create a scale-down policy.
```
aws autoscaling put-scaling-policy --policy-name lab-scale-down-policy --auto-scaling-group-name lab-as-group --scaling-adjustment -1 --adjustment-type ChangeInCapacity --cooldown 300 --query 'PolicyARN' --output text
```
###### Notice the policy name change as well as the slight change enclosing the –scaling-adjustment -1, which specifies a value of -1 in order to decrease the number of running instances by one each time this policy is executed.

###### An ARN should be returned similar to this: arn:aws:autoscaling:us-west-2:327756672493:scalingPolicy:5918d1b7-3a82-446b-b112-c8cfbc7b847c:autoScalingGroupName/lab-as-group:policyName/lab-scale-down-policy

### CREATE A CLOUDWATCH HIGH CPU ALERT

###### In this section you create a CloudWatch alarm to monitor the aggregate average CPU of the Auto Scaling fleet and to trigger the lab-scale-up-policy.

###### In the AWS Management Console, on the Services menu, select CloudWatch.

###### In the navigation pane, select Alarms -> In alarm.

###### Select Create alarm.

###### Choose Select metric

###### Under Metrics, select EC2.

###### Select By Auto Scaling Group.

###### In the search  box, enter CPUUtilization

###### Select  lab-as-group

###### Select View graphed metrics tab, then configure:
* Statistic: Average
* Period: 1 Minute
###### Choose Select metric.
###### Scroll down to the Conditions section
* Whenever CPU Utilization is… select  Greater/Equal
* than: enter 40

###### Note: This is referring to the instance using 40% CPU utilization or more for 1 datapoint in 1 time interval (1 minute).

###### Select Next.
###### For Notification, configure:
* Under Alarm state trigger section, select In alarm
* Select an SNS Topic:  Select an existing SNS topic
* Send a notification to… in the text box, select lab-as-topic

###### Note: Under email endpoint, the email address you added to the SNS subscription will be displayed.

###### Under Auto Scaling action, select Add Autoscaling action.
* Under Alarm state trigger section, select In alarm
* Resource type:  EC2 Auto Scaling group.
* Select a group choose lab-as-group
* Take the following action select lab-scale-up-policy (Add 1 instance)
###### Select Next.
###### Enter a unique name under Alarm name: enter High CPU Alarm
###### Select Next.
###### Review your settings, then select Create alarm.
### CREATE A CLOUDWATCH LOW CPU ALERT
###### In this section (using previous steps as a guide) configure a CPU utilization alert when usage is <=20% to trigger the lab-scale-down-policy.

###### In the navigation pane, click Alarms -> In alarm.

###### Choose Create alarm.

###### Choose Select metric

###### Under Metrics, select EC2.

###### Select By Auto Scaling Group.

###### In the search  box, enter CPUUtilization

###### Select  lab-as-group

###### Select View graphed metrics tab, then configure:
# Statistic: Average
# Period: 1 Minute

###### Choose Select metric.

###### Scroll down to the Conditions
* Whenever CPU Utilization is… select  Lower/Equal
* than: enter 20
###### Note: This is referring to the instance using 20% CPU utilization or less for 1 datapoint in 1 time interval (1 minute).

###### Select Next.
###### For Notification, configure:
* Under Alarm state trigger section, select in Alarm

* Select an SNS Topic:  Select an existing SNS topic

* Send a notification to… in the text box, select lab-as-topic

###### Note: Under email endpoint, the email address you added to the SNS subscription will be displayed.

###### Under Auto Scaling action, select Add Autoscaling action.
* Under Alarm state trigger section, select in Alarm

* Resource type:  EC2 Auto Scaling group.

* Select a group choose lab-as-group

* Take the following action select lab-scale-down-policy (Remove 1 instance)

###### Select Next.

###### Define a unique name:, enter Low CPU Alarm

###### Select Next.

###### eview your settings, then select Create alarm.

###### Note AWS recommends configuring Auto Scaling policies to scale up quickly and scale down slowly. This allows the application to better respond to increased traffic loads after a scale-up event and to make more economical use of the AWS hourly billing cycle. The example here is intentionally simple. From a billing perspective, it costs no more if the instance is scaled down after three minutes than if it is allowed to run for 59 minutes.

### Test Auto Scaling

###### All pieces are in place to demonstrate Auto Scaling based on application usage. You have an Auto Scaling group with a minimum of two instances and a maximum of four instances. You also have Auto Scaling policies to increase and decrease the group by one instance, and you created CloudWatch alarms to trigger policies when the aggregate average CPU utilization of the group is ≥ 40% and < 20%. Currently, one instance is running because the minimum size is one and the group is not currently under any load. Also, if you view your list of alarms, the two alarms you created should be in two different states: The High CPU alarm should be in the OK state (because your total CPU utilization is very low), and the Low CPU alarm should be in the ALARM state (because your total CPU utilization indicates that you have a surplus of processing cycles).

###### Even though your CPU utilization is below 20%, Auto Scaling is not removing instances because the group size is currently at its minimum (one). Also, remember that the cooldown time for the Auto Scaling polices is five minutes (300 seconds). This is important because it influences how quickly you see the Auto Scaling activities.

###### In this section, you see what happens with the scaling policies you have configured when you put additional strain on the servers.

###### On the Services menu, click EC2.

###### In the navigation pane, click Load Balancers.

###### Copy the public DNS name value of your load balancer to your Clipboard (without the “(A Record)” text.

###### In a new browser window, paste the public DNS name in the address bar. You should only see a single instance.

###### Click Generate Load

###### The CPU Load will jump to 100% (you may have to refresh your browser to see the CPU Load increase).

###### Note This button triggers a simple background process to copy, zip, and unzip ~1 GB of nothing (/dev/zero) for 10–20 minutes.

###### On the Services menu, click CloudWatch.

###### Click Alarms.

###### Click High CPU Alarm.

###### In a few minutes you should see the Low CPU alarm state change to OK and the High CPU alarm state change to Alarm. Additionally, you should receive an email notification from Auto Scaling informing you that a scale-up action was triggered.

###### You will now return to the EC2 console and examine the instance autoscaling created.

###### On the Services menu, click EC2.
###### Click Instances. A new instance has been added to your group.
###### Switch to the browser tab that points to the A record for your load balancer, and refresh the load balancer several times to see that one server is under heavy load, while the other is not:

<img width="437" alt="Screen Shot 2022-08-10 at 9 42 43 PM" src="https://user-images.githubusercontent.com/67527927/184050958-fb66c3c3-3fea-4729-9e3d-e59adff2bbb9.png">
###### Return to the CloudWatch Dashboard to view the CPU utilization of the fleet.
###### You probably triggered another Auto Scaling scale-up event because your server fleet average CPU equaled 40% (one instance at ~100% and the other at ~0%), and the alarm trigger was set to ≥ 40%.

###### After 15–20 minutes, your Auto Scaling fleet should have scaled up to two or three instances and then scaled back down to one instance. Also note that the instances were terminated in launch order, meaning that the “oldest” instances were terminated first. This allows you to roll in new changes to your application by updating your launch configuration to a newer AMI and then triggering Auto Scaling events (increase the minimum size).

###### Note You have probably noticed that this is not an overly realistic test because it essentially simulates a user overloading a single server and does not take advantage of your load balancer. In this case, Auto Scaling helps additional clients but does not load balance this “work” across multiple servers.

### Wrapping Up

#### VIEWING AUTO SCALING ACTIVITIES

###### The Auto Scaling API provides a programmatic way to display all of the Auto Scaling activities that have occurred. You can use the aws autoscaling describe-scaling-activities command to demonstrate this capability. For example:
```
aws autoscaling describe-scaling-activities --auto-scaling-group-name lab-as-group
```
#### SUSPENDING AND RESUMING AUTO SCALING PROCESSES
###### Auto Scaling also allows you to intentionally tell an Auto Scaling group to suspend or resume Auto Scaling processes using the as-suspend-processes and as-resume processes commands. This can be helpful if you know ahead of time that certain activities (such as maintenance events) will trigger Auto Scaling alerts, but you do not want instances to be automatically added or removed during the course of the event.

#### HOW LARGE AN AUTO SCALING FARM CAN I HAVE?
###### By default each account is assigned limits on a per-region basis. Initially, accounts are set to a maximum of 20 Amazon EC2 instances, 100 spot instances, and 5000 EBS volumes or an aggregate size of 20 TB (whichever is smaller).

###### The max-instances limit only applies for instances that are in the pending, running, shutting-down, and stopping states. There is another limit, which is set to 4 times the max-instances limit that governs the total number of instances in any state. For a default account with no limit overrides, you can have 20 running instances and up to 60 stopped instances, bringing this to a total of 80 instances.




