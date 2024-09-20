---
title : "Actions workflow để tự động tiêm Network Latency và khắc phục"
date : "`r Sys.Date()`"
weight : 5
chapter : false
pre : " <b> 2.5. </b> "
---

Tham khảo: .github/workflows/ inject-network-latency.yml và .github/workflows/remediate-network-latency.yml của repo mostlycloudysky/aws-chaos-experiments. 


#### 1. Yêu cầu

Tài nguyên EC2 instance, SNS, github repo, CloudWatch Alarm đã tạo ở phần CPU Stress

Setup secrets

Theo file yaml tham khảo,  cần thiết lập các secret cho action, nhưng đã thêm ở phần CPU Stress nên chỉ cần thay đổi CLOUDWATCH_ALARM_NAME thành giá trị “NetworkInAlarm”. 

Bước 1: Vào setting của repo
![2.5](/images/2/2.5/Picture1.png)
Bước 2:  Bên các section bên trái, bên dưới “Secrets and variables”, chọn Actions
![2.5](/images/2/2.5/Picture2.png)
Bước 3: Chọn “New repository secret” và đặt tên secret và value.
![2.5](/images/2/2.5/Picture3.png)
Bước 4: Thêm các secret vào repo

#### 2. Cài đặt

Đẩy thư mục fis-templates, thư mục scripts, thư mục .github của repo mẫu lên repo cá nhân (đã thực hiện trong phần CPU Stress)
![2.5](/images/2/2.5/Picture4.png)
#### 3. Chạy workflow action
##### a. Chạy experiment

Bước 1: Vào Actions của repo cá nhân, chọn “Inject Network Latency Experiment” bên thanh bên trái -> Chọn Run workflow -> Chọn nút xanh tên “Run workflow”
![2.5](/images/2/2.5/Picture5.png)
Nhận được thông báo:
![2.5](/images/2/2.5/Picture6.png)
Bước 2: Quan sát workflow
![2.5](/images/2/2.5/Picture7.png)
Quan sát các step:
![2.5](/images/2/2.5/Picture8.png)
Quan sát step tạo experiment template cho Inject Network Latency, thấy tạo thành công:
![2.5](/images/2/2.5/Picture9.png)
Đợi vài phút để tiến hành experiment:
![2.5](/images/2/2.5/Picture10.png)
Quan sát mốc thời gian chạy experiment. Kiểm tra FIS console:
![2.5](/images/2/2.5/Picture11.png)
Kiểm tra chi tiết Experiment đang chạy:
![2.5](/images/2/2.5/Picture12.png)
Kiểm tra NetworkInAlarm:
![2.5](/images/2/2.5/Picture13.png)
Alarm thông báo tới mail:
![2.5](/images/2/2.5/Picture14.png)
Mail NetworkInAlarm:
![2.5](/images/2/2.5/Picture15.png)
Mọi thứ vẫn đang diễn ra theo workflow! Nhưng tiếp tục thì khi step này hoàn thành sẽ không còn alarm như phần test CPU Stress. 

##### b. Manually run detect

Vì thế mình tạo 1 file tên detect-network-latency.yml cho workflow để detect network latency (xóa các step tạo và chạy experiment, giữ lại step detect của file inject-network-latency.yml):

Chạy thủ công detect:
![2.5](/images/2/2.5/Picture16.png)
Phát hiện được alarm và tạo issue trên github repo cá nhân:
![2.5](/images/2/2.5/Picture17.png)
Quan sát chi tiết issue:
![2.5](/images/2/2.5/Picture18.png)

##### c. Khắc phục

Bước 1: Đóng issue bằng cách chọn “Close issue” để xem có trigger workflow remediate không:
![2.5](/images/2/2.5/Picture19.png)
Bước 2: Kiểm tra workflow remediate:
![2.5](/images/2/2.5/Picture20.png)
Tuy nhiên, remediate đã không chạy:
![2.5](/images/2/2.5/Picture21.png)
Dù vẫn còn alarm:
![2.5](/images/2/2.5/Picture22.png)
Xem lại workflow của inject-network-latency.yaml thì job chỉ chạy khi có nhãn network-latency:
![2.5](/images/2/2.5/Picture23.png)
Tuy nhiên trong tất cả các file đều không đề cập tới chuyện dán nhãn issue, hóa ra là phải dán nhãn thủ công:
![2.5](/images/2/2.5/Picture24.png)
Bước 3: Để tạo nhãn cho issue trên repo cá nhân, vào tag Issue chọn nút “Labels” kế nút xanh New issue:
![2.5](/images/2/2.5/Picture25.png)
Bước 4: Tạo các label “network-latency”, “remediate” và “cpu-stress”:
![2.5](/images/2/2.5/Picture26.png)
Bước 5: Để thêm label cho issue, chọn issue -> Chọn setting bên cạnh “Labels” -> Chọn nhãn network-latency:
![2.5](/images/2/2.5/Picture27.png)
Kết quả:
![2.5](/images/2/2.5/Picture28.png)
Bước 6: Thử khắc phục lại bằng cách đóng issue này thông qua việc chọn “Close issue”.

Workflow Remediate được kích hoạt:
![2.5](/images/2/2.5/Picture29.png)
Bước 7: Kiểm tra chi tiết workflow
![2.5](/images/2/2.5/Picture30.png)
Bước 8: Kiểm tra instance trên trang EC2. Lúc đầu, EC2Rescue instance được tạo:
![2.5](/images/2/2.5/Picture31.png)
Sau đó, instance cho experiment bị dừng hẳn:
![2.5](/images/2/2.5/Picture32.png)
Volume của Instance trong issue bị shut down đã được chuyển qua cho EC2Rescue instance:
![2.5](/images/2/2.5/Picture33.png)
Vài phút sau, instance mới shutdown, instance cho experiment được mở lại, chú ý volume đã được gắn trở lại:
![2.5](/images/2/2.5/Picture34.png)
Montior NetworkIn qua CloudWatch:
![2.5](/images/2/2.5/Picture35.png)

Như vậy, dù không mang tính tự động do phải chạy thủ công phần detect (do logic workflow step phát hiện sau step experiment) nhưng nhìn chung thì cũng khắc phục thành công.