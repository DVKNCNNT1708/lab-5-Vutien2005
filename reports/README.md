# Reports – Evidence của Lab 05

Thư mục này chứa bằng chứng và báo cáo từ quá trình kiểm thử Lab 05.

## Nội dung

- **test-results/**: Báo cáo kết quả từ Newman (HTML report, JSON report).
- **screenshots/**: Ảnh chụp màn hình từ Postman, Docker Desktop, terminal output.
- **logs/**: Log từ các container Docker.

## Hướng dẫn

1. Sau khi chạy `make compose-up`, lấy log từ các container:
   ```bash
   docker compose logs --all > logs/compose-all.log
   ```

2. Chạy Newman để test API:
   ```bash
   npm run test:compose
   ```

3. Lưu báo cáo HTML từ Newman vào `test-results/`.

4. Chụp ảnh từ Postman khi test endpoints và lưu vào `screenshots/`.
