---
title : "Set up"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 3.2. </b> "
---

#### 1. Create template

Step 1: Go to the Fault Injection Service page on the console -> Experiment templates page -> Select Create experiment template -> Name it
![3.2](/images/3/3.2/Picture1.png)
Step 2: Add target
![3.2](/images/3/3.2/Picture2.png)
Step 3: Add action

Uncheck the 'Completed if instances terminated' line in this case because we are testing while the instance is running.
![3.2](/images/3/3.2/Picture3.png)
Step 4: Review
![3.2](/images/3/3.2/Picture4.png)
Step 5: Select IAM role (This role has a Trusted Entity of FIS)
![3.2](/images/3/3.2/Picture5.png)
Step 6: Leave the remaining settings as default. Select Create experiment template.
![3.2](/images/3/3.2/Picture6.png)
Enter "Create":
![3.2](/images/3/3.2/Picture7.png)
Successfully created the template:
![3.2](/images/3/3.2/Picture8.png)
Go to the Export section to see how to install this template in CMD:
![3.2](/images/3/3.2/Picture9.png)
```bash
aws fis create-experiment-template ^
    --cli-input-json "{\"description\":\"Experiment to stop EC2 instances\",\"targets\":{\"TargetInstances\":{\"resourceType\":\"aws:ec2:instance\",\"resourceArns\":[\"arn:aws:ec2:ap-northeast-2:590183822512:instance/i-06771d5fe9accdc18\"],\"selectionMode\":\"ALL\"}},\"actions\":{\"StopInstances\":{\"actionId\":\"aws:ec2:stop-instances\",\"description\":\"Stop target EC2 instance\",\"parameters\":{\"completeIfInstancesTerminated\":\"false\"},\"targets\":{\"Instances\":\"TargetInstances\"}}},\"stopConditions\":[{\"source\":\"none\"}],\"roleArn\":\"arn:aws:iam::590183822512:role/HaiAnh-FIS\",\"tags\":{\"Name\":\"stop-instance\"},\"experimentOptions\":{\"accountTargeting\":\"single-account\",\"emptyTargetResolutionMode\":\"fail\"}}"
```

#### 2. Run the template

Step 1: You can run the template on the CLI using the command

```bash
aws fis start-experiment --experiment-template-id <template-id>
```
In this particular case:

```bash
aws fis start-experiment --experiment-template-id EXTADVWqNsdEa5UBa
```

Or run it on the console by going to the Experiment templates page, selecting the template you just created, and then clicking 'Start experiment'.
![3.2](/images/3/3.2/Picture10.png)

Select Start experiment:
![3.2](/images/3/3.2/Picture11.png)
Step 2: Enter "start"
![3.2](/images/3/3.2/Picture12.png)
According to the configuration, FIS will link to the EC2 API to stop the instance, so there is no need to check SSM.

Run for a few seconds. When done, return to the experiment page:
![3.2](/images/3/3.2/Picture13.png)
Check the detailed Action of the experiment:
![3.2](/images/3/3.2/Picture14.png)