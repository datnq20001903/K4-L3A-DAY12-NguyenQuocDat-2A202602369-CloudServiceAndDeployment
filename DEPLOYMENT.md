# Thông Tin Deploy — Checkpoint 5

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Nguyen Quoc Dat |
| Mã học viên | 2A202602369 |
| Repo | https://github.com/datnq20001903/K4-L3A-DAY12-NguyenQuocDat-2A202602369-CloudServiceAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-1sop.onrender.com |
| Platform | Render |
| Ngày deploy | 2026-09-28 |

## Biến Môi Trường Đã Set Trên Cloud

Chỉ ghi tên biến và nguồn giá trị, không ghi giá trị secret:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | Render tự cấp |
| `AGENT_API_KEY` | ✅ | Secret nhập trên Render Dashboard |
| `REDIS_URL` | ✅ | Tự nối từ service `day12-redis` |
| `RATE_LIMIT_PER_MINUTE` | ✅ | 10 |
| `MONTHLY_BUDGET_USD` | ✅ | 10.0 |
| `LOG_LEVEL` | ✅ | INFO |

## Cấu Hình Render

Blueprint trong `render.yaml` tạo hai service:

- Web service: `day12-agent`
- Redis service: `day12-redis`

Ứng dụng dùng `REDIS_URL` do Render cung cấp để kết nối nội bộ tới Redis.

## Kết Quả Kiểm Tra Public URL

### Liveness

Lệnh:

```text
GET https://day12-agent-1sop.onrender.com/health
```

Kết quả:

```text
HTTP 200
{"status":"ok","service":"day12-agent","version":"1.0.0"}
```

### Readiness

Lệnh:

```text
GET https://day12-agent-1sop.onrender.com/ready
```

Kết quả:

```text
HTTP 200
{"status":"ready","redis":true}
```

### Không Có API Key

Lệnh POST tới `/ask` không gửi header `X-API-Key`.

Kết quả:

```text
HTTP 401
{"detail":"invalid or missing API key"}
```

### Có API Key

Kết quả kiểm tra bằng API key đã cấu hình trên Render:

```text
HTTP 200
{"answer":"...","user_id":"cp5-test","history_length":0,"cost_usd":0.00002265,"tokens":{"in":3,"out":37}}
```

API key đã được kiểm tra nhưng không ghi vào repository.

## Ảnh Minh Chứng

Đặt các ảnh sau vào thư mục `screenshots/`:

- `screenshots/dashboard.png`: trang service `day12-agent` trên Render.
- `screenshots/health.png`: kết quả mở public URL `/health`.
