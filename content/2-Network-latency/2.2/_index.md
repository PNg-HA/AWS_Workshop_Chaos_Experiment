---
title : "Set up"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 2.2. </b> "
---


The resources were created in the previous step; however, an additional policy is needed for the FIS role."

#### 1. Create a Template

Step 1: Go to the Fault Injection Service page on the console -> Experiment templates page -> Select Create experiment template -> Enter a name
![2.2](/images/2/2.2/Picture1.png)
Step 2: Add target
![2.2](/images/2/2.2/Picture2.png)
Step 3: Add action

Document parameters: From the JSON file.
![2.2](/images/2/2.2/Picture3.png)
Document parameters execute the purpose of delaying 1 second for network traffic passing through port 8080, using 2 commands:

```bash
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=8080 connectaddress=127.0.0.1
```
- Create a port proxy rule: forward traffic from the Internet to the loopback address on port 8080

```bash
netsh interface portproxy set v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=8080 connectaddress=127.0.0.1 delay=1000
```

- Edit rule from command 1: 1-second delay for forwarded traffic

Step 4: Review
![2.2](/images/2/2.2/Picture4.png)
Step 5: Choose IAM role (This role has Trusted Entity as FIS)
![2.2](/images/2/2.2/Picture5.png)
Step 6: Leave the remaining options as default. Select Create experiment template
![2.2](/images/2/2.2/Picture6.png)
Enter 'Create':
![2.2](/images/2/2.2/Picture7.png)
Template created successfully:
![2.2](/images/2/2.2/Picture8.png)
Go to the Export section to see how to install this template using CMD:
![2.2](/images/2/2.2/Picture9.png)
```bash
aws fis create-experiment-template ^
    --cli-input-json "{\"description\":\"Experiment to inject network latency on EC2 Windows instance\",\"targets\":{\"TargetInstances\":{\"resourceType\":\"aws:ec2:instance\",\"resourceArns\":[\"arn:aws:ec2:ap-northeast-2:590183822512:instance/i-06771d5fe9accdc18\"],\"selectionMode\":\"ALL\"}},\"actions\":{\"InjectNetworkLatency\":{\"actionId\":\"aws:ssm:send-command\",\"description\":\"Inject network latency on target EC2 instances\",\"parameters\":{\"documentArn\":\"arn:aws:ssm:ap-northeast-2::document/AWS-RunPowerShellScript\",\"documentParameters\":\"{\\\"commands\\\":[\\\"etsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=8080 connectaddress=127.0.0.1\\\", \\\"netsh interface portproxy set v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=8080 connectaddress=127.0.0.1 delay=1000\\\", \\\"Write-Output 'Introduced 1000ms network latency using netsh.'\\\"]}\",\"duration\":\"PT20M\"},\"targets\":{\"Instances\":\"TargetInstances\"}}},\"stopConditions\":[{\"source\":\"none\"}],\"roleArn\":\"arn:aws:iam::590183822512:role/HaiAnh-FIS\",\"tags\":{\"Name\":\"inject network latency\"},\"experimentOptions\":{\"accountTargeting\":\"single-account\",\"emptyTargetResolutionMode\":\"fail\"}}"
```

#### 2. Run the template

Step 1: You can run the template on the CLI using the command

```bash
aws fis start-experiment --experiment-template-id <template-id>
```

In this case specifically:
```bash
aws fis start-experiment --experiment-template-id EXT9VzeDWJYFwcJN4
```

Or run it on the console by going to the Experiment templates page, selecting the template you just created -> 'Start experiment':
![2.2](/images/2/2.2/Picture10.png)
![2.2](/images/2/2.2/Picture11.png)
Step 2: Enter 'start'
![2.2](/images/2/2.2/Picture12.png)
While running, check SSM:
![2.2](/images/2/2.2/Picture13.png)
Go back to the FIS experiment page:
![2.2](/images/2/2.2/Picture14.png)
Check detailed Action:
![2.2](/images/2/2.2/Picture15.png)
Timeline:
![2.2](/images/2/2.2/Picture16.png)