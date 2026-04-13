# project-whalyx

> Whalyx 프로젝트 전체 컨텍스트. 개발 시작 시 로드.

## 프로젝트 정체

**AI 기반 글로벌 투자 인텔리전스 플랫폼 — 뉴스·주식·코인·부동산·자금흐름을 AI가 통합 분석**

- GitHub: https://github.com/forexms78/whalyx
- 로컬 경로: `/Users/bellboi/code/whalyx`
- 프론트 배포: https://whalyx.vercel.app (Vercel, project: frontend)
- 백엔드 배포: https://whalyx.onrender.com (Render Free Tier)
- `.env` 위치: `/Users/bellboi/code/whalyx/backend/.env`

## 기술 스택

| 영역 | 기술 |
|------|------|
| LLM | Gemini 2.5 Flash (google-genai SDK) |
| 백엔드 | Python FastAPI 0.115 (async + ThreadPoolExecutor) |
| 프론트 | Next.js 16, TypeScript |
| 주가 | Yahoo Finance REST (yfinance 한국 IP 차단) |
| 코인 | CoinGecko API v3 (5분 캐시, sparkline) |
| 뉴스 | Google News RSS + feedparser (실시간, API 키 없음) |
| 캐시 | Supabase PostgreSQL (서버 재시작 대응 DB 캐시) |
| 배포 | Render Free Tier (BE) + Vercel (FE) |

## 아키텍처 결정사항

| 영역 | 결정 사항 |
|------|----------|
| 백엔드 구조 | `backend/api/main.py` 단일 라우터. `backend/services/*.py` 서비스 분리 |
| 비동기 패턴 | 서비스 함수는 **sync**. `main.py`의 `_run(fn, *args)` 래퍼로 async 변환. `ThreadPoolExecutor(8)` |
| AI 호출 | `backend/utils/gemini.py`의 `call_gemini(prompt, system)` 통해서만. 직접 genai 호출 금지 |
| 백엔드 캐시 | Supabase `api_cache` 테이블. `db_get(key, ttl)` / `db_set(key, data)` 패턴 |
| 프론트 캐시 | localStorage + TTL. `CACHE_KEY` + `CACHE_TTL` 상수 → `useEffect`에서 처리 |
| API 연동 | `const API = process.env.NEXT_PUBLIC_API_URL`. URL 하드코딩 금지 |
| 컴포넌트 | 전부 `"use client"`. TypeScript interface 파일 상단 정의. `@/components/` 경로 |
| DB-Only | 유저 접속 시 외부 API 호출 완전 차단. 스케줄러가 외부 API → Supabase 저장, 화면은 Supabase에서만 읽음 |

## UI/UX 원칙

- **이모지 전면 금지** — 코드·UI·커밋 메시지·응답 어디에도 사용 금지. 기존 코드에 있으면 발견 즉시 제거

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
| 백엔드 로컬 | `cd /Users/bellboi/code/whalyx && python3 -m uvicorn backend.api.main:app --host 0.0.0.0 --port 8000` |
| 백엔드 MOCK | `MOCK_MODE=true python3 -m uvicorn backend.api.main:app --host 0.0.0.0 --port 8000` |
| 프론트 로컬 | `cd /Users/bellboi/code/whalyx/frontend && npm run dev` |
| 프론트 빌드 | `cd /Users/bellboi/code/whalyx/frontend && npm run build` |
| 배포 | `git push origin main` → Render(BE) / Vercel(FE) 자동 배포 |

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
| SUPABASE_KEY anon 키 사용 | 반드시 service_role 키 사용 (쓰기 권한 필요) |
| scheduler에서 sync 함수 직접 await | `_run_sync()` 래핑 후 asyncio.gather에 전달 |

## 알려진 이슈

- yfinance 직접 호출은 한국 IP 차단됨 → Yahoo Finance REST 폴백으로 처리
- Gemini 무료 티어 분당 요청 한도 → retries=1, 25초 타임아웃으로 방어. 503도 20초 대기 3회 재시도
- Render Free Tier: 15분 비활성 슬립 → UptimeRobot 5분 핑으로 방지

## 완료된 기능 (v2.1 기준)

- 마켓 드라이버: Gemini가 오늘 시장 핵심 뉴스 3선
- 오늘의 투자포인트: 뉴스 AI 분석 기반 (전 자산군: 주식/코인/부동산/광물/채권/한국)
- 8인 유명 투자자 포트폴리오 추적 (Buffett·Wood·Burry·Dalio 등)
- 핫 종목 TOP 12, 30일 가격 차트 + AI 인사이트
- 주식 상세 모달: 5기간 차트, 핵심 지표, 재무지표, 애널리스트 컨센서스
- 코인 탭: CoinGecko 10종 + 7일 스파크라인
- 부동산 탭: 서울 지표 + 뉴스
- 돈의 흐름: 금리·SPY·TLT·GLD·BTC 30d 수익률
- DB-Only 아키텍처 (v2.0): 스케줄러 → Supabase → 화면
- 스켈레톤 UI, 탭별 lazy loading, 다크/라이트 모드

## 최근 작업 이력

### v2.1 (2026-04-10)
- DB-Only 아키텍처 안정화
- Gemini 503 재시도 로직 추가 (`backend/utils/gemini.py`)
- scheduler `unhashable type: list` 버그 수정 — sync 함수는 `_run_sync()` 래핑 필수
- `warm_all_caches` DB 미스 시 즉시 실행 (첫 배포 빈 화면 수정)
- NewsAISection TypeError 크래시 수정 (null safety)
- 오늘의 투자포인트 뉴스 기반 전환 (S&P 500 FinBERT 파이프라인 제거)
- Supabase service role 키 교체 (anon → service_role)
