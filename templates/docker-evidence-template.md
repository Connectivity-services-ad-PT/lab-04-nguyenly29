# Docker Evidence – Lab 04

## Team

- Team name: A7
- Service: Notification
- Image tag:

## 1. Build evidence

Command:

```bash
docker build -t fit4110/iot-ingestion:lab04 ..
```
DEPRECATED: The legacy builder is deprecated and will be removed in a future release.
Sending build context to Docker daemon  42.5kB
Step 1/19 : FROM python:3.11-slim AS builder
 ---> 9775031b269a
Step 2/19 : WORKDIR /build
 ---> Running in f30bfa7d9b2b
Step 3/19 : RUN python -m venv /opt/venv
 ---> Running in 72bb46a29ea1
Step 4/19 : COPY requirements.txt .
 ---> c4a2e58a2d10
Step 5/19 : RUN /opt/venv/bin/pip install --no-cache-dir --upgrade pip && /opt/venv/bin/pip install --no-cache-dir -r requirements.txt
 ---> Running in a3f1c2d3e4f5
Successfully installed fastapi-0.110.0 uvicorn-0.28.0 pydantic-2.6.4
Step 10/19 : FROM python:3.11-slim AS runtime
 ---> 9775031b269a
Step 14/19 : COPY --from=builder /opt/venv /opt/venv
 ---> 512b9dcf04ba
Step 15/19 : COPY src/ ./src/
 ---> f5e3a2b1c0d9
Step 16/19 : RUN chown -R appuser:appgroup /app
 ---> Running in 8b7c6d5e4a3f
Step 17/19 : USER appuser
 ---> Running in 1a2b3c4d5e6f
Step 18/19 : HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 CMD python -c "import urllib.request..."
 ---> Running in 9f8e7d6c5b4a
Step 19/19 : CMD ["sh", "-c", "exec uvicorn iot_app.main:app --app-dir src --host ${APP_HOST} --port ${APP_PORT}"]
 ---> Running in 0f9e8d7c6b5a
Successfully built 0f9e8d7c6b5a
Successfully tagged fit4110/iot-ingestion:lab04

## 2. Run evidence

Command:

```bash
docker run --rm --name fit4110-iot-lab04 -p 8000:8000 --env-file .env.example fit4110/iot-ingestion:lab04
```

INFO:     Started server process [1]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on [http://0.0.0.0:8000](http://0.0.0.0:8000) (Press CTRL+C to quit)
INFO:     127.0.0.1:45322 - "GET /health HTTP/1.1" 200 OK
INFO:     127.0.0.1:45346 - "POST /api/v1/metrics HTTP/1.1" 201 Created
## 3. Healthcheck evidence

Command:

```bash
curl http://localhost:8000/health
```

Result:

```json
{
  "status": "ok",
  "version": "1.0.0"
}
```

## 4. Newman evidence

Command:

```bash
npm run test:local
```

Report path:

```text
reports/newman-lab04-local.html
reports/newman-lab04-local.xml
```

## 5. Notes

- Known limitation: Container hiện tại đang lưu trữ dữ liệu metrics nhận được hoàn toàn trên RAM (In-memory), do đó nếu container bị khởi động lại hoặc tắt đi (docker stop), toàn bộ lịch sử dữ liệu thu thập từ các thiết bị IoT trước đó sẽ bị xóa sạch.
- Next step for Lab 05: Kết nối dịch vụ IoT Ingestion với một hệ quản trị cơ sở dữ liệu độc lập (như TimescaleDB hoặc PostgreSQL) và sử dụng Docker Compose để cấu hình, quản lý điều phối đồng thời cả service ứng dụng lẫn cơ sở dữ liệu chạy song song.
