#  Delivery Checklist — Day 12 Lab Submission

> **Student Name:** Trần Long Hải  
> **Student ID:** 2A202600427  
> **Date:** 17/4/2026

---

##  Submission Requirements

Submit a **GitHub repository** containing:

### 1. Mission Answers (40 points)

---
# Day 12 Lab - Mission Answers

## Part 1: Localhost vs Production

### Exercise 1.1: Anti-patterns found
1.  **Hardcoded Secrets**: API Key (`OPENAI_API_KEY`) và Database URL được ghi trực tiếp vào code. Đây là lỗ hổng bảo mật cực lớn nếu đẩy code lên GitHub.
2.  **Fixed Port & Host**: Chạy cố định trên `localhost:8000`. Điều này khiến ứng dụng không thể chạy được trong Docker hoặc các nền tảng Cloud (vốn cần `0.0.0.0` và Port động).
3.  **Thiếu Health Check endpoints**: Không có các đường dẫn như `/health` hoặc `/ready`. Các hệ thống tự động (như Railway/Kubernetes) sẽ không biết khi nào app bị treo để khởi động lại.
4.  **Sử dụng `print()` thay vì logging**: Khi gặp các ký tự Unicode (tiếng Việt), lệnh `print()` trên Windows dễ gây lỗi `UnicodeEncodeError` làm crash app. Ngoài ra, log bằng print không có cấu trúc nên rất khó truy vết lỗi sau này.
5.  **Luôn bật Debug/Reload**: Cấu hình `reload=True` trong hàm chạy server. Chế độ này gây tốn tài nguyên và không an toàn khi chạy thực tế (Production).

### Exercise 1.3: Comparison table
| Feature | Develop | Production | Why Important? |
|---------|-------|----------|---------------------|
| **Config** | Ghi cứng trong code | Đọc từ biến môi trường (.env) | Giúp triển khai lên nhiều môi trường (Dev/Test/Prod) mà không cần sửa code. |
| **Health Check** | Không có | Có `/health` và `/ready` | Giúp hệ thống giám sát tự động phát hiện và phục hồi khi ứng dụng gặp sự cố. |
| **Logging** | Dùng `print()` (không cấu trúc) | Structured JSON logging | Giúp tìm kiếm và phân tích lỗi nhanh chóng bằng các công cụ quản lý log tập trung. |
| **Shutdown** | Dừng đột ngột | Graceful (xử lý tín hiệu SIGTERM) | Đảm bảo các yêu cầu khách đang xử lý được hoàn tất trước khi app đóng hẳn, tránh mất dữ liệu. |
| **Bảo mật** | Lộ lọt key trong code | Key tách rời khỏi mã nguồn | Bảo vệ tài khoản và chi phí sử dụng AI (OpenAI API key). |

---

## Part 2: Docker

### Exercise 2.1: Dockerfile questions
1. **Base image**: 
   - Bản Develop: `python:3.11` (Full distribution, đầy đủ công cụ nhưng nặng).
   - Bản Production: `python:3.11-slim` (Bản rút gọn, nhẹ và an toàn hơn).
2. **Working directory**: `/app` (Đường dẫn gốc bên trong container nơi code được lưu trữ).
3. **Tại sao COPY requirements.txt trước?**: Để tận dụng cơ chế **Layer Cache** của Docker. Nếu nội dung `requirements.txt` không thay đổi, Docker sẽ sử dụng lại kết quả của các bước `pip install` cũ, giúp giảm thời gian build từ vài phút xuống còn vài giây.
4. **CMD vs ENTRYPOINT**: 
   - `CMD`: Quy định lệnh mặc định khi container khởi động, có thể dễ dàng bị ghi đè (override) khi chạy lệnh `docker run`.
   - `ENTRYPOINT`: Quy định chương trình chính của container, khó bị ghi đè hơn, thường dùng để biến container thành một công cụ dòng lệnh chuyên dụng.

