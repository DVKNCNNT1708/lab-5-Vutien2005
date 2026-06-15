# Readiness Checklist – Lab 05

Đây là danh sách kiểm tra (checklist) để đảm bảo stack Docker Compose của bạn đã sẵn sàng trước khi gửi bài. Hãy tick vào mỗi mục sau khi hoàn thành.

## Checklist Readiness 6 Điểm

- [ ] **1. Database Ready:** Container DB (`fit4110-db-lab05`) đã chạy và phản hồi `pg_isready`. 
  - Lệnh kiểm tra: `docker exec -it fit4110-db-lab05 pg_isready -U lab05`
  - Kết quả mong đợi: `accepting connections`
  - Hoặc: `docker compose exec db pg_isready -U lab05`

- [ ] **2. AI Service Ready:** Container AI service (`fit4110-ai-lab05`) trả về `200` cho endpoint `/health` và `/predict` hoạt động.
  - Lệnh kiểm tra: `curl http://localhost:9000/health`
  - Kết quả mong đợi: `{"status":"ok","service":"ai-service","version":"0.5.0"}`
  - Test /predict: `curl -X POST http://localhost:9000/predict`

- [ ] **3. API Ready:** Container API (`fit4110-api-lab05`) trả `200` cho `/health` và có thể tạo/lấy readings khi token hợp lệ.
  - Lệnh kiểm tra: `curl http://localhost:8000/health`
  - Kết quả mong đợi: `{"status":"ok","service":"iot-ingestion","version":"0.5.0"}`
  - Test với token: `curl -H "Authorization: Bearer local-dev-token" http://localhost:8000/readings/latest`

- [ ] **4. Environment Variables:** File `.env` đã được thiết lập đúng với các biến: `APP_PORT`, `POSTGRES_USER`, `AUTH_TOKEN`, `SERVICE_VERSION`. 
  - Kiểm tra: `cat .env`
  - Lưu ý: Không commit secret thật vào repo; lưu vào `.env` cục bộ, commit `.env.example`
  - Xác nhận trong Postman environment cũng phải match

- [ ] **5. Network & Connectivity:** 
  - Mạng `team-internal` hoạt động và các service kết nối được với nhau
  - API gọi được AI bằng hostname `ai-service` (không dùng localhost)
  - API gọi được Database bằng hostname `db`
  - Ports được map đúng: 8000 (API), 9000 (AI), 5432 (DB)
  - Lệnh kiểm tra: 
    ```bash
    docker exec fit4110-api-lab05 curl http://ai-service:9000/health
    docker compose ps
    ```

- [ ] **6. Docker Image Tags & Registry (Tuỳ chọn cho plug-a-thon):**
  - Image đã được build với tag `v0.1.0-<team-id>`
  - Đã push lên container registry (GHCR hoặc Docker Hub) – nếu là shared project
  - Xác nhận rằng tag xuất hiện trong registry public/private
  - Lệnh: `docker images | grep fit4110`

---

## Thêm Checklist – OpenAPI & Testing

- [ ] **OpenAPI Contract Valid:** File `contracts/iot-ingestion.openapi.yaml` hợp lệ
  - Lệnh: `npm run lint:openapi`
  - Hoặc dùng Prism mock: `npm run mock:iot` (mở tab khác, test qua `http://localhost:4010`)

- [ ] **Postman Collection & Environment Setup:**
  - Collection `postman/collections/FIT4110_lab05_IoT_Ingestion.postman_collection.json` tồn tại
  - Environment `postman/environments/FIT4110_lab05_local.postman_environment.json` có `baseUrl` và `authToken`
  - Có thể import vào Postman Desktop và chạy tất cả request

- [ ] **Newman Test Suite Pass:**
  - Lệnh: `npm run wait:compose && npm run test:compose`
  - Tất cả test case (health, create reading, get latest) đều pass
  - Báo cáo HTML lưu tại `reports/test-results/newman-report.html`

- [ ] **Evidence & Documentation:**
  - Log từ compose: `docker compose logs --all > reports/logs/compose-all.log`
  - Screenshots từ Postman test (successful requests)
  - Báo cáo Newman HTML
  - README.md, RUN_COMPOSE.md, Dockerfile đều hợp lệ

---

## Checklist – Code Quality

- [ ] **Dockerfile:**
  - Có `FROM python:3.11-slim`
  - Sử dụng multi-stage build (builder + runtime)
  - User non-root (`appuser`)
  - HEALTHCHECK đúng cú pháp
  - `EXPOSE 8000` khai báo port

- [ ] **docker-compose.yml:**
  - Ba service: `api`, `db`, `ai-service` định nghĩa đầy đủ
  - `depends_on` với `condition: service_healthy` đúng
  - Networks: `team-internal` và `class-net` (external)
  - Environment variables từ `.env`
  - Volumes cho database persistence

- [ ] **API main.py (src/iot_app/main.py):**
  - Đọc ENV từ `os.getenv()`
  - Endpoint `/health` trả `HealthResponse`
  - Endpoint `POST /readings` validate bearer token
  - Endpoint `GET /readings/latest` hỗ trợ query params
  - Exception handler cho validation error (422)

- [ ] **AI Service main.py (src/ai_service/main.py):**
  - Endpoint `/health` trả `{"status": "ok", ...}`
  - Endpoint `POST /predict` trả về `Prediction` object
  - Chạy trên port 9000

---

## Ghi chú thêm

Hãy ghi lại những vấn đề gặp phải hoặc điều chỉnh tại đây (nếu có):

```
- Mô tả vấn đề 1...
- Mô tả vấn đề 2...
- Thay đổi/điều chỉnh...
```

---

## Tóm tắt Trạng Thái

| Item | Trạng Thái | Ghi Chú |
|------|-----------|--------|
| Database ready | ✓ / ✗ | |
| AI service ready | ✓ / ✗ | |
| API ready | ✓ / ✗ | |
| Environment variables | ✓ / ✗ | |
| Network & Connectivity | ✓ / ✗ | |
| Docker image tags | ✓ / ✗ | |
| OpenAPI valid | ✓ / ✗ | |
| Postman collection | ✓ / ✗ | |
| Newman tests pass | ✓ / ✗ | |
| Evidence complete | ✓ / ✗ | |
