---
title : "Thiết lập"
date : "`r Sys.Date()`"
weight : 2
chapter : false
pre : " <b> 1.2. </b> "
---



#### 1. Tạo template

Chuyển qua console vì tạo trên AWS CLI bị lỗi invalid json:
![1.2](/images/1/1.2/Picture1.png) 

Nhưng trước khi gặp lỗi invalid JSON mà gặp lỗi “An error occurred (UnauthorizedException) when calling the CreateExperimentTemplate operation: Unauthorized”, nghĩa là IAM role cho FIS thiếu permission

Bước 1: Truy cập trang Fault Injection Service trên console -> Trang Experiment templates -> Chọn Create experiment template -> Đặt tên
![1.2](/images/1/1.2/Picture2.png) 

Bước 2: Thêm target
![1.2](/images/1/1.2/Picture3.png) 

Bước 3: Thêm action

Document parameters: Từ file json.
![1.2](/images/1/1.2/Picture4.png) 

Bước 4: Kiểm tra lại
![1.2](/images/1/1.2/Picture5.png) 

Bước 5: Chọn IAM role (Role này có Trusted Entity là FIS)
![1.2](/images/1/1.2/Picture6.png) 

Bước 6: Mặc định những cái còn lại. Chọn Create experiment template
![1.2](/images/1/1.2/Picture7.png) 

Nhưng sẽ bị lỗi: 
![1.2](/images/1/1.2/Picture8.png) 

Bằng chút tinh tế, ta sẽ phát hiện AWS console tự động xử lý chuỗi string nhập vào, nhưng không “tinh tế” lắm, vì nó tự thêm escape char (ký tự ‘\’) cho mỗi nháy kép, dù mình đã thêm trước. Có thể vào Cloudtrail, tìm Event name CreateExperimentTemplate để quan sát:
![1.2](/images/1/1.2/Picture9.png) 

Quy trình debug: 
![1.2](/images/1/1.2/Picture10.png) 
-	error: lỗi hiển thị cho user
-	for input: input lỗi từ user nhập vào ô DocumentParameters
-	designed: input đúng từ tham số DocumentParameters của file json
-	Fixed input: input đúng mà user cần nhập vào ô DocumentParameters


Input đúng cho DocumentParameters là: 
```bash
{"commands":["for ($i=0; $i -lt 8; $i++) {Start-Job -ScriptBlock {while ($true) {}}}","Start-Sleep -Seconds 600","Get-Job | Stop-Job"]}
```

Kết quả:
![1.2](/images/1/1.2/Picture11.png)
![1.2](/images/1/1.2/Picture12.png) 

Mục “Export …” giải thích tại sao khi chạy AWS CLI trên PowerShell lại lỗi, đó là do phải chạy trên Linux, và phải chạy như phần export ghi, thay vì vắng tắt như trên repo: 


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

Tạo experiment template trên Linux:
![1.2](/images/1/1.2/Picture13.png) 

Full output:
![1.2](/images/1/1.2/Picture14.png) 

Check trên console:
![1.2](/images/1/1.2/Picture15.png) 
#### 2. Chạy template

Bước 1: Chỉnh qua “Start experiment CLI command” để xem cách chạy template trên CLI:
![1.2](/images/1/1.2/Picture16.png) 

Hoặc chạy trên console bằng vào trang Experiment templates, chọn “Start experiment” :
![1.2](/images/1/1.2/Picture17.png) 
![1.2](/images/1/1.2/Picture18.png) 

Bước 2: Nhập “start”
![1.2](/images/1/1.2/Picture19.png) 

Kết quả:
![1.2](/images/1/1.2/Picture20.png) 

Trong khi chạy, check CloudWatch:
![1.2](/images/1/1.2/Picture21.png) 

Check SSM:
![1.2](/images/1/1.2/Picture22.png) 
![1.2](/images/1/1.2/Picture23.png) 

Chờ vài phút:
![1.2](/images/1/1.2/Picture24.png) 

Khi đó, quay lại trang experiment:
![1.2](/images/1/1.2/Picture25.png) 

Kiểm tra Action chi tiết:
![1.2](/images/1/1.2/Picture26.png) 

Timeline:
![1.2](/images/1/1.2/Picture27.png) 