### Exercise 2.3: Image size comparison
- **Develop**: 1.66 GB (Sử dụng full base image và single-stage build)
- **Production**: 240 MB (Sử dụng slim image và multi-stage build để loại bỏ build tools)
- **Difference**: Giảm khoảng **85.5%** dung lượng.

## Part 3: Cloud Deployment

### Exercise 3.1: Railway deployment
- URL: https://firstdeploy-production-d0f6.up.railway.app/
- Screenshot: [View Screenshot](03-cloud-deployment/railway/screenshot.png)

## Part 4: API Security

### Exercise 4.1-4.3: Test results

1. **Test không có Key (Unauthorized)**: 
   - Lệnh: `curl.exe "http://localhost:8000/ask?question=hello" -X POST`
   - Kết quả: `401 Unauthorized`

2. **Test với Key đúng (Success)**:
   - Lệnh: `curl.exe "http://localhost:8000/ask?question=hello" -H "X-API-Key: my-secret-key" -X POST`
   - Kết quả: `200 OK` (Bản Basic)

3. **Nâng cao: JWT Authentication (Advanced)**:
   - Quy trình: Login nhận Token -> Gửi Token qua Authorization Header.
   - Kết quả: Login thành công, Token có cấu trúc `iat` (phát hành) và `exp` (hết hạn).
   - Lệnh test: `Invoke-RestMethod -Headers @{Authorization="Bearer <token>"} ...`
   - Kết quả: AI trả lời đúng câu hỏi bảo mật.

4. **Nâng cao: Rate Limiting**:
   - Cơ chế: Sliding Window (10 request/phút cho Role student).
   - Kết quả test: Sau 10 request liên tiếp, hệ thống trả về lỗi `429 Too Many Requests` và yêu cầu đợi đến khi Window reset.

### Exercise 4.4: Cost guard implementation
- **Cách tiếp cận**: Lab sử dụng lớp `CostGuard` (trong file `production/cost_guard.py`) để theo dõi chi phí sử dụng LLM theo thời gian thực.
- **Cơ chế hoạt động**:
  1. **Định giá**: Quy định giá tiền cho mỗi 1000 Tokens đầu vào và đầu ra.
  2. **Pre-check**: Trước khi gọi AI, hệ thống kiểm tra ngân sách người dùng. Nếu vượt quá hạn mức ($1.0/ngày), hệ thống trả về lỗi `402 Payment Required`.
  3. **Usage Update**: Sau khi AI phản hồi, hệ thống lấy số Token thực tế tiêu thụ để cập nhật vào bản ghi của người dùng đó.
  4. **Global Limit**: Có một hạn mức tổng cho toàn bộ ứng dụng để tránh việc API Key bị lạm dụng làm tăng hóa đơn bất ngờ.

## Part 5: Scaling & Reliability

### Exercise 5.1-5.5: Implementation notes
- **Khả năng mở rộng (Scaling)**: Hệ thống được triển khai với 3 Agent instances chạy song song. Sử dụng Nginx làm Load Balancer để điều hướng request theo thuật toán Round Robin.
- **Thiết kế Stateless**: Đã chuyển đổi Agent từ lưu trữ Session trong bộ nhớ thành lưu trữ trong Redis. Nhờ đó, người dùng có thể trò chuyện liên tục mà không bị mất context dù request được xử lý bởi các instance khác nhau.
- **Độ tin cậy (Reliability)**:
    - **Health Checks**: Cấu hình tự động kiểm tra trạng thái `/health` của Agent và `ping` của Redis. Nếu một instance gặp sự cố, Nginx sẽ tự động loại bỏ nó khỏi danh sách điều hướng.
    - **Graceful Shutdown**: Sử dụng `lifespan` context manager của FastAPI để đảm bảo Agent hoàn tất các request dở dang trước khi dừng hoàn toàn.
