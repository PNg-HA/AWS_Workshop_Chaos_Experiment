---
title : "Script phát hiện và giải quyết CPU Stress"
date : "`r Sys.Date()`"
weight : 4
chapter : false
pre : " <b> 1.4. </b> "
---

#### Part 1: Detect
##### 1. Yêu cầu

Cần có:
-	Python3.9 trở lên,
-	Tài khoản GitHub,
-	GitHub token, 
-	GitHub repo.

Cách tạo github token: 

Bước 1: Sau khi đăng nhập github website, truy cập github.com/settings/tokens -> Chọn Generate new token 
![1.4](/images/1/1.4/Picture1.png)
Bước 2: Chọn Generate new token (classic):
![1.4](/images/1/1.4/Picture2.png)
Bước 3: Cấu hình thời hạn và quyền hạn theo nhu cầu (mình cấu hình đầy đủ quyền) rồi chọn “Generate token”:
![1.4](/images/1/1.4/Picture3.png)
Bước 4: Lưu token này để dùng ở phần sau.

Cách tạo github repo: tham khảo https://docs.github.com/en/repositories/creating-and-managing-repositories/quickstart-for-repositories.

Bài này mình tạo 1 repo tên AWS-Chaos-Experiments trên tài khoản PNg-HA.
![1.4](/images/1/1.4/Picture4.png)
##### 2. Cài đặt
a. Chỉnh sửa script

Chỉnh sửa lại code file scripts/detect_issues.py trong repo tham khảo (mostlycloudysky/aws-chaos-experiments) để script dùng credential trong code, không khuyến khích làm theo vì không phải best practice cho security:

```python
DEFAULT_AWS_REGION = "ap-northeast-2"
DEFAULT_GITHUB_REPO = "PNg-HA/AWS-Chaos-Experiments" (syntax is user/repo)
DEFAULT_GITHUB_TOKEN = "ghp_*******"
DEFAULT_CLOUDWATCH_ALARM_NAME = "CPUUtilizationAlarm"
```

Script đã sửa:

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
b. Thực hiện phát hiện instance dừng:

Bước 1: Chạy script (song song chạy experiment cpu_stress)
![1.4](/images/1/1.4/Picture5.png)
Bước 2: Kiểm tra issue trên github repo cá nhân
![1.4](/images/1/1.4/Picture6.png)
Bước 3: Quan sát chi tiết issue
![1.4](/images/1/1.4/Picture7.png)
Instance ID trong issue giống EC instance id ở phần trên. Như vậy đã phát hiện instance bị CPU stress bằng script thành công.


#### Part 2: Remediate

Tham khảo: file scripts/remediate_stopped_instances.py của repo mostlycloudysky/aws-chaos-experiments.

##### 1. Chỉnh sửa script

Như phần 1, đặt credential vào script. Ngoài region, repo và token, script này cần:

```python
DEFAULT_GITHUB_ISSUE_NUMBER: số thứ tự (#) mà issue muốn giải quyết
```
Ngoài ra cần một biến môi trường cho biến issue_body, nhưng mình sẽ thêm trực tiếp giá trị này vào mặc định cho hàm getenv().
Script sau khi sửa:


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

##### 2. Thực hiện khắc phục issue

Chạy script khắc phục issue #4:
![1.4](/images/1/1.4/Picture8.png)
The script will stop the heavy job on the instance (and will not close the issue).

Đoạn script sẽ tắt job nặng trong instance (và cũng không đóng issue). 
![1.4](/images/1/1.4/Picture9.png)
Sau khi CPU vượt ngưỡng (lúc alarm), bắt đầu giảm dần (sau khi kích hoạt script khắc phục)

Kiểm tra instance qua CloudTrail event:
![1.4](/images/1/1.4/Picture10.png)
![1.4](/images/1/1.4/Picture11.png)
Kiểm tra “Run command” được chạy để khắc phục trong System Manager:
![1.4](/images/1/1.4/Picture12.png)
Thấy trùng thời gian (trên SSM và trên CloudTrail là UTC). Ngoài ra, khi dừng job thì experiment cũng không lỗi:
![1.4](/images/1/1.4/Picture13.png)
Như vậy đã thực hiện phát hiện và khắc phục Stress CPU thành công.
