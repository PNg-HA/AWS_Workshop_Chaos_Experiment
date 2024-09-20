---
title : "Cài đặt"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 2.2. </b> "
---

#### 1. Tạo Template

Bước 1: Truy cập trang Fault Injection Service trên console -> Trang Experiment templates -> Chọn Create experiment template -> Đặt tên
![2.2](/images/2/2.2/Picture1.png)
Bước 2: Add target
![2.2](/images/2/2.2/Picture2.png)
Bước 3: Add action

Document parameters: Từ file json.
![2.2](/images/2/2.2/Picture3.png)
Document parameters thực thi mục đích làm trễ 1 giây đối với lưu lượng mạng đi qua port 8080, nhờ 2 lệnh:

```bash
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=8080 connectaddress=127.0.0.1
```
- Giải thích: Tạo rule port proxy: forward traffic từ Internet vào địa chỉ loopback port 8080 

```bash
netsh interface portproxy set v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=8080 connectaddress=127.0.0.1 delay=1000
```

- Giải thích: Chỉnh sửa rule từ lệnh 1: độ trễ 1 giây cho traffic được forward

Bước 4: Kiểm tra lại
![2.2](/images/2/2.2/Picture4.png)
Bước 5: Chọn IAM role (Role này có Trusted Entity là FIS)
![2.2](/images/2/2.2/Picture5.png)
Bước 6: mặc định những cái còn lại. Chọn Create experiment template
![2.2](/images/2/2.2/Picture6.png)
Nhập “Create”:
![2.2](/images/2/2.2/Picture7.png)
Tạo thành công template:
![2.2](/images/2/2.2/Picture8.png)
Vào mục Export để xem cách cài template này trên CMD:
![2.2](/images/2/2.2/Picture9.png)
```bash
aws fis create-experiment-template ^
    --cli-input-json "{\"description\":\"Experiment to inject network latency on EC2 Windows instance\",\"targets\":{\"TargetInstances\":{\"resourceType\":\"aws:ec2:instance\",\"resourceArns\":[\"arn:aws:ec2:ap-northeast-2:590183822512:instance/i-06771d5fe9accdc18\"],\"selectionMode\":\"ALL\"}},\"actions\":{\"InjectNetworkLatency\":{\"actionId\":\"aws:ssm:send-command\",\"description\":\"Inject network latency on target EC2 instances\",\"parameters\":{\"documentArn\":\"arn:aws:ssm:ap-northeast-2::document/AWS-RunPowerShellScript\",\"documentParameters\":\"{\\\"commands\\\":[\\\"etsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=8080 connectaddress=127.0.0.1\\\", \\\"netsh interface portproxy set v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=8080 connectaddress=127.0.0.1 delay=1000\\\", \\\"Write-Output 'Introduced 1000ms network latency using netsh.'\\\"]}\",\"duration\":\"PT20M\"},\"targets\":{\"Instances\":\"TargetInstances\"}}},\"stopConditions\":[{\"source\":\"none\"}],\"roleArn\":\"arn:aws:iam::590183822512:role/HaiAnh-FIS\",\"tags\":{\"Name\":\"inject network latency\"},\"experimentOptions\":{\"accountTargeting\":\"single-account\",\"emptyTargetResolutionMode\":\"fail\"}}"
```

#### 2. Chạy template

Bước 1: Có thể chạy template trên CLI bằng lệnh

```bash
aws fis start-experiment --experiment-template-id <template-id>
```

Trong trường này cụ thể là:
```bash
aws fis start-experiment --experiment-template-id EXT9VzeDWJYFwcJN4
```

Hoặc chạy trên console bằng vào trang Experiment templates, chọn template vừa tạo -> “Start experiment” :
![2.2](/images/2/2.2/Picture10.png)
![2.2](/images/2/2.2/Picture11.png)
Bước 2: Nhập “start”
![2.2](/images/2/2.2/Picture12.png)
Trong khi chạy, check SSM:
![2.2](/images/2/2.2/Picture13.png)
Quay lại trang experiment của FIS:
![2.2](/images/2/2.2/Picture14.png)
Kiểm tra Action chi tiết:
![2.2](/images/2/2.2/Picture15.png)
Timeline:
![2.2](/images/2/2.2/Picture16.png)