---
title : "Yêu cầu"
date : "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 1.1. </b> "
---



#### 1. Tạo EC2 instance OS Windows

Type và có IP public
![1.1](/images/1/1.1/Picture1.png) 

Security group của instance
![1.1](/images/1/1.1/Picture2.png) 

IAM instance profile:
![1.1](/images/1/1.1/Picture3.png) 

Kiểm tra xem Windows instance đã có SSM:
![1.1](/images/1/1.1/Picture4.png) 

#### 2. Cấu hình AWS CLI với quyền tối thiếu (ở đây dùng AdminFullAccess)

![1.1](/images/1/1.1/Picture5.png) 
#### 3. Cấu hình CloudWatch Alarm

Bước 1: Tạo SNS topic bằng cách chọn Create Topic -> Đặt tên ở ô Name -> Chọn Create topic.
![1.1](/images/1/1.1/Picture6.png) 
![1.1](/images/1/1.1/Picture7.png) 

Subscribe:
![1.1](/images/1/1.1/Picture8.png) 

Bước 2: Từ bước 2 sẽ dành cho CPU Usage. Chọn instance -> Action -> Monitor and troubleshoot -> Manage CloudWatch alarms
![1.1](/images/1/1.1/Picture9.png) 

Bước 3: Tạo CloudWatch alarm

Cách 1: Tạo CloudWatch alarm trên EC2 instance
![1.1](/images/1/1.1/Picture10.png) 

Set ngưỡng
![1.1](/images/1/1.1/Picture11.png) 

Cách 2: Dùng AWS CLI để tạo Alarm

Tạo Alarm cho CPU usage, NetworkIn và NetworkOut theo lệnh trên repo mostlycloudysky/aws-chaos-experiments, thay YourInstanceId và YourSNSTopicARN:

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

Kiểm tra trên console:
![1.1](/images/1/1.1/Picture13.png) 

Kiểm tra chi tiết CPU Utilization alarm:
![1.1](/images/1/1.1/Picture14.png) 
#### 4. Roles
Role cho FIS: 

Bước 1: Chọn Trusted entity là FIS, use case cho EC2
![1.1](/images/1/1.1/Picture15.png)
![1.1](/images/1/1.1/Picture16.png)