# Phiếu Phản Ánh — K4 Level 3A, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `> *Câu trả lời của bạn*` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Nguyễn Quốc Đạt  Mã học viên: 2A202602369

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Ví dụ khi deploy lên cloud nhưng quên cấu hình AGENT_API_KEY, ứng dụng sẽ dừng ngay và báo thiếu secret. Nhờ vậy tôi phát hiện lỗi cấu hình trước khi service nhận request. Nếu dùng mặc định "changeme", ứng dụng vẫn chạy với khóa yếu, người khác có thể truy cập API và gây phát sinh chi phí.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> *Ví dụ một dòng log:
{"event": "ask_completed", "level": "info", "timestamp": "2026-09-28T09:59:26.492456+00:00", "user_id": "sv01", "cost_usd": 0.0001}. 
Từ log JSON, tôi có thể:
1. Lọc và thống kê số request theo user_id, event hoặc khoảng thời gian.
2. Tính tổng chi phí theo người dùng hoặc theo ngày bằng công cụ phân tích log.
Với print("đã trả lời xong"), dữ liệu không có cấu trúc nên khó tìm kiếm và thống kê tự động.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | ... MB |
| Multi-stage | 271 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Bản 1-stage dùng python:3.11 đầy đủ, cài thư viện và giữ toàn bộ nội dung build trong cùng image. Bản multi-stage dùng python:3.11-slim; các dependency được cài ở stage builder, sau đó chỉ copy kết quả cần thiết sang stage runtime. Vì vậy runtime không chứa compiler, cache cài đặt và các file build thừa.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi chỉ sửa app/main.py, các layer cài dependency được dùng lại từ cache:
- COPY requirements.txt
- RUN pip install
- Stage runtime và phần copy dependency từ builder
Layer COPY app ./app phải chạy lại vì source trong thư mục app đã thay đổi. Các layer sau đó cũng được Docker xử lý lại nếu nội dung phụ thuộc vào layer này.
Nếu đặt COPY . . trước RUN pip install, chỉ cần sửa một dòng code cũng làm layer COPY . . thay đổi. Khi đó Docker phải chạy lại pip install, khiến build chậm hơn không cần thiết.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Nếu code có lỗ hổng và cho phép kẻ tấn công thực thi lệnh, lệnh đó ban đầu chạy trong container. Nếu process chạy bằng root, kẻ tấn công có toàn quyền trong container, có thể đọc secret, sửa file hoặc khai thác thêm lỗ hổng để tìm cách tác động đến host.
Lệnh USER appuser chuyển process sang user thường. Vì vậy nếu ứng dụng bị khai thác, quyền của kẻ tấn công bị giới hạn trong phạm vi user thường thay vì có quyền root. Điều này giảm đáng kể mức độ ảnh hưởng, dù không thay thế hoàn toàn các biện pháp bảo mật khác

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Người dùng có thể gửi tối đa 20 request trong 2 giây.
Ví dụ:
- Gửi 10 request vào giây 59 của phút hiện tại.
- Khi đồng hồ chuyển sang giây 00, bộ đếm được reset.
- Gửi tiếp 10 request ở đầu phút tiếp theo.
Như vậy trong khoảng thời gian rất ngắn quanh thời điểm chuyển phút, người dùng gửi được 20 request dù giới hạn là 10 request/phút. Sliding window khắc phục vấn đề này bằng cách xét đúng 60 giây gần nhất

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn số lượng request trong một khoảng thời gian. Cost guard giới hạn tổng chi phí của người dùng trong một tháng.
Ví dụ rate limit cho qua nhưng cost guard chặn: người dùng mới gửi 2 request trong một phút, nhưng ngân sách tháng gần hết và request tiếp theo vượt ngân sách. Khi đó rate limit cho phép nhưng cost guard trả 402.
Tình huống ngược lại: người dùng vẫn còn ngân sách tháng, nhưng gửi hơn 10 request trong một phút. Khi đó cost guard cho phép về mặt chi phí, nhưng rate limit trả 429

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu /health cũng kiểm tra Redis, khi Redis mất kết nối:
1. Cả 3 container đều bắt đầu trả trạng thái lỗi.
2. Load balancer hoặc orchestrator đánh dấu cả 3 container là unhealthy.
3. Các container bị loại khỏi danh sách nhận traffic hoặc bị khởi động lại.
4. Vì Redis vẫn đang mất kết nối, container mới tiếp tục fail health check.
5. Cụm có thể rơi vào vòng lặp restart và dịch vụ bị gián đoạn.
Cách đúng là /health chỉ kiểm tra process còn sống, còn /ready kiểm tra Redis. Khi Redis mất kết nối, /health vẫn trả 200, /ready trả lỗi để tạm ngừng nhận traffic nhưng không restart container không cần thiết.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Với Redis dùng chung, lịch sử được lưu tập trung nên các request đi vào những container khác nhau vẫn thấy cùng một lịch sử. Vì mỗi request thêm hai message, history_length có thể tăng theo kiểu 0, 2, 4, 6 cho các lần gọi tiếp theo.
Nếu dùng một dict Python trong từng container, mỗi container có một lịch sử riêng. Khi request chuyển sang container khác, history_length có thể quay lại 0 hoặc nhỏ hơn, ví dụ 0, 2, 0, 4. Kết quả sẽ không nhất quán khi scale nhiều instance

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Trong quá trình chuẩn bị deploy, tôi gặp lỗi:
remote: Repository not found.
fatal: repository not found
Nguyên nhân là remote trên máy đang trỏ tới tên repository khác với repository thật trên GitHub. URL cũ có phần tên không khớp với repository đã đổi tên.
Tôi kiểm tra lại URL repository trên GitHub, cập nhật remote bằng git remote set-url origin rồi chạy git ls-remote --heads origin để xác nhận kết nối. Sau đó push lại lên nhánh main và Render có thể đọc repository để deploy.

