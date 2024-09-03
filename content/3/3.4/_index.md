---
title : "Detect & Remediate Stopped Instances with scripts"
date : "`r Sys.Date()`"
weight : 4
chapter : false
pre : " <b> 3.4. </b> "
---

#### Part 1: Detect
##### 1. Prerequisites

Like the CPU Stress section.

##### 2. Set up

a. Edit the script

Edit the code in the file scripts/detect_issues.py in the reference repository (mostlycloudysky/aws-chaos-experiments) to use credentials in the code. Note that this approach is not recommended as it is not a best practice for security.

```python
DEFAULT_AWS_REGION = "ap-northeast-2"
DEFAULT_GITHUB_REPO = "PNg-HA/AWS-Chaos-Experiments" (syntax is user/repo)
DEFAULT_GITHUB_TOKEN = "ghp_*******"
```

Script has been updated:

```python
import os
import boto3
import requests
import json

# Default environment variables
DEFAULT_AWS_REGION = "ap-northeast-2"
DEFAULT_GITHUB_REPO = "PNg-HA/AWS-Chaos-Experiments"
DEFAULT_GITHUB_TOKEN = "ghp_***********************"

def detect_issues():
    print("Detecting issues...")
    
    # Use environment variables or fall back to default values
    aws_region = os.getenv("AWS_REGION", DEFAULT_AWS_REGION)
    ec2 = boto3.client("ec2", region_name=aws_region)
    
    response = ec2.describe_instances(
        Filters=[{"Name": "instance-state-name", "Values": ["stopped"]}]
    )
    
    instance_ids = []
    for reservation in response["Reservations"]:
        for instance in reservation["Instances"]:
            instance_ids.append(instance["InstanceId"])

    if not instance_ids:
        print("No issues detected.")
        return

    # Call create_github_issues for each instance
    for instance_id in instance_ids:
        create_github_issues(instance_id)

def create_github_issues(instance_id):
    # Use environment variables or fall back to default values
    repo = os.getenv("GITHUB_REPO", DEFAULT_GITHUB_REPO)
    token = os.getenv("GITHUB_TOKEN", DEFAULT_GITHUB_TOKEN)
    url = f"https://api.github.com/repos/{repo}/issues"
    
    headers = {"Authorization": f"token {token}"}
    issue_title = f"EC2 Instance {instance_id} Stopped"
    issue_body = f"The EC2 instance {instance_id} is currently stopped. Remediation action is required."
    issue = {"title": issue_title, "body": issue_body}
    
    response = requests.post(url, headers=headers, json=issue)
    if response.status_code == 201:
        print(f"GitHub issue created for instance {instance_id}")
    else:
        print(f"Failed to create GitHub issue for instance {instance_id}: {response.text}")

if __name__ == "__main__":
    detect_issues()
```

b. Perform instance stop detection

Step 1: Run the script
![3.4](/images/3/3.4/Picture1.png)
Step 2: Check the issue on your personal GitHub repo
![3.4](/images/3/3.4/Picture2.png)
Step 3: Observe the issue details.
![3.4](/images/3/3.4/Picture3.png)
Instance ID in the issue is similar to the EC instance ID mentioned above. This means that the instance has been successfully detected as stopped by the script.

#### Part 2: Remediate

Reference: file scripts/remediate_stopped_instances.py from the repo mostlycloudysky/aws-chaos-experiments.

##### 1. Edit the script

As in Part 1, place the credentials into the script. In addition to the region, repo, and token, this script requires:

```python
DEFAULT_GITHUB_ISSUE_NUMBER: the issue number (#) that the issue wants to address
```

Additionally, an environment variable for the issue_body variable is needed, but I will directly add this value as the default for the getenv() function.

Script has been updated:

```python
import os
import boto3
import re
import requests

# Default environment variables
DEFAULT_AWS_REGION = "ap-northeast-2"
DEFAULT_GITHUB_REPO = "PNg-HA/AWS-Chaos-Experiments"
DEFAULT_GITHUB_TOKEN = "ghp_***********************"
DEFAULT_GITHUB_ISSUE_NUMBER = "<number>"
def start_instance(instance_id):
    aws_region = os.getenv("AWS_REGION", DEFAULT_AWS_REGION)
    ec2 = boto3.client("ec2", region_name=aws_region)
    response = ec2.start_instances(InstanceIds=[instance_id])
    print(f"Starting instance {instance_id}: {response}")

def parse_issue_body(issue_body):
    # Use regex to extract instance ID from the issue body
    match = re.search(r"EC2 instance (\bi-\w+\b)", issue_body)
    if match:
        return match.group(1)
    else:
        raise ValueError("Instance ID not found in issue body...")

def reopen_issue():
    repo = os.getenv("GITHUB_REPO", DEFAULT_GITHUB_REPO)
    issue_number = os.getenv("GITHUB_ISSUE_NUMBER", DEFAULT_GITHUB_ISSUE_NUMBER)
    token = os.getenv("GITHUB_TOKEN", DEFAULT_GITHUB_TOKEN)
    url = f"https://api.github.com/repos/{repo}/issues/{issue_number}"
    headers = {"Authorization": f"token {token}"}
    data = {"state": "open"}

    response = requests.patch(url, headers=headers, json=data)
    if response.status_code == 200:
        print(f"Issue #{issue_number} reopened successfully.")
    else:
        print(f"Failed to reopen issue #{issue_number}: {response.text}")

def remediate():
    issue_body = os.getenv("ISSUE_BODY", "The EC2 instance i-06771d5fe9accdc18 is currently stopped. Remediation action is required.")
    try:
        instance_id = parse_issue_body(issue_body)
        print(f"Instance ID to remediate: {instance_id}")
        start_instance(instance_id)
    except Exception as e:
        print(f"Error during remediation: {e}")
        reopen_issue()

if __name__ == "__main__":
    remediate()
```
##### 2. Resolve the issue

Run the script to fix issue #3:
![3.4](/images/3/3.4/Picture4.png)
Reformat the JSON output:

```bash
{'StartingInstances': [
        {'CurrentState': {'Code': 0, 'Name': 'pending'
            }, 'InstanceId': 'i-06771d5fe9accdc18', 'PreviousState': {'Code': 80, 'Name': 'stopped'
            }
        }
    ], 'ResponseMetadata': {'RequestId': 'b55130d0-4bd5-4937-a2de-94e1201d65e6', 'HTTPStatusCode': 200, 'HTTPHeaders': {'x-amzn-requestid': 'b55130d0-4bd5-4937-a2de-94e1201d65e6', 'cache-control': 'no-cache, no-store', 'strict-transport-security': 'max-age=31536000; includeSubDomains', 'content-type': 'text/xml;charset=UTF-8', 'content-length': '411', 'date': 'Wed,
            21 Aug 2024 10: 13: 45 GMT', 'server': 'AmazonEC2'
        }, 'RetryAttempts': 0
    }
}
```

The script only launches an instance in the issue without encountering errors (and also does not close the issue).

Check the instance through CloudTrail event StartInstances:
![3.4](/images/3/3.4/Picture5.png)
Check the instance on the EC2 page and see that the times are inconsistent (on EC2 it is UTC+7 while on CloudTrail it is UTC):
![3.4](/images/3/3.4/Picture6.png)
Thus, the detection and remediation of the stopped EC2 instance has been successfully carried out.