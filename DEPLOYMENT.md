# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Phó Viết Tiến Anh |
| Mã học viên | 2A202601341 |
| Repo | https://github.com/photienanh/DAY12-2A202601341-PhoVietTienAnh |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://agent-production-835b.up.railway.app/ |
| Platform | Railway |
| Ngày deploy | 10/08/2026 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | Railway tự động cấp tại runtime |
| `AGENT_API_KEY` | ✅ | Secret đặt trong Railway Dashboard, không lưu trong repo |
| `REDIS_URL` | ✅ | Reference variable lấy từ Redis service của Railway |
| `RATE_LIMIT_PER_MINUTE` | ✅ | Cấu hình trên service `agent` |
| `MONTHLY_BUDGET_USD` | ✅ | Cấu hình trên service `agent` |
| `LOG_LEVEL` | ✅ | Cấu hình trên service `agent` |

## Lệnh Kiểm Tra

Thay `<URL>` bằng Public URL ở trên:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i <URL>/health

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i <URL>/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST <URL>/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST <URL>/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST <URL>/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Dán output của các lệnh trên vào đây:

```
curl -i https://agent-production-835b.up.railway.app/health
HTTP/2 200 
content-type: application/json
date: Mon, 10 Aug 2026 05:18:42 GMT
server: railway-hikari
x-railway-request-id: iqLm5ZoyTCqSJzjM6WHkDg
content-length: 57
x-hikari-trace: sin1.hs0s
x-railway-edge: sin1

{"status":"ok","service":"day12-agent","version":"1.0.0"}

curl -i https://agent-production-835b.up.railway.app/ready
HTTP/2 200 
content-type: application/json
date: Mon, 10 Aug 2026 05:19:04 GMT
server: railway-hikari
x-railway-request-id: Oy5VFyiHT4u6M-iWnpoFkQ
content-length: 31
x-hikari-trace: sin1.d1nj
x-railway-edge: sin1

{"status":"ready","redis":true}

curl -i -X POST https://agent-production-835b.up.railway.app/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'
HTTP/2 401 
content-type: application/json
date: Mon, 10 Aug 2026 05:20:15 GMT
server: railway-hikari
x-railway-request-id: wY-sRBqNRtaDsjn-npoFkQ
content-length: 39
x-hikari-trace: sin1.98a6
x-railway-edge: sin1

{"detail":"invalid or missing API key"}

curl -i -X POST https://agent-production-835b.up.railway.app/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'
HTTP/2 200 
content-type: application/json
date: Mon, 10 Aug 2026 05:21:09 GMT
server: railway-hikari
x-railway-request-id: XMXNLQ9jTJKY1yQK9I3ezw
content-length: 340
x-hikari-trace: sin1.98a6
x-railway-edge: sin1
vary: accept-encoding

{"answer":"Câu hỏi hay. Deploy là gì thường được giải quyết bằng cách chuẩn hóa môi trường chạy: cùng một image chạy giống nhau ở laptop và trên cloud. (Mình đang nhớ 20 lượt trao đổi trước đó.)","user_id":"sv-test","history_length":20,"cost_usd":9.345e-05,"tokens":{"in":443,"out":45}}

for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST https://agent-production-835b.up.railway.app/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done; echo
200 200 200 200 200 200 200 200 200 429 429 429 429 429 429 
```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl

---

## Nếu Dùng Phương Án Dự Phòng

Không đăng ký được tài khoản cloud? Vẫn nộp được bài, nhưng CP5 tối đa 60% điểm:

1. Đặt `LOCAL_FALLBACK=true` trong `.env`
2. Chạy `docker compose up -d` rồi kiểm tra `docker compose ps`
3. Chụp màn hình vào `screenshots/`
4. Chạy `pytest tests/test_cp5.py -v` — bộ test sẽ tự chuyển sang kiểm tra
   `http://localhost:8000`
5. Ghi rõ lý do không deploy được vào phần dưới đây:
