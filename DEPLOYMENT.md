# Deployment Information — AI Agent Lab 12

## Public URL
https://agent-service-production-495c.up.railway.app

## Cloud Platform
Railway 

## Features Implemented
- [x] Docker Containerization (Multi-stage)
- [x] JWT / API Key Authentication
- [x] Rate Limiting (20 req/min)
- [x] Daily Cost Guard ($5.0/day)
- [x] Stateless Design (Redis-ready)
- [x] Health Checks & Graceful Shutdown

## Test Commands

### 1. API Test (PowerShell - Windows)
```powershell
# Lưu ý: Thay 'your-secret-key' bằng key thật của bạn
$headers = @{"X-API-Key" = "your-secret-key"}
$body = @{question = "Hello Agent"} | ConvertTo-Json
Invoke-RestMethod -Uri "https://agent-service-production-495c.up.railway.app/ask" -Method Post -Headers $headers -Body $body -ContentType "application/json"
```

## Environment Variables Set
- `PORT`: (Dynamic by Railway)
- `REDIS_URL`: redis://...
- `AGENT_API_KEY`: your-secret-key
- `LOG_LEVEL`: info

## Screenshots

### 1. Deployment Dashboard
[Deployment dashboard](06-lab-complete/screenshots/dashboard.png)

### 2. Service Running
[Service running](06-lab-complete/screenshots/running.png)

### 3. Test Results
[Test results](06-lab-complete/screenshots/test.png)