# RUN_COMPOSE.md – Hướng dẫn chạy Lab 05

Tài liệu này hướng dẫn người khác clone repo sạch và chạy lại stack Docker Compose của Lab 05.

---

## 0. Yêu cầu tiên quyết

- **Docker Desktop** hoặc **Docker Engine + Compose v2** (kiểm tra bằng `docker --version` và `docker compose --version`)
- **Git** để clone repo
- **Node.js 20.x LTS** và **npm** (nếu muốn chạy Newman, Prism, hoặc Spectral)
- **curl** hoặc **Postman** để test endpoint (tuỳ chọn)

---

## 1. Clone repo

```bash
git clone <repo-url>
cd FIT4110_lab05_docker_compose_readiness
```

---

## 2. Cài đặt dependencies

### Tuỳ chọn: Cài Node.js dependencies (để chạy Newman/Prism/Spectral)

```bash
npm install
```

### Kiểm tra npm scripts

```bash
npm run
```

Bạn sẽ thấy các scripts:
- `npm run lint:openapi` – Kiểm tra OpenAPI contract
- `npm run mock:iot` – Chạy Prism mock server
- `npm run wait:compose` – Chờ cho đến khi API sẵn sàng
- `npm run test:compose` – Chạy Newman test suite

---

## 3. Chuẩn bị file `.env`

```bash
# Copy .env.example sang .env
cp .env.example .env

# (Tuỳ chọn) Chỉnh sửa .env nếu cần
# Ví dụ: thay đổi APP_PORT, POSTGRES_USER, AUTH_TOKEN
```

Nội dung `.env.example`:

```env
# API configuration
APP_HOST=0.0.0.0
APP_PORT=8000
AUTH_TOKEN=local-dev-token
SERVICE_NAME=iot-ingestion
SERVICE_VERSION=0.5.0
ENV=local

# Database configuration
POSTGRES_USER=lab05
POSTGRES_PASSWORD=lab05pass
POSTGRES_DB=iotdb
```

---

## 4. Build & chạy stack Docker Compose

### Cách 1: Dùng Docker Compose CLI trực tiếp

```bash
# Build images và khởi động tất cả container trong nền
docker compose up -d --build

# Hoặc tách biệt: build trước, sau đó chạy
docker compose build
docker compose up -d
```

Lệnh trên sẽ tạo ba container:

- **`fit4110-db-lab05`** – PostgreSQL 15 (port 5432, nội bộ trên `team-internal`)
- **`fit4110-ai-lab05`** – AI service mock (port 9000, nội bộ)
- **`fit4110-api-lab05`** – API FastAPI (port 8000, exposed)

### Cách 2: Dùng Makefile

Nếu bạn có `make` hoặc `make.exe` (Windows):

```bash
# Build image API
make build

# Khởi động Compose stack
make compose-up

# Xem log real-time
make logs

# Tắt stack
make compose-down
```

---

## 5. Kiểm tra trạng thái các service

### Theo dõi log real-time

```bash
docker compose logs -f
```

### Kiểm tra health của từng service

```bash
# API health
curl http://localhost:8000/health

# AI service health
curl http://localhost:9000/health

# Database ready
docker exec -it fit4110-db-lab05 pg_isready -U lab05
```

Các response mong đợi:

```json
// API health (200)
{"status":"ok","service":"iot-ingestion","version":"0.5.0"}

// AI service health (200)
{"status":"ok","service":"ai-service","version":"0.5.0"}

// Database ready
accepting connections
```

### Kiểm tra container đang chạy

```bash
docker compose ps
```

---

## 6. Kiểm tra kết nối giữa các service

### API gọi AI service (từ trong container API)

```bash
docker exec fit4110-api-lab05 curl http://ai-service:9000/health
```

### API gọi Database

```bash
docker exec fit4110-api-lab05 python -c "import psycopg2; print('DB OK')"
```

---

## 7. Test API endpoint bằng curl

### Test POST /readings (tạo reading)

```bash
curl -X POST http://localhost:8000/readings \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer local-dev-token" \
  -d '{
    "device_id": "ESP32-LAB-A01",
    "metric": "temperature",
    "value": 31.5,
    "unit": "celsius",
    "timestamp": "2026-06-09T10:30:00+07:00"
  }'
```

Response mong đợi (201 Created):

```json
{
  "reading_id": "R-20260609-0001",
  "device_id": "ESP32-LAB-A01",
  "metric": "temperature",
  "accepted": true,
  "created_at": "2026-06-09T03:30:01+00:00"
}
```

### Test GET /readings/latest

```bash
curl -H "Authorization: Bearer local-dev-token" \
  http://localhost:8000/readings/latest?limit=5
```

