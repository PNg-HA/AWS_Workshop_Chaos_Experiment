---
title : "Detect & Remediate Network Latency with scripts"
date : "`r Sys.Date()`"
weight : 4
chapter : false
pre : " <b> 2.4. </b> "
---

#### Part 1: Detect

##### 1. Prerequisites

As in the CPU Stress section

##### 2. Set up

a. Edit the script

Edit the code in the file scripts/detect_issues.py in the reference repository (mostlycloudysky/aws-chaos-experiments) to use credentials directly in the code. This is not recommended as it is not a best practice for security.

```python
DEFAULT_AWS_REGION = "ap-northeast-2"
DEFAULT_GITHUB_REPO = "PNg-HA/AWS-Chaos-Experiments" (syntax is user/repo)
DEFAULT_GITHUB_TOKEN = "ghp_**************************"
DEFAULT_CLOUDWATCH_ALARM_NAME = "NetworkInAlarm" (there are 2 alarms for the network, but the reference repo uses NetworkIn)
```

Script has been updated:
![2.4](/images/2/2.4/Picture1.png)
b. Perform network latency detection

Step 1: Run the script (simultaneously run the experiment with a network latency delay of 30 seconds due to the instance being too powerful).
![2.4](/images/2/2.4/Picture2.png)
Step 2: Check the issue on your personal GitHub repo
![2.4](/images/2/2.4/Picture3.png)
Step 3: Observe the issue details.
![2.4](/images/2/2.4/Picture4.png)
The Instance ID in the issue matches the EC instance ID mentioned above. This indicates that the instance experiencing network latency was successfully detected using the script.

#### Part 2: Remediate

Reference: the scripts/remediate_network_latency.py file from the mostlycloudysky/aws-chaos-experiments repository.

##### 1. Edit the script

As in part 1, place the credentials into the script. Besides the region, repo, and token, this script also requires:

```python
DEFAULT_GITHUB_ISSUE_NUMBER: The sequence number (#) of the issue to be resolved
```

Additionally, an environment variable is needed for the variable issue_body, but I will directly add this value as the default for the getenv() function.

Script has been updated:

```python
import os
import boto3
import requests
import re
import base64

# Default environment variables
DEFAULT_AWS_REGION = "ap-northeast-2"
DEFAULT_GITHUB_REPO = "PNg-HA/AWS-Chaos-Experiments"
DEFAULT_GITHUB_TOKEN = "ghp_***********************"
DEFAULT_GITHUB_ISSUE_NUMBER = "5"

def run_ssm_document(instance_id):
    ssm = boto3.client("ssm", region_name=os.getenv("AWS_REGION", DEFAULT_AWS_REGION))
    script = "echo Hello"
    encoded_script = base64.b64encode(script.encode("utf-8")).decode("utf-8")
    response = ssm.start_automation_execution(
        DocumentName="AWSSupport-StartEC2RescueWorkflow",
        Parameters={
            "InstanceId": [instance_id],
            "AllowEncryptedVolume": ["True"],
            "OfflineScript": [encoded_script],
        },
    )
    execution_id = response["AutomationExecutionId"]
    print(f"Started SSM document on instance {instance_id}: {execution_id}")

def parse_body(issue_body):
    match = re.search(r"EC2 instance (\bi-\w+\b)", issue_body)
    if match:
        return match.group(1)
    else:
        raise ValueError("Instance ID not found in issue body")

def remediate():
    issue_body = os.getenv("ISSUE_BODY", "The NetworkIn alarm for EC2 instance i-06771d5fe9accdc18 has been triggered. Remediation action is required.")
    try:
        instance_id = parse_body(issue_body)
        print(f"Instance ID to remediate: {instance_id}")
        run_ssm_document(instance_id)
    except Exception as e:
        print(f"Error during remediation: {e}")
        reopen_issue()

def reopen_issue():
    issue_number = os.getenv("ISSUE_NUMBER", DEFAULT_GITHUB_ISSUE_NUMBER)
    repo = os.getenv("GITHUB_REPO", DEFAULT_GITHUB_REPO)
    token = os.getenv("GITHUB_TOKEN", DEFAULT_GITHUB_TOKEN)
    url = f"https://api.github.com/repos/{repo}/issues/{issue_number}"
    headers = {"Authorization": f"token {token}"}
    data = {"state": "open"}
    response = requests.patch(url, headers=headers, json=data)
    if response.status_code == 200:
        print(f"Reopened issue {issue_number}")
    else:
        print(f"Failed to reopen issue {issue_number}: {response.content}")
if __name__ == "__main__":
    remediate()
```

##### 2. Resolve the issue

Run the script to fix issue #:
![2.4](/images/2/2.4/Picture5.png)
The most important part of the script is running the AWSSupport-StartEC2RescueWorkflow document (see https://docs.aws.amazon.com/systems-manager-automation-runbooks/latest/userguide/automation-awssupport-startec2rescueworkflow.html). The script launches a new instance (with a name containing "rescue"), attaches the volume from the problematic instance, and then fixes it using the pre-configured script (variable "script"). However, it is confusing that the script only runs “echo Hello” instead of making any repairs.

Check the logic through CloudTrail:
![2.4](/images/2/2.4/Picture6.png)
Check the instance on the EC2 page. Initially, the EC2Rescue instance was created:
![2.4](/images/2/2.4/Picture7.png)
A few seconds later, start shutting down the instance in the issue to detach the volume:
![2.4](/images/2/2.4/Picture8.png)
Then, the instance for the experiment was stopped completely:
![2.4](/images/2/2.4/Picture9.png)
A few minutes later, the new instance shuts down, and the instance for the experiment is restarted.
![2.4](/images/2/2.4/Picture10.png)
Monitor via CloudWatch:
![2.4](/images/2/2.4/Picture11.png)
Thus, the detection and resolution of the network latency issue on the EC2 instance have been successfully implemented.
