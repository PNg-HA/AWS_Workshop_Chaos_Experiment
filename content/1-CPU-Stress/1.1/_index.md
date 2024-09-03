---
title : "Prerequisites"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 1.1. </b> "
---



#### 1. Create EC2 instance with Windows OS

Type and has a public IP:
![1.1](/images/1/1.1/Picture1.png) 

Security group of the instance:
![1.1](/images/1/1.1/Picture2.png) 

IAM instance profile:
![1.1](/images/1/1.1/Picture3.png) 

Check if the Windows instance has SSM:
![1.1](/images/1/1.1/Picture4.png) 

#### 2. Configure AWS CLI with least privileges (here using AdminFullAccess).

![1.1](/images/1/1.1/Picture5.png) 
#### 3. Configure CloudWatch Alarm

Step 1: Create an SNS topic by selecting Create Topic -> Enter a name in the Name field -> Select Create topic.
![1.1](/images/1/1.1/Picture6.png) 
![1.1](/images/1/1.1/Picture7.png) 

Subscribe:
![1.1](/images/1/1.1/Picture8.png) 

Step 2: Step 2 will be for CPU Usage. Select instance -> Action -> Monitor and troubleshoot -> Manage CloudWatch alarms
![1.1](/images/1/1.1/Picture9.png) 

Step 3: Create a CloudWatch alarm

Option 1: Create a CloudWatch alarm on an EC2 instance
![1.1](/images/1/1.1/Picture10.png) 

Set threshhold
![1.1](/images/1/1.1/Picture11.png) 

Option 2: Use AWS CLI to create an Alarm

Create an Alarm for CPU usage, NetworkIn, and NetworkOut according to the command in the mostlycloudysky/aws-chaos-experiments repo, replacing YourInstanceId and YourSNSTopicARN:

```bash
aws cloudwatch put-metric-alarm --alarm-name CPUUtilizationAlarm --alarm-description "Alarm when CPU exceeds 80%" --metric-name CPUUtilization --namespace AWS/EC2 --statistic Average --period 60 --threshold 80 --comparison-operator GreaterThanOrEqualToThreshold --dimensions Name=InstanceId,Value=i-06771d5fe9accdc18 --evaluation-periods 1 --alarm-actions arn:aws:sns:ap-northeast-2:590183822512:HaiAnh-FIS-stress-test
```
```bash
aws cloudwatch put-metric-alarm --alarm-name NetworkInAlarm --alarm-description "Alarm when NetworkIn is below 1000 bytes for 1 data point within 1 minute" --metric-name NetworkIn --namespace AWS/EC2 --statistic Average --period 60 --threshold 1000 --comparison-operator LessThanThreshold --dimensions Name=InstanceId,Value=i-06771d5fe9accdc18 --evaluation-periods 1 --alarm-actions arn:aws:sns:ap-northeast-2:590183822512:HaiAnh-FIS-stress-test
```

```bash
aws cloudwatch put-metric-alarm --alarm-name NetworkOutAlarm --alarm-description "Alarm when NetworkOut is below 1000 bytes for 1 data point within 1 minute" --metric-name NetworkOut --namespace AWS/EC2 --statistic Average --period 60 --threshold 1000 --comparison-operator LessThanThreshold --dimensions Name=InstanceId,Value=i-06771d5fe9accdc18 --evaluation-periods 1 --alarm-actions arn:aws:sns:ap-northeast-2:590183822512:HaiAnh-FIS-stress-test
```

![1.1](/images/1/1.1/Picture12.png) 

Check on the console:
![1.1](/images/1/1.1/Picture13.png) 

Check CPU Utilization alarm details:
![1.1](/images/1/1.1/Picture14.png) 
#### 4. Roles
Role for FIS: 

Step 1: Select the Trusted entity as FIS, use case for EC2:
![1.1](/images/1/1.1/Picture15.png)
![1.1](/images/1/1.1/Picture16.png)