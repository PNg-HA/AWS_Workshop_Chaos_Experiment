---
title : "Script phát hiện và giải quyết Network latency"
date : "`r Sys.Date()`"
weight : 4
chapter : false
pre : " <b> 2.4. </b> "
---

### Part 1: Detect

#### 1. Yêu cầu

Như mục CPU Stress

#### 2. Thiết lập

##### a. Chỉnh sửa script

Chỉnh sửa lại code file scripts/detect_issues.py trong repo tham khảo (mostlycloudysky/aws-chaos-experiments) để script dùng credential trong code, không khuyến khích làm theo vì không phải best practice cho security:

```python
DEFAULT_AWS_REGION = "ap-northeast-2"
DEFAULT_GITHUB_REPO = "PNg-HA/AWS-Chaos-Experiments" (cú pháp là user/repo)
DEFAULT_GITHUB_TOKEN = "ghp_**************************"
DEFAULT_CLOUDWATCH_ALARM_NAME = "NetworkInAlarm" (có 2 alarm cho network nhưng vì repo tham khảo dùng NetWorkIn)
```

Script đã sửa:
![2.4](/images/2/2.4/Picture1.png)

##### b. Thực hiện phát hiện network latency:

Bước 1: Chạy script (song song chạy experiment network latency delay 30s do cấu hình instance quá mạnh)
![2.4](/images/2/2.4/Picture2.png)
Bước 2: Kiểm tra issue trên github repo cá nhân
![2.4](/images/2/2.4/Picture3.png)
Bước 3: Quan sát chi tiết issue
![2.4](/images/2/2.4/Picture4.png)
Instance ID trong issue giống EC instance id ở phần trên. Như vậy đã phát hiện instance bị trễ mạng bằng script thành công.

### Part 2: Remediate

Tham khảo: file scripts/remediate_network_latency.py của repo mostlycloudysky/aws-chaos-experiments.

#### 1. Chỉnh sửa script

Như phần 1, đặt credential vào script. Ngoài region, repo và token, script này cần:

```python
DEFAULT_GITHUB_ISSUE_NUMBER: số thứ tự mà issue muốn giải quyết
```

Ngoài ra cần một biến môi trường cho biến issue_body, nhưng mình sẽ thêm trực tiếp giá trị này vào mặc định cho hàm getenv().

Script sau khi sửa:

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

#### 2. Thực hiện khắc phục issue

Chạy script khắc phục issue #:
![2.4](/images/2/2.4/Picture5.png)
Phần quan trọng nhất của đoạn script là chạy document AWSSupport-StartEC2RescueWorkflow (tham khảo https://docs.aws.amazon.com/systems-manager-automation-runbooks/latest/userguide/automation-awssupport-startec2rescueworkflow.html). Đoạn script khởi chạy instance mới (tên có từ “rescue”), gắn volume của instance trong issue vào và sửa bằng script (biến “script”) đã cấu hình trước. Tuy nhiên, khó hiểu là script lại chỉ chạy “echo Hello” thay vì sửa bất cứ cái gì. 

Kiểm tra logic qua CloudTrail:
![2.4](/images/2/2.4/Picture6.png)
Kiểm tra instance trên trang EC2. Lúc đầu, EC2Rescue instance được tạo:
![2.4](/images/2/2.4/Picture7.png)
Vài giây sau, bắt đầu shut down instance trong issue để gỡ volume:
![2.4](/images/2/2.4/Picture8.png)
Sau đó, instance cho experiment bị dừng hẳn:
![2.4](/images/2/2.4/Picture9.png)
Vài phút sau, instance mới shutdown, instance cho experiment được mở lại:
![2.4](/images/2/2.4/Picture10.png)
Montior NetworkIn qua CloudWatch:
![2.4](/images/2/2.4/Picture11.png)
Như vậy đã thực hiện phát hiện và khắc phục EC2 instance chậm độ trễ mạng thành công.
