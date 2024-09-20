---
title : "Dọn dẹp"
date : "`r Sys.Date()`"
weight : 4
chapter : false
pre : " <b> 4. </b> "
---

#### Xóa repository GitHub
Theo [trang GitHub Action Billing](https://docs.github.com/en/billing/managing-billing-for-github-actions/about-billing-for-github-actions), việc sử dụng miễn phí đối với các runner do GitHub cung cấp trong repository công khai, điều này áp dụng trong kịch bản của workshop này. Tuy nhiên, đối với các repository cá nhân, mỗi tài khoản GitHub nhận được một lượng phút và dung lượng miễn phí nhất định để sử dụng với các runner do GitHub cung cấp, tùy thuộc vào gói tài khoản. Tóm lại, nếu bạn không kích hoạt luồng GitHub Action, bạn sẽ không bị tính phí ngay cả khi sử dụng tất cả các workflow đã xây dựng trong workshop này.

1. Đi đến phần Cài đặt (Setting).
![4](/images/4/s1a.png)

1. Kéo xuống cuối để nhấn nút "Delete this repository".
![4](/images/4/s1b.png)

1. Một cửa sổ bật lên sẽ xuất hiện. Nhấn vào "I want to delete this repository".
![4](/images/4/s1c.png)

{{% notice info %}}
Workshop này được thực hiện vào tháng 8 năm 2024. Tại thời điểm này, tôi chọn GitHub Mobile để xác minh cho bất kỳ hành động nào trên GitHub console, bao gồm cả việc xóa repository GitHub.
{{% /notice %}}

#### Xóa tài nguyên AWS FIS
Theo [Giá của AWS](https://aws.amazon.com/fis/pricing/), AWS FIS chỉ tính phí cho các thử nghiệm đang chạy. Vì vậy, các mẫu (templates) mà bạn đã cấu hình sẽ không bị tính phí nếu bạn giữ chúng mà không chạy thử nghiệm.

1. Truy cập trang [AWS FIS Experiment templates](https://us-east-1.console.aws.amazon.com/fis/home?region=us-east-1#ExperimentTemplates).
2. Chọn các mẫu đã triển khai -> Nhấn vào "Actions".

![4](/images/4/s2.png)

3. Chọn "Delete experiment template".
![4](/images/4/s3.png)

4. Một cửa sổ bật lên sẽ xuất hiện. Nhập "delete" vào ô và nhấn nút "Delete experiment template".
![4](/images/4/s4.png)

#### Xóa AWS CloudWatch Alarms
1. Truy cập trang [AWS CloudWatch Alarm](https://us-east-1.console.aws.amazon.com/cloudwatch/home?region=us-east-1#alarmsV2:).

2. Chọn cảnh báo mà bạn đã tạo. Chọn nút "Action" -> Chọn "Delete".
![4](/images/4/s5.png)

3. Một cửa sổ bật lên sẽ xuất hiện. Nhấn nút "Delete".
![4](/images/4/s6.png)

#### Xóa AWS EC2 instance

1. Truy cập bảng điều khiển EC2.

2. Điều hướng đến mục Instances.

3. Chọn các instance liên quan đến phòng thí nghiệm.

4. Nhấn vào "Instance state".

5. Chọn "Terminate instance".
![4](/images/4/s7.png)