- **Kết quả thực tế**: Test script cho thấy request được phân phối đều qua 3 instance ID khác nhau nhưng Session History vẫn được bảo toàn 100%.
```

---

### 2. Full Source Code - Lab 06 Complete (60 points)

Your final production-ready agent with all files:

```
your-repo/
├── app/
│   ├── main.py              # Main application
│   ├── config.py            # Configuration
│   ├── auth.py              # Authentication
│   ├── rate_limiter.py      # Rate limiting
│   └── cost_guard.py        # Cost protection
├── utils/
│   └── mock_llm.py          # Mock LLM (provided)
├── Dockerfile               # Multi-stage build
├── docker-compose.yml       # Full stack
├── requirements.txt         # Dependencies
├── .env.example             # Environment template
├── .dockerignore            # Docker ignore
├── railway.toml             # Railway config (or render.yaml)
└── README.md                # Setup instructions
```

**Requirements:**
-  All code runs without errors
-  Multi-stage Dockerfile (image < 500 MB)
-  API key authentication
-  Rate limiting (10 req/min)
-  Cost guard ($10/month)
-  Health + readiness checks
-  Graceful shutdown
-  Stateless design (Redis)
-  No hardcoded secrets

---

### 3. Service Domain Link

Create a file `DEPLOYMENT.md` with your deployed service information:

```markdown
# Deployment Information

## Public URL
https://your-agent.railway.app

## Platform
Railway / Render / Cloud Run

## Test Commands

### Health Check
```bash
curl https://your-agent.railway.app/health
# Expected: {"status": "ok"}
```

### API Test (with authentication)
```bash
curl -X POST https://agent-service-production-495c.up.railway.app/ask \
  -H "X-API-Key: your-secret-key" \
  -H "Content-Type: application/json" \
  -d '{"user_id": "test", "question": "Hello"}'
```

## Environment Variables Set
- PORT
- REDIS_URL
- AGENT_API_KEY
- LOG_LEVEL

## Screenshots
- [Deployment dashboard](screenshots/dashboard.png)
- [Service running](screenshots/running.png)
- [Test results](screenshots/test.png)
```

##  Pre-Submission Checklist

- [ ] Repository is public (or instructor has access)
- [ ] `MISSION_ANSWERS.md` completed with all exercises
- [ ] `DEPLOYMENT.md` has working public URL
- [ ] All source code in `app/` directory
- [ ] `README.md` has clear setup instructions
- [ ] No `.env` file committed (only `.env.example`)
- [ ] No hardcoded secrets in code
- [ ] Public URL is accessible and working
- [ ] Screenshots included in `screenshots/` folder
- [ ] Repository has clear commit history

---

##  Self-Test

Before submitting, verify your deployment:

```bash
# 1. Health check
curl https://your-app.railway.app/health

# 2. Authentication required
curl https://your-app.railway.app/ask
# Should return 401

# 3. With API key works
curl -H "X-API-Key: YOUR_KEY" https://your-app.railway.app/ask \
  -X POST -d '{"user_id":"test","question":"Hello"}'
# Should return 200

# 4. Rate limiting
for i in {1..15}; do 
  curl -H "X-API-Key: YOUR_KEY" https://your-app.railway.app/ask \
    -X POST -d '{"user_id":"test","question":"test"}'; 
done
# Should eventually return 429
```

---

##  Submission

**Submit your GitHub repository URL:**

```
https://github.com/your-username/day12-agent-deployment
```

**Deadline:** 17/4/2026

---

##  Quick Tips

1.  Test your public URL from a different device
2.  Make sure repository is public or instructor has access
3.  Include screenshots of working deployment
4.  Write clear commit messages
5.  Test all commands in DEPLOYMENT.md work
6.  No secrets in code or commit history

---

##  Need Help?

- Check [TROUBLESHOOTING.md](TROUBLESHOOTING.md)
- Review [CODE_LAB.md](CODE_LAB.md)
- Ask in office hours
- Post in discussion forum

---

**Good luck! **
