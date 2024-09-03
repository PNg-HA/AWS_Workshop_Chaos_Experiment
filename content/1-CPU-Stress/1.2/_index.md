---
title : "Set up"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 1.2. </b> "
---



#### 1. Create a template

Switching to the console because creating it on AWS CLI resulted in an invalid JSON error:
![1.2](/images/1/1.2/Picture1.png) 

But before encountering the invalid JSON error, if you encounter the error 'An error occurred (UnauthorizedException) when calling the CreateExperimentTemplate operation: Unauthorized', it means that the IAM role for FIS is missing permissions.

Step 1: Access the Fault Injection Service page in the console -> Navigate to the Experiment templates page -> Select Create experiment template -> Enter a name.
![1.2](/images/1/1.2/Picture2.png) 

Step 2: Add target
![1.2](/images/1/1.2/Picture3.png) 

Step 3: Add action

Document parameters: From the JSON file.
![1.2](/images/1/1.2/Picture4.png) 

Step 4: Review
![1.2](/images/1/1.2/Picture5.png) 

Step 5: Select the IAM role (This role has FIS as the Trusted Entity)
![1.2](/images/1/1.2/Picture6.png) 

Bước 6: Leave the remaining ones as default. Select Create experiment template.
![1.2](/images/1/1.2/Picture7.png) 

But it will result in an error:
![1.2](/images/1/1.2/Picture8.png) 

With a bit of finesse, we can observe that the AWS console automatically handles the input string, but not very "elegantly," as it adds escape characters `\` for each double quote, even though we had already added them. You can go to CloudTrail and look for the Event name CreateExperimentTemplate to observe this:
![1.2](/images/1/1.2/Picture9.png) 

Debugging process:
![1.2](/images/1/1.2/Picture10.png) 
- error: error displayed to the user.
- for input: incorrect input from the user entered into the DocumentParameters field.
- designed: correct input from the DocumentParameters parameter in the JSON file.
- Fixed input: correct input that the user needs to enter into the DocumentParameters field.

The correct input for DocumentParameters is: {"commands":["for ($i=0; $i -lt 8; $i++) {Start-Job -ScriptBlock {while ($true) {}}}","Start-Sleep -Seconds 600","Get-Job | Stop-Job"]} 

Result:
![1.2](/images/1/1.2/Picture11.png)
![1.2](/images/1/1.2/Picture12.png) 

The "Export ..." section explains why running AWS CLI on PowerShell results in an error. This is because it must be run on Linux and must be executed as specified in the export section, rather than being omitted as in the repo:


```bash
aws fis create-experiment-template \
    --cli-input-json '{
        "description": "Experiment to inject CPU stress on EC2 instances",
        "targets": {
            "TargetInstances": {
                "resourceType": "aws:ec2:instance",
                "resourceArns": [
                    "arn:aws:ec2:ap-northeast-2:590183822512:instance/i-06771d5fe9accdc18"
                ],
                "selectionMode": "ALL"
            }
        },
        "actions": {
            "InjectCPUStress": {
                "actionId": "aws:ssm:send-command",
                "description": "Inject CPU stress on target EC2 instances",
                "parameters": {
                    "documentArn": "arn:aws:ssm:ap-northeast-2::document/AWS-RunPowerShellScript",
                    "documentParameters": "{\"commands\":[\"for ($i=0; $i -lt 8; $i++) {Start-Job -ScriptBlock {while ($true) {}}}\",\"Start-Sleep -Seconds 600\",\"Get-Job | Stop-Job\"]}",
                    "duration": "PT20M"
                },
                "targets": {
                    "Instances": "TargetInstances"
                }
            }
        },
        "stopConditions": [
            {
                "source": "none"
            }
        ],
        "roleArn": "arn:aws:iam::590183822512:role/HaiAnh-FIS",
        "tags": {},
        "experimentOptions": {
            "accountTargeting": "single-account",
            "emptyTargetResolutionMode": "fail"
        }
    }'
```

Create experiment template on Linux:
![1.2](/images/1/1.2/Picture13.png) 

Full output:
![1.2](/images/1/1.2/Picture14.png) 

Check on the console:
![1.2](/images/1/1.2/Picture15.png) 
#### 2. Run Template

Step 1: Switch to "Start experiment CLI command" to see how to run the template on the CLI:
![1.2](/images/1/1.2/Picture16.png) 

Or run it on the console by going to the Experiment templates page and selecting 'Start experiment':
![1.2](/images/1/1.2/Picture17.png) 
![1.2](/images/1/1.2/Picture18.png) 

Step 2: Enter 'start':
![1.2](/images/1/1.2/Picture19.png) 

Result:
![1.2](/images/1/1.2/Picture20.png) 

While running, check CloudWatch:
![1.2](/images/1/1.2/Picture21.png) 

Check SSM:
![1.2](/images/1/1.2/Picture22.png) 
![1.2](/images/1/1.2/Picture23.png) 

Wait a few minutes:
![1.2](/images/1/1.2/Picture24.png) 

When that happens, go back to the experiment page:
![1.2](/images/1/1.2/Picture25.png) 

Check detailed Action:
![1.2](/images/1/1.2/Picture26.png) 

Timeline:
![1.2](/images/1/1.2/Picture27.png) 