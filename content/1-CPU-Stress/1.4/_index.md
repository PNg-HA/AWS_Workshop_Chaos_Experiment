---
title : "Detect & Remediate CPU Stress with scripts"
date : "`r Sys.Date()`"
weight : 4
chapter : false
pre : " <b> 1.4. </b> "
---

#### Part 1: Detect
##### 1. Prerequisites

Required:
- Python 3.9 or higher.
- GitHub account.
- GitHub token.
- GitHub repository.

How to create a GitHub token:

Step 1: After logging in to the GitHub website, go to github.com/settings/tokens -> Select Generate new token:
![1.4](/images/1/1.4/Picture1.png)
Step 2: Select Generate new token (classic):
![1.4](/images/1/1.4/Picture2.png)
Step 3: Configure the duration and permissions as needed (I configure full permissions) and then select "Generate token":
![1.4](/images/1/1.4/Picture3.png)
Step 4: Save this token for use later.

How to create a GitHub repository: Refer to https://docs.github.com/en/repositories/creating-and-managing-repositories/quickstart-for-repositories.

In this article, I created a repository named AWS-Chaos-Experiments on the account PNG-HA.
![1.4](/images/1/1.4/Picture4.png)
##### 2. Set up
a. Edit the script

Edit the code in the file scripts/detect_issues.py in the reference repository (mostlycloudysky/aws-chaos-experiments) to use credentials directly in the code. This is not recommended as it is not a best practice for security.

```python
DEFAULT_AWS_REGION = "ap-northeast-2"
DEFAULT_GITHUB_REPO = "PNg-HA/AWS-Chaos-Experiments" (syntax is user/repo)
DEFAULT_GITHUB_TOKEN = "ghp_*******"
DEFAULT_CLOUDWATCH_ALARM_NAME = "CPUUtilizationAlarm"
```

Script has been updated:

```python
import os
import boto3
import requests
from datetime import datetime, timedelta

# Default environment variables
DEFAULT_AWS_REGION = "ap-northeast-2"
DEFAULT_GITHUB_REPO = "PNg-HA/AWS-Chaos-Experiments"
DEFAULT_GITHUB_TOKEN = "ghp_***********************"
DEFAULT_CLOUDWATCH_ALARM_NAME = "CPUUtilizationAlarm"

def detect_cpu_stress():
    print("Detecting CPU stress issues...")
    cloudwatch = boto3.client("cloudwatch", region_name=os.getenv("AWS_REGION", DEFAULT_AWS_REGION))

    alarm_name = os.getenv("CLOUDWATCH_ALARM_NAME", DEFAULT_CLOUDWATCH_ALARM_NAME)
    response = cloudwatch.describe_alarms(AlarmNames=[alarm_name])
    alarms = response["MetricAlarms"]

    if not alarms:
        print("No alarms found with the specified name.")
        return

    alarm_state = alarms[0]["StateValue"]

    if alarm_state == "ALARM":
        instance_id = alarms[0]["Dimensions"][0][
            "Value"
        ]  # Assuming the alarm is set on a single EC2 instance
        create_github_issue(instance_id)
    else:
        print("No CPU stress issues detected.")

def create_github_issue(instance_id):
    repo = os.getenv("GITHUB_REPO", DEFAULT_GITHUB_REPO)
    token = os.getenv("GITHUB_TOKEN", DEFAULT_GITHUB_TOKEN)
    url = f"https://api.github.com/repos/{repo}/issues"
    headers = {"Authorization": f"token {token}"}
    issue_title = f"EC2 Instance {instance_id} CPU Stress Detected"
    issue_body = f"The CloudWatch alarm for EC2 instance {instance_id} is in the ALARM state, indicating high CPU utilization. Remediation action is required."
    issue = {"title": issue_title, "body": issue_body}
    response = requests.post(url, headers=headers, json=issue)
    if response.status_code == 201:
        print(f"GitHub issue created for instance {instance_id}")
    else:
        print(
            f"Failed to create GitHub issue for instance {instance_id}: {response.text}"
        )
if __name__ == "__main__":
    detect_cpu_stress()
```
b. Perform instance stop detection

Step 1: Run the script (simultaneously run the cpu_stress experiment)
![1.4](/images/1/1.4/Picture5.png)
Step 2: Check the issue on your personal GitHub repo
![1.4](/images/1/1.4/Picture6.png)
Step 3: Observe issue details
![1.4](/images/1/1.4/Picture7.png)
The Instance ID in the issue is the same as the EC instance ID mentioned above. Thus, the instance experiencing CPU stress has been successfully detected by the script.


#### Part 2: Remediate

Reference: file scripts/remediate_stopped_instances.py from the repo mostlycloudysky/aws-chaos-experiments.

##### 1. Edit the script

As in Part 1, place the credentials into the script. Besides the region, repository, and token, this script requires:

```python
DEFAULT_GITHUB_ISSUE_NUMBER: the number (#) of the issue you want to address.
```
Additionally, an environment variable for issue_body is needed, but I will directly add this value as the default for the getenv() function.

Script has been updated:

```python
import os
import boto3
import requests
from datetime import datetime, timedelta

# Default environment variables
DEFAULT_AWS_REGION = "ap-northeast-2"
DEFAULT_GITHUB_REPO = "PNg-HA/AWS-Chaos-Experiments"
DEFAULT_GITHUB_TOKEN = "ghp_***********************"
DEFAULT_CLOUDWATCH_ALARM_NAME = "CPUUtilizationAlarm"
DEFAULT_GITHUB_ISSUE_NUMBER = "<number>"

def detect_cpu_stress():
    print("Detecting CPU stress issues...")
    cloudwatch = boto3.client("cloudwatch", region_name=os.getenv("AWS_REGION", DEFAULT_AWS_REGION))

    alarm_name = os.getenv("CLOUDWATCH_ALARM_NAME", DEFAULT_CLOUDWATCH_ALARM_NAME)
    response = cloudwatch.describe_alarms(AlarmNames=[alarm_name])
    alarms = response["MetricAlarms"]

    if not alarms:
        print("No alarms found with the specified name.")
        return

    alarm_state = alarms[0]["StateValue"]

    if alarm_state == "ALARM":
        instance_id = alarms[0]["Dimensions"][0][
            "Value"
        ]  # Assuming the alarm is set on a single EC2 instance
        create_github_issue(instance_id)
    else:
        print("No CPU stress issues detected.")

def create_github_issue(instance_id):
    repo = os.getenv("GITHUB_REPO", DEFAULT_GITHUB_REPO)
    token = os.getenv("GITHUB_TOKEN", DEFAULT_GITHUB_TOKEN)
    url = f"https://api.github.com/repos/{repo}/issues"
    headers = {"Authorization": f"token {token}"}
    issue_title = f"EC2 Instance {instance_id} CPU Stress Detected"
    issue_body = f"The CloudWatch alarm for EC2 instance {instance_id} is in the ALARM state, indicating high CPU utilization. Remediation action is required."
    issue = {"title": issue_title, "body": issue_body}
    response = requests.post(url, headers=headers, json=issue)
    if response.status_code == 201:
        print(f"GitHub issue created for instance {instance_id}")
    else:
        print(
            f"Failed to create GitHub issue for instance {instance_id}: {response.text}"
        )
if __name__ == "__main__":
    detect_cpu_stress()
```

##### 2. Resolve the issue

Run the script to fix issue #4:
![1.4](/images/1/1.4/Picture8.png)
The script will stop the heavy job on the instance (and will not close the issue).

Monitor via CloudWatch:
![1.4](/images/1/1.4/Picture9.png)
After the CPU exceeds the threshold (at the time of the alarm), begin to gradually reduce (after activating the remediation script).

Check the instance through CloudTrail events:
![1.4](/images/1/1.4/Picture10.png)
![1.4](/images/1/1.4/Picture11.png)
Check the 'Run command' that was executed for remediation in System Manager:
![1.4](/images/1/1.4/Picture12.png)
Noticed overlapping timestamps (on SSM and on CloudTrail are UTC). Additionally, when stopping the job, the experiment does not fail:
![1.4](/images/1/1.4/Picture13.png)
Thus, the detection and resolution of CPU stress have been successfully implemented.
