# Scale and Load Balance Your Architecture

Lab overview
This lab walks you through using the Elastic Load Balancing (ELB) and Auto Scaling services to load balance and automatically scale your infrastructure.

Elastic Load Balancing automatically distributes incoming application traffic across multiple Amazon EC2 instances. It enables you to achieve fault tolerance in your applications by seamlessly providing the required amount of load balancing capacity needed to route application traffic.

Auto Scaling helps you maintain application availability and allows you to scale your Amazon EC2 capacity out or in automatically according to conditions you define. You can use Auto Scaling to help ensure that you are running your desired number of Amazon EC2 instances. Auto Scaling can also automatically increase the number of Amazon EC2 instances during demand spikes to maintain performance and decrease capacity during lulls to reduce costs. Auto Scaling is well suited to applications that have stable demand patterns or that experience hourly, daily, or weekly variability in usage.

You start with the following infrastructure:

Architecture - Lab 2

The final state of the infrastructure is:

Architecture - Lab 3

## OBJECTIVES
After completing this lab, you can:

Create a load balancer.
Create a launch template and an Auto Scaling group.
Automatically scale new instances within a private subnet
Create Amazon CloudWatch alarms and monitor performance of your infrastructure.
DURATION
This lab takes approximately 45 minutes.

Start lab
To launch the lab, at the top of the page, choose Start lab.
 You must wait for the provisioned AWS services to be ready before you can continue.

To open the lab, choose Open Console.
You are automatically signed in to the AWS Management Console in a new web browser tab.

 Do not change the Region unless instructed.

COMMON SIGN-IN ERRORS
Error: You must first sign out


If you see the message, You must first log out before logging into a different AWS account:

Choose the click here link.
Close your Amazon Web Services Sign In web browser tab and return to your initial lab page.
Choose Open Console again.
Error: Choosing Start Lab has no effect
In some cases, certain pop-up or script blocker web browser extensions might prevent the Start Lab button from working as intended. If you experience an issue starting the lab:

Add the lab domain name to your pop-up or script blocker’s allow list or turn it off.
Refresh the page and try again.
Task 1: Create a Load Balancer
In this task, you will create a load balancer that can balance traffic across multiple EC2 instances and Availability Zones.

Choose Services and search for 

EC2
.

In the left navigation pane, click Load Balancers.

Click Create load Balancer

Several different types of load balancer are displayed. You will be using an Application Load Balancer that operates at the request level (layer 7), routing traffic to targets — EC2 instances, containers, IP addresses and Lambda functions — based on the content of the request. For more information, see: Comparison of Load Balancers

Under Application Load Balancer click Create and configure:
Load balancer name: 

LabELB
VPC: Lab VPC (In the Network mapping section)
Mappings: Select  both to see the available subnets.
Select Public Subnet 1 and Public Subnet 2
This configures the load balancer to operate across multiple Availability Zones.

Under Security groups select  Web Security Group and deselect  default.
This Web Security Group has already been created for you, which permits HTTP access.

Routing configures where to send requests that are sent to the load balancer. You will create a Target Group that will be used by Auto Scaling.

Under Listeners and routing choose Create target group.

For Target group name, enter: 

LabGroup

Choose Next

Auto Scaling will automatically register instances as targets later in the lab.

Choose Create target group

Switch back to the Load Balancer tab. Under the Forward to drop-down select 

LabGroup
.

Choose Create load balancer

The load balancer will show a state of provisioning. There is no need to wait until it is ready. Please continue with the next task.

Task 2: Create a Launch Template and an Auto Scaling Group
In this task, you will create a launch template for your Auto Scaling group. A launch template is a template that an Auto Scaling group uses to launch EC2 instances. When you create a launch template, you specify information for the instances such as the AMI, the instance type, a key pair, security group and disks.

In the left navigation pane, click Launch Templates.

Click Create launch template

Configure these settings:

Launch template name: 

LabTemplate

Under Application and OS Images (Amazon Machine Image) choose Amazon Linux.

Under Amazon Machine Image (AMI) choose Amazon Linux 2 AMI.

Instance type: t3.micro.

Key pair name: Don’t include in launch template.

Security groups: Web Security Group. Make sure the security group belongs to Lab VPC.

Expand Advanced details and in the Detailed CloudWatch monitoring menu, select Enable

Under User Data paste in the following:


#!/bin/bash -ex
yum -y update
yum -y install httpd php mysql php-mysql
amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
yum install -y httpd mariadb-server
systemctl start httpd
systemctl enable httpd
cd /var/www/html
wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-100-RSJAWS-1-49486/174-lab-JAWS-scale-load-balance/scripts/lab-app-php7.zip
unzip lab-app-php7.zip -d /var/www/html/
chown apache:root /var/www/html/rds.conf.php
This will capture metrics at 1-minute intervals, which allows Auto Scaling to react quickly to changing usage patterns.

Click Create launch template followed by View launch templates
You will now create an Auto Scaling group that uses this Launch Template.

Select  LabTemplate and then in the Actions  menu, select Create Auto Scaling group

Configure the following settings:

Auto Scaling group name: 

Lab Auto Scaling Group

Click Next

Network: Lab VPC

Subnet: Select Private Subnet 1 (10.0.1.0/24) and Private Subnet 2 (10.0.3.0/24)

Click Next

Load balancing

Select  Attach to an existing load balancer

Select  Choose from your load balancer target groups.

Existing load balancer target groups LabGroup

Monitoring: Select  Enable group metrics collection within CloudWatch.

This will capture metrics at 1-minute intervals, which allows Auto Scaling to react quickly to changing usage patterns.

Click Next

Group size: Enter the below values

Desired capacity: 

2

Minimum capacity: 

2

Maximum capacity: 

4

This will allow Auto Scaling to automatically add/remove instances, always keeping between 2 and 4 instances running.

Scaling policies - optional

Select  Target tracking scaling policy

Metric type: Average CPU Utilization

Target value: 

60

This tells Auto Scaling to maintain an average CPU utilization across all instances at 60%. Auto Scaling will automatically add or remove capacity as required to keep the metric at, or close to, the specified target value. It adjusts to fluctuations in the metric due to a fluctuating load pattern.

Click Next

Click Next again on the Add notifications section.

Add tags: click Add tag and enter

Key: 

Name

Value: 

Lab Instance

Click Next

Finally click Create Auto Scaling group

This will launch EC2 instances in private subnets across both Availability Zones.

Your Auto Scaling group will initially show an instance count of zero, but new instances will be launched to reach the Desired count of 2 instances. Note: If you experience an error related to the t3.micro instance type not being available, then rerun this task by selecting t2.micro instead.

Task 3: Verify that Load Balancing is Working
In this task, you will verify that Load Balancing is working correctly.

In the left navigation pane, click Instances.
You should see two new instances named Lab Instance. These were launched by Auto Scaling.

 If the instances or names are not displayed, wait 30 seconds and click refresh  in the top-right.

First, you will confirm that the new instances have passed their Health Check.

In the left navigation pane, click Target Groups (in the Load Balancing section).

Click LabGroup followed by the Targets tab.

Two Lab Instance targets should be listed for this target group.

Wait until the Status of both instances transitions to healthy. Click Refresh  in the upper-right to check for updates.
Healthy indicates that an instance has passed the Load Balancer’s health check. This means that the Load Balancer will send traffic to the instance.

You can now access the Auto Scaling group via the Load Balancer.

In the left navigation pane, click Load Balancers.

In the lower pane, copy the DNS name of the load balancer, making sure to omit “(A Record)”.

It should look similar to: LabELB-1998580470.us-west-2.elb.amazonaws.com

Open a new web browser tab, paste the DNS Name you just copied, and press Enter.
The application should appear in your browser. This indicates that the Load Balancer received the request, sent it to one of the EC2 instances, then passed back the result.

Task 4: Test Auto Scaling
You created an Auto Scaling group with a minimum of two instances and a maximum of four instances. Currently two instances are running because the minimum size is two and the group is currently not under any load. You will now increase the load to cause Auto Scaling to add additional instances.

Return to the AWS management console, but do not close the application tab — you will return to it soon.

In the AWS Management Console, select the  Services menu, and then select CloudWatch under Management & Governance.

In the left navigation pane, click expand Alarms and choose All alarms.

Two alarms will be displayed. These were created automatically by the Auto Scaling group. They will automatically keep the average CPU load close to 60% while also staying within the limitation of having two to six instances.

The alarm which has AlarmHigh in its name should show as OK.
 If no alarm is showing OK, wait a minute then click refresh  in the top-right until the alarm status changes.

The OK indicates that the alarm has not been triggered. It is the alarm for CPU Utilization > 60, which will add instances when average CPU is high. The chart should show very low levels of CPU at the moment.

You will now tell the application to perform calculations that should raise the CPU level.

Return to the browser tab with the web application.

Click Load Test beside the AWS logo.

This will cause the application to generate high loads. The browser page will automatically refresh so that all instances in the Auto Scaling group will generate load. Do not close this tab.

Return to browser tab with the CloudWatch console.
In less than 5 minutes, the AlarmLow alarm should change to OK and the AlarmHigh alarm status should change to ALARM.

 You can click Refresh  in the top-right every 60 seconds to update the display.

You should see the AlarmHigh chart indicating an increasing CPU percentage. Once it crosses the 60% line for more than 3 minutes, it will trigger Auto Scaling to add additional instances.

Wait until the AlarmHigh alarm enters the ALARM state.
You can now view the additional instance(s) that were launched.

In the AWS Management Console, select the  Services menu, and then select EC2 under Compute.

In the left navigation pane, click Instances.

More than two instances labeled Lab Instance should now be running. The new instance(s) were created by Auto Scaling in response to the Alarm.

End lab