Response mong đợi (200):

```json
{
  "items": [
    {
      "reading_id": "R-20260609-0001",
      "device_id": "ESP32-LAB-A01",
      "metric": "temperature",
      "value": 31.5,
      "unit": "celsius",
      "timestamp": "2026-06-09T10:30:00+07:00",
      "created_at": "2026-06-09T03:30:01+00:00"
    }
  ]
}
```

---

## 8. Test bằng Postman (GUI)

1. Mở **Postman Desktop** hoặc **Web**.
2. Import collection:
   - Click **Import** → Select `postman/collections/FIT4110_lab05_IoT_Ingestion.postman_collection.json`
3. Import environment:
   - Click **Environments** → **Import** → Select `postman/environments/FIT4110_lab05_local.postman_environment.json`
4. Chọn environment `FIT4110 Lab05 Local` từ dropdown góc phải
5. Mở collection `FIT4110 Lab05 IoT Ingestion` rồi test từng endpoint

---

## 9. Test bằng Newman (CLI)

```bash
# Chờ cho đến khi API sẵn sàng
npm run wait:compose

# Chạy toàn bộ test suite
npm run test:compose

# Hoặc chạy Newman trực tiếp với tùy chọn chi tiết
npx newman run postman/collections/FIT4110_lab05_IoT_Ingestion.postman_collection.json \
  -e postman/environments/FIT4110_lab05_local.postman_environment.json \
  -r cli,htmlextra \
  --reporter-htmlextra-export reports/test-results/newman-report.html
```

Báo cáo HTML sẽ được lưu vào `reports/test-results/newman-report.html`.

---

## 10. Lấy bằng chứng (Evidence)

### Log từ Docker Compose

```bash
# Lấy toàn bộ log
docker compose logs --all > reports/logs/compose-all.log

# Hoặc riêng từng service
docker compose logs api > reports/logs/api.log
docker compose logs db > reports/logs/db.log
docker compose logs ai-service > reports/logs/ai.log
```

### Chụp ảnh (screenshots)

Sử dụng Postman hoặc browser developer tools:

1. Test các endpoint và chụp ảnh response
2. Lưu vào `reports/screenshots/`
3. Ví dụ:
   - `health-check.png`
   - `create-reading-success.png`
   - `latest-readings.png`

### Docker stats (tài nguyên)

```bash
docker compose stats --no-stream > reports/logs/docker-stats.log
```

---

## 11. Tắt stack

```bash
# Tắt tất cả container nhưng giữ volume
docker compose down

# Hoặc tắt và xoá volume
docker compose down -v

# Dùng Makefile
make compose-down
```

---

## 12. Troubleshooting

### Container không start

```bash
# Kiểm tra log
docker compose logs api
docker compose logs db
docker compose logs ai-service

# Kiểm tra docker disk space
docker system df

# Xoá cache và rebuild
docker compose down -v
docker system prune -a
docker compose up -d --build
```

### Database không kết nối

```bash
# Kiểm tra DB health
docker exec -it fit4110-db-lab05 pg_isready -U lab05

# Xem DB log
docker compose logs db

# Xoá volume và rebuild
docker compose down -v
docker compose up -d --build
```

### Port conflict

Nếu port 8000, 9000 hoặc 5432 đã dùng:

1. Chỉnh sửa `.env`:
   ```env
   APP_PORT=8080
   ```

2. Hoặc tìm và tắt process đang chiếm port:
   ```bash
   # Windows
   netstat -ano | findstr :8000
   taskkill /PID <PID> /F

   # Linux/Mac
   lsof -i :8000
   kill -9 <PID>
   ```

### Test không pass

```bash
# Kiểm tra API có chạy không
curl http://localhost:8000/health

# Kiểm tra token trong .env có match với Postman environment không
grep AUTH_TOKEN .env

# Chạy test với verbose output
npx newman run postman/collections/FIT4110_lab05_IoT_Ingestion.postman_collection.json \
  -e postman/environments/FIT4110_lab05_local.postman_environment.json \
  -v
```

---

## Tóm tắt

| Bước | Lệnh |
|------|------|
| 1. Clone | `git clone <url> && cd FIT4110_lab05_docker_compose_readiness` |
| 2. Cài npm | `npm install` |
| 3. Chuẩn bị .env | `cp .env.example .env` |
| 4. Start stack | `docker compose up -d --build` (hoặc `make compose-up`) |
| 5. Kiểm tra health | `curl http://localhost:8000/health` |
| 6. Test API | `npm run wait:compose && npm run test:compose` |
| 7. Xem báo cáo | `reports/test-results/newman-report.html` |
| 8. Tắt stack | `docker compose down` |
