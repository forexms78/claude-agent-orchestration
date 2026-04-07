# Whalyx — 확정된 결정 사항

> 이미 결정된 사항. 매번 다시 설명 불필요.

## 아키텍처

| 영역 | 결정 사항 |
|------|----------|
| 백엔드 구조 | `backend/api/main.py` 단일 라우터. `backend/services/*.py` 서비스 분리 (도메인별 1파일) |
| 비동기 패턴 | 서비스 함수는 **sync**. `main.py`의 `_run(fn, *args)` 래퍼로 async 변환. `ThreadPoolExecutor(8)` |
| AI 호출 | `backend/utils/gemini.py`의 `call_gemini(prompt, system)` 통해서만. 직접 genai 호출 금지 |
| 백엔드 캐시 | Supabase `api_cache` 테이블. `db_get(key, ttl)` / `db_set(key, data)` 패턴 |
| 프론트 캐시 | localStorage + TTL. `CACHE_KEY` + `CACHE_TTL` 상수 → `useEffect`에서 처리 |
| API 연동 | `const API = process.env.NEXT_PUBLIC_API_URL`. URL 하드코딩 금지 |
| 컴포넌트 | 전부 `"use client"`. TypeScript interface 파일 상단 정의. `@/components/` 경로 |

## 코딩 컨벤션

**백엔드 (Python/FastAPI)**
- import 경로: `from backend.services.xxx import yyy` (`backend.` 접두사 필수)
- 새 서비스: `backend/services/새기능.py` 생성 → `main.py`에 import + `_run()` 래핑
- 응답: dict 반환 (Pydantic Response 모델 미사용)
- Gemini 프롬프트: 파일 상단에 `SYSTEM = """..."""` 상수로 분리

**프론트엔드 (Next.js/TypeScript)**
- 파일 최상단: `"use client";`
- 컴포넌트명·파일명: PascalCase `.tsx`
- 데이터 fetch: `useEffect` + `useState`. React Query 미사용
- 스타일: Tailwind CSS. MUI는 기존 것만 유지, 신규 추가 금지

## 명령어

| 작업 | 명령어 |
|------|--------|
| 백엔드 로컬 | `cd /Users/bellboi/code/war-inve && python3 -m uvicorn backend.api.main:app --host 0.0.0.0 --port 8000` |
| 백엔드 MOCK | `MOCK_MODE=true python3 -m uvicorn backend.api.main:app --host 0.0.0.0 --port 8000` |
| 프론트 로컬 | `cd /Users/bellboi/code/war-inve/frontend && npm run dev` |
| 프론트 빌드 | `cd /Users/bellboi/code/war-inve/frontend && npm run build` |
| 배포 | `git push origin main` → Railway(BE) / Vercel(FE) 자동 배포 |
| .env 위치 | `/Users/bellboi/code/war-inve/backend/.env` |

## 자주 하는 실수

| 실수 | 해결책 |
|------|--------|
| `gemini-2.0-flash` 사용 | 이 계정 limit:0 → 반드시 `gemini-2.5-flash` |
| yfinance 직접 호출 | 한국 IP 차단 → `financial.py`의 Yahoo REST 폴백 사용 |
| Gemini 429 직접 처리 | `call_gemini()`가 자동 재시도 → 서비스 레이어에 retry 추가 금지 |
| Railway 포트 하드코딩 | `$PORT` 환경변수 사용 |
| 서비스 함수를 async로 작성 | sync 작성 후 `main.py`에서 `_run()` 래핑 |
| 프론트 API URL 하드코딩 | `process.env.NEXT_PUBLIC_API_URL` 사용 |
| CORS 중복 설정 | `main.py`에 이미 `allow_origins=["*"]` → 중복 추가 금지 |
