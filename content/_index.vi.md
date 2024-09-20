---
title: "Thử nghiệm hỗn loạn AWS"
date: "`r Sys.Date()`"
weight: 1
chapter: false
---

# Thử nghiệm hỗn loạn trên AWS

#### Giới thiệu
Trong bối cảnh kỹ thuật số phát triển nhanh chóng ngày nay, khả năng phục hồi hệ thống là rất quan trọng đối với các doanh nghiệp ở mọi quy mô. Điều gì sẽ xảy ra khi hỗn loạn tấn công các phiên bản AWS EC2 Windows của bạn? Bạn có tự tin rằng chúng sẽ phục hồi, hay mọi thứ sẽ sụp đổ? Hội thảo này được thiết kế để trang bị cho bạn một số cách để thực hiện thử nghiệm hỗn loạn trên phiên bản EC2 của bạn, bao gồm: CPU-Stress, Network-latency và Dừng phiên bản.

#### AWS Fault Injection Simulator
AWS FIS là một dịch vụ được quản lý hoàn toàn để chạy các thử nghiệm kỹ thuật hỗn loạn nhằm kiểm tra và xác minh khả năng phục hồi của các ứng dụng của bạn. FIS cho phép bạn tiêm lỗi vào các tài nguyên tính toán, mạng và lưu trữ cơ bản. Điều này bao gồm dừng các phiên bản Amazon Elastic Compute Cloud (Amazon EC2), thêm độ trễ vào lưu lượng mạng và tạm dừng các hoạt động I/O trên các khối lưu trữ Amazon Elastic Block Store (Amazon EBS).

Nội dung:
1. [CPU-Stress]((1-CPU-Stress))
2. [Network-latency]((2-Network-latency))
3. [Dừng instance](3)
4. [Dọn dẹp](4)

#### Kiến trúc tổng thể
Trong hội thảo này, bạn sẽ tạo các mẫu thử nghiệm trong AWS FIS và thực hiện các thử nghiệm đó trên phiên bản EC2. 3 Cảnh báo CloudWatch được thiết lập (CPUUtilization, NetworkIn, NetworkOut) để thông báo cho bạn bất cứ khi nào thử nghiệm làm cho phiên bản EC2 bị quá tải. Ngoài ra, trong hội thảo này, có các phần mà bạn có thể sử dụng code và GitHub Action để tự động thực hiện, phát hiện và khắc phục các sự kiện hỗn loạn.
!arch

{{% notice info %}}
Hội thảo này được lấy cảm hứng từ kho lưu trữ tại https://github.com/mostlycloudysky/aws-chaos-experiments. Ngoài ra, tất cả mã trong hội thảo này đều có tại https://github.com/PNg-HA/AWS-Chaos-Experiments.
{{% /notice %}}
