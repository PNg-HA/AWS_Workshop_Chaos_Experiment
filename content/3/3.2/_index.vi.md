---
title : "Thiết lập"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 3.2. </b> "
---

#### 1. Tạo template

Bước 1: Truy cập trang Fault Injection Service trên console -> Trang Experiment templates -> Chọn Create experiment template -> Đặt tên
![3.2](/images/3/3.2/Picture1.png)
Bước 2: Add target
![3.2](/images/3/3.2/Picture2.png)
Bước 3: Add action

Bỏ chọn dòng “Completed if instances terminated” trong trường hợp này vì ta test khi instance đang chạy
![3.2](/images/3/3.2/Picture3.png)
Bước 4: Kiểm tra lại
![3.2](/images/3/3.2/Picture4.png)
Bước 5: Chọn IAM role (Role này có Trusted Entity là FIS)
![3.2](/images/3/3.2/Picture5.png)
Bước 6: mặc định những cái còn lại. Chọn Create experiment template
![3.2](/images/3/3.2/Picture6.png)
Nhập “Create”:
![3.2](/images/3/3.2/Picture7.png)
Tạo thành công template:
![3.2](/images/3/3.2/Picture8.png)
Vào mục Export để xem cách cài template này trên CMD:
![3.2](/images/3/3.2/Picture9.png)

```bash
aws fis create-experiment-template ^
    --cli-input-json "{\"description\":\"Experiment to stop EC2 instances\",\"targets\":{\"TargetInstances\":{\"resourceType\":\"aws:ec2:instance\",\"resourceArns\":[\"arn:aws:ec2:ap-northeast-2:590183822512:instance/i-06771d5fe9accdc18\"],\"selectionMode\":\"ALL\"}},\"actions\":{\"StopInstances\":{\"actionId\":\"aws:ec2:stop-instances\",\"description\":\"Stop target EC2 instance\",\"parameters\":{\"completeIfInstancesTerminated\":\"false\"},\"targets\":{\"Instances\":\"TargetInstances\"}}},\"stopConditions\":[{\"source\":\"none\"}],\"roleArn\":\"arn:aws:iam::590183822512:role/HaiAnh-FIS\",\"tags\":{\"Name\":\"stop-instance\"},\"experimentOptions\":{\"accountTargeting\":\"single-account\",\"emptyTargetResolutionMode\":\"fail\"}}"
```

#### 2. Chạy template

Bước 1: Có thể chạy template trên CLI bằng lệnh

```bash
aws fis start-experiment --experiment-template-id <template-id>
```
Trong trường này cụ thể là:

```bash
aws fis start-experiment --experiment-template-id EXTADVWqNsdEa5UBa
```

Hoặc chạy trên console bằng vào trang Experiment templates, chọn template vừa tạo -> “Start experiment”:
![3.2](/images/3/3.2/Picture10.png)

Chọn Start experiment:
![3.2](/images/3/3.2/Picture11.png)
Bước 2: Nhập “start”
![3.2](/images/3/3.2/Picture12.png)
Theo cấu hình thì FIS sẽ link tới API của EC2 để dừng instance nên không cần coi SSM.

Chạy trong vài giây. Khi xong, quay lại trang experiment:
![3.2](/images/3/3.2/Picture13.png)
Kiểm tra Action của experiment chi tiết:
![3.2](/images/3/3.2/Picture14.png)