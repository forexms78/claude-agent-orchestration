---
name: deploy
description: OMS 팀 배포 워크플로우. Railway(백엔드) + Vercel(프론트) 배포
---

# OMS 배포 워크플로우

## 트리거

PO가 배포 요청 시 또는 PM이 스프린트 완료 후 배포 판단 시

## 백엔드 배포 (Railway)

### 1. 사전 확인
```bash
# 로컬 서버 정상 동작 확인
python -m uvicorn backend.api.main:app --port 8000
curl http://localhost:8000/
```

### 2. railway.json 생성 (없을 경우)
```json
{
  "build": { "builder": "NIXPACKS" },
  "deploy": {
    "startCommand": "python -m uvicorn backend.api.main:app --host 0.0.0.0 --port $PORT",
    "restartPolicyType": "ON_FAILURE"
  }
}
```

### 3. 환경변수 설정 (Railway 대시보드)
- GEMINI_API_KEY
- NEWS_API_KEY
- SUPABASE_URL
- SUPABASE_KEY

### 4. 배포
```bash
railway login
railway up
```

## 프론트 배포 (Vercel)

### 1. 환경변수 설정
- NEXT_PUBLIC_API_URL=https://[railway-domain]

### 2. 배포
```bash
cd frontend
vercel --prod
```

## 배포 후 확인

```bash
curl https://[railway-domain]/
curl -X POST https://[railway-domain]/analyze \
  -H "Content-Type: application/json" \
  -d '{"portfolio": ["NVDA"]}'
```

## 현재 배포 상태

- 백엔드: 미배포 (로컬 localhost:8000)
- 프론트: 미배포 (로컬 localhost:3000)
