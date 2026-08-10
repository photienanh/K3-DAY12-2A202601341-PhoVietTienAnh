# Phiếu Phản Ánh — K3 Ngày 12

**Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
quan sát được khi chạy code — không sao chép đáp án của người khác.
>
Cách trả lời: thay dòng `*Câu trả lời của bạn*` bằng câu trả lời.
`grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
Họ và tên: Phó Viết Tiến Anh  Mã học viên: 2A202601341

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

*Ví dụ, khi deploy lên Railway mà quên khai báo `AGENT_APIKEY`, ứng dụng sẽ dừng ngay ở bước khởi động và log báo thiếu cấu hình. Nhờ vậy phiên bản lỗi không thể nhận traffic. Nếu có khóa mặc định `"changeme"`, service vẫn báo deploy thành công; người ngoài có thể đoán khóa này, gọi API và làm phát sinh chi phí trước khi phát hiện cấu hình sai.*

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

*Log JSON thu được: `{"event": "ask_completed", "level": "info", "timestamp": "2026-08-10T03:09:32.671808+00:00", "user_id": "sv01", "cost_usd": 0.0001}`. Từ các trường JSON, tôi có thể nhóm theo `user_id` rồi cộng `cost_usd` để tìm người dùng tiêu nhiều nhất; đồng thời lọc/đếm theo `event` và `level` để tạo dashboard hoặc cảnh báo khi tỷ lệ lỗi tăng. Chuỗi `print("đã trả lời xong")` không chứa dữ liệu có cấu trúc để thực hiện hai việc này.*

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage-t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | 1.17 GB |
| Multi-stage | 183 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

*Phần dung lượng chênh lệch chủ yếu đến từ image `python:3.11` đầy đủ chứa nhiều gói hệ điều hành và công cụ phục vụ build nhưng không cần khi chạy. Bản mới dùng `python:3.11-slim` dependency được cài ở builder và runtime chỉ nhận kết quả trong `/install` nên không mang môi trường build sang image cuối.*

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

*Khi chỉ sửa `app/main.py`, các layer base image, `WORKDIR`, `COPY requirements.txt`, `pip install` và copy dependency từ builder đều được lấy lại từ cache. Docker chỉ phải chạy lại từ layer `COPY app ./app` trở về sau; `COPY utils ./utils` vẫn có thể dùng cache nếu nội dung `utils` không đổi, sau đó image được export lại. Nếu đặt `COPY . .` trước `RUN pip install`, thay đổi một ký tự trong source sẽ làm layer copy đổi, kéo theo layer cài toàn bộ dependency bị mất cache và phải cài lại dù `requirements.txt` không đổi.*

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

*Giả sử endpoint Python có lỗ hổng cho phép thực thi lệnh, kẻ tấn công trước tiên chạy được lệnh trong container với quyền của tiến trình ứng dụng. Nếu tiến trình là root, họ có thể sửa mọi file trong container, đọc secret mà root truy cập được và lợi dụng thêm lỗi runtime/kernel hoặc socket/mount cấu hình sai để thoát container; khi đó quyền root làm mức ảnh hưởng trên host rất lớn. Lệnh `USER appuser` cắt chuỗi ngay sau bước thực thi lệnh: mã độc chỉ có UID 10001 không đặc quyền, nên bị giới hạn quyền file và thao tác hệ thống.*

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

*Người dùng có thể gửi tối đa 20 request trong 2 giây: gửi 10 request vào khoảng 10:00:59, sau đó gửi tiếp 10 request vào 10:01:01. Bộ đếm theo phút cố định coi hai nhóm thuộc hai phút khác nhau nên cả hai đều hợp lệ. Sliding window xét đúng 60 giây gần nhất, vì vậy ở 10:01:01 nó vẫn nhìn thấy 10 request lúc 10:00:59 và sẽ chặn nhóm tiếp theo.*

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

*Rate limit giới hạn tốc độ/số request trong 60 giây, còn cost guard giới hạn tổng số tiền của từng user trong cả tháng. Rate limit có thể cho qua một request thứ nhất trong phút nhưng cost guard phải chặn nếu user đã tiêu hết ngân sách tháng. Ngược lại, một user mới gần như chưa tốn tiền vẫn bị rate limit chặn ở request thứ 11 trong cùng cửa sổ 60 giây, dù cost guard còn rất nhiều ngân sách.*

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

*Redis mất kết nối làm cả ba container trả lỗi ở probe đã bị gộp. Orchestrator hiểu đó là lỗi liveness nên đánh dấu cả ba container unhealthy và lần lượt restart chúng, thay vì chỉ để load balancer tạm ngừng gửi traffic. Trong lúc Redis chưa hồi phục, container mới khởi động vẫn probe lỗi và tiếp tục bị restart. Khi Redis trở lại sau 30 giây, có thể chưa container nào sẵn sàng, khiến toàn service gián đoạn lâu hơn sự cố Redis ban đầu. Tách riêng `/health` giúp process vẫn sống, còn `/ready` 503 chỉ rút instance khỏi nhận traffic.*

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

*Với Redis dùng chung, mỗi câu hỏi thêm hai message nên các lần gọi liên tiếp sẽ quan sát `history_length` là 0, 2, 4, 6... dù request được chuyển qua các ontainer khác nhau. Nếu dùng dict Python, mỗi container chỉ thấy request đã đi vào chính nó. Qua load balancer, số liệu có thể nhảy như 0, 0, 0 rồi 2, 2, hoặc tăng/giảm thất thường theo container được chọn; restart container còn làm lịch sử của container đó trở về 0.*

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

*Sau khi service đã deploy và `/health` trả 200, endpoint `/ready` vẫn trả `503 {"status":"not ready","redis":false}`. Phản hồi này cho thấy process FastAPI vẫn sống nhưng dependency Redis chưa dùng được, nên tôi kiểm tra riêng Variables và các service trên Railway. Nguyên nhân là `REDIS_URL` của service `agent` chưa tham chiếu đúng Redis service. Tôi đặt `REDIS_URL` trên service `agent` bằng reference variable `${{Redis.REDIS_URL}}`, rồi deploy staged changes, đồng thời redeploy lại service Redis. Sau khi sửa theo cách trên, `/ready` trả `200 {"status":"ready","redis":true}`, chứng minh agent đã kết nối được Redis.*
