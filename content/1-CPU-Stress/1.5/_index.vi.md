---
title : "Actions workflow to Inject CPU Stress & Remediate"
date : "`r Sys.Date()`"
weight : 5
chapter : false
pre : " <b> 1.5. </b> "
---

Tham khảo: .github/workflows/inject-cpu-stress.yml from the repo mostlycloudysky/aws-chaos-experiments.

Tất cả các phần liên quan Actions đều dùng instance mới trên us-east-2. 

#### 1. Prerequisites

a. Tạo instance
![1.5](/images/1/1.5/Picture1.png)
b. Notification
![1.5](/images/1/1.5/Picture2.png)
c. Setup secrets

Theo file yaml tham khảo,  cần thiết lập các secret cho action (căn cứ theo file inject-cpu-stress.json, ta sẽ biết được format của secret), gồm: 

- AWS_ACCESS_KEY_ID: ***
- AWS_SECRET_ACCESS_KEY: ***
- AWS_REGION: us-east-2 (đổi gió)
- AWS_ACCOUNT_ID: *****
- INSTANCE_ID: i-****
- IAM_ROLE:  <tên role sau ký tự “/” của role arn, vd: HaiAnh-FIS>
- CLOUDWATCH_ALARM_NAME: CPUUtilizationAlarm
- GITHUB_TOKEN: Không cần vì mỗi lần job chạy sẽ tự tạo token tạm thời

Bước 1: Vào setting của repo
![1.5](/images/1/1.5/Picture3.png)
Bước 2:  Bên các section bên trái, bên dưới “Secrets and variables”, chọn Actions
![1.5](/images/1/1.5/Picture4.png)
Bước 3: Chọn “New repository secret” và đặt tên secret và value.
![1.5](/images/1/1.5/Picture5.png)

Tuy nhiên sẽ bị lỗi vi phạm đặt tên do tên chứa ký tự đặc biệt. Tham khảo https://github.com/orgs/community/discussions/11941 thì thấy người ta không fix, vậy thì ta sẽ dùng HashiCorp Vault (được gợi ý bởi khóa GitHub Action trên KodeKloud) (từ bước 4 tới bước 14 là dùng HashiCorp, nhưng sau đó vẫn lỗi. Hóa ra là trong yaml có 1 ký tự đặc biệt không phải “_”, chỉ cần paste nó lên một chỗ không font như chỗ nhập URL thì sẽ ok).

Bước 4: Truy cập trang https://www.vaultproject.io/. Chọn “Try HCP Vault”.
![1.5](/images/1/1.5/Picture6.png)
Bước 5: Chọn Sign in với tài khoản GitHub
![1.5](/images/1/1.5/Picture7.png)
Bước 6: Nếu chưa có organization thì tạo 1 cái. Nếu có rồi thì chọn organization đó (hiện giới hạn 1 organization)
![1.5](/images/1/1.5/Picture8.png)
Bước 7: Chọn Projects -> View project
![1.5](/images/1/1.5/Picture9.png)
Bước 8: Kéo xuống hoặc tìm thanh bên trái, chọn Vault Secrets
![1.5](/images/1/1.5/Picture10.png)
Bước 9: Chọn Create first app
![1.5](/images/1/1.5/Picture11.png)
Bước 10: Đặt tên “AWS-FIS-secrets” và chọn Create App
![1.5](/images/1/1.5/Picture12.png)
Bước 11: Chọn Create new secret -> Static secret
![1.5](/images/1/1.5/Picture13.png)
Bước 12: Nhập Secret rồi Save
![1.5](/images/1/1.5/Picture14.png)
Bước 13: Chọn Integrations trên thanh bên trái -> GitHub Actions
![1.5](/images/1/1.5/Picture15.png)
Bước 14: Chọn Authorize HCP Vault Secrets
![1.5](/images/1/1.5/Picture16.png)
Các bước trên là làm cho biết cách lưu secret trên Vault. Quay lại với việc lưu secret trên GitHub Action.

Bước 15: Thêm các secret vào repo như bước 3
![1.5](/images/1/1.5/Picture17.png)
#### 2. Cài đặt

Đẩy thư mục fis-templates, thư mục scripts, thư mục .github của repo mẫu lên [repo cá nhân](https://github.com/PNg-HA/AWS-Chaos-Experiments)
![1.5](/images/1/1.5/Picture18.png)
#### 3. Chạy workflow action

Bước 1: Vào Actions của repo cá nhân, chọn “Inject CPU Stress Experiment” bên thanh bên trái -> Chọn Run workflow -> Chọn nút xanh tên “Run workflow”
![1.5](/images/1/1.5/Picture19.png)

Bước 2: Quan sát workflow
![1.5](/images/1/1.5/Picture20.png)
Quan sát các step:
![1.5](/images/1/1.5/Picture21.png)
Quan sát step tạo experiment template cho CPU stress, thấy tạo thành công:
![1.5](/images/1/1.5/Picture22.png)
Đợi 20 phút để tiến hành Experiment (Xem tại https://github.com/PNg-HA/AWS-Chaos-Experiments/actions/runs/10514440060/job/29132272786 để biết thêm chi tiết):
![1.5](/images/1/1.5/Picture23.png)
![1.5](/images/1/1.5/Picture24.png)
![1.5](/images/1/1.5/Picture25.png)
Quan sát mốc thời gian chạy experiment. Kiểm tra FIS console:
![1.5](/images/1/1.5/Picture26.png)
Kiểm tra alarm:
![1.5](/images/1/1.5/Picture27.png)
Kiểm ta CPUUtilizationAlarm:
![1.5](/images/1/1.5/Picture28.png)
Alarm thông báo tới mail:
![1.5](/images/1/1.5/Picture29.png)
Email CPU Utilization Alarm:
![1.5](/images/1/1.5/Picture30.png)
Mọi thứ đang diễn ra theo workflow! Khi xong step này, các step sau hoàn thành rất nhanh chóng và xong luôn workflow:
![1.5](/images/1/1.5/Picture31.png)
Kiểm tra workflow:
![1.5](/images/1/1.5/Picture32.png)
Không đúng flow mong đợi vì CPU stress trong 10 phút và khôi phục bình thường trước khi step chạy experiment hoàn thành.
![1.5](/images/1/1.5/Picture33.png)
Vì thế khi chạy step “Detect Issues”, alarm không trigger và sẽ không tạo issue trên repo cá nhân, nói cách khác là không có vấn đề nên không thể khắc phục:
![1.5](/images/1/1.5/Picture34.png)
Để khắc phục tạm thì tạo 1 file detect_cpu_stress_issues.yml (lấy nội dung file inject-cpu-stress.yml, xóa các step liên quan experiment, giữ lại step Detect Issues) cho workflow detect cpu stress và chạy thủ công.

Tiến hành chạy lại experiment:
![1.5](/images/1/1.5/Picture35.png)
Workflow inject-cpu-stress.yml có xử lý cho tình huống experiment template đã có sẵn, bằng cách lấy template id đã tồn tại lưu vào $GITHUB_OUTPUT cho step “Start FIS Experiment”:
![1.5](/images/1/1.5/Picture36.png)
Trong lúc chạy experiment, kích hoạt Detect CPU Stress workflow:
![1.5](/images/1/1.5/Picture37.png)
Kiểm tra chi tiết workflow, issue đã được tạo:
![1.5](/images/1/1.5/Picture38.png)
Vào issue vừa được tạo từ Action, dán nhãn thủ công “cpu-stress”:
![1.5](/images/1/1.5/Picture39.png)
Kết quả:
![1.5](/images/1/1.5/Picture40.png)
Thử Close issue xem workflow remediate có được kích hoạt không:
![1.5](/images/1/1.5/Picture41.png)
Kiểm tra Workflow remediate:
![1.5](/images/1/1.5/Picture42.png)
Chi tiết workflow:
![1.5](/images/1/1.5/Picture43.png)
Monitor CloudWatch CPU alarm:
![1.5](/images/1/1.5/Picture44.png)

Như vậy đã kích hoạt thành công workflow, dù không hoàn toàn tự động khi phải kích hoạt thủ công quá trình phát hiện CPU vượt ngưỡng.