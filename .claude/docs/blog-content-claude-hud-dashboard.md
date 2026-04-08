# 블로그 콘텐츠 원본 — Claude Code HUD 대시보드 커스터마이징

> 이 파일은 블로그 포스트용 원본 내용 정리. 포맷은 AI가 작성.

---

## 주제 / 핵심 메시지

Claude Code(AI 코딩 CLI)의 터미널 상태바 플러그인을 AI 자신이 직접 뜯어고쳐서 커스텀 대시보드로 만든 경험.
AI가 자신의 개발 환경을 AI 오케스트레이션 방식으로 개선한 메타적 사례.

---

## 배경

- Claude Code: Anthropic이 만든 터미널 기반 AI 코딩 어시스턴트 CLI
- claude-hud: Claude Code 터미널 하단에 실시간 상태를 표시하는 오픈소스 플러그인 (npm 패키지)
- 기존 화면: `[claude-sonnet-4-6] │ bellboi` / `Context ████░░░░ 45%` 형태의 2줄 텍스트
- 목표: 아래 이미지처럼 컬럼 구조의 대시보드 형태로 바꾸기

```
현재 모델              컨텍스트 사용률         토큰 수
claude-sonnet-4-6      ▓▓▓▓▓▓▓▓▓░░░░░ 62%      124,800 / 200,000
```

---

## 플러그인 구조 파악 과정

### 데이터 흐름
```
Claude Code → stdin JSON → TypeScript 렌더러 → stdout → 터미널 상태바
           ↘ transcript JSONL → tool/agent/todo 파싱
```

Claude Code는 약 300ms마다 플러그인을 실행. 매번 JSON을 stdin으로 전달:
- `model.display_name`: 현재 모델명
- `context_window.current_usage`: 토큰 사용량 (input + cache_creation + cache_read)
- `context_window.context_window_size`: 컨텍스트 윈도우 최대 크기
- `context_window.used_percentage`: 사용률 % (v2.1.6+)
- `rate_limits.five_hour.used_percentage`: 5시간 사용 한도 %
- `rate_limits.seven_day.used_percentage`: 주간 사용 한도 %
- `rate_limits.*.resets_at`: 리셋 Unix timestamp

### 파일 구조
```
dist/
├── index.js              # 진입점
├── config.js             # 설정 로드·검증
├── stdin.js              # stdin JSON 파싱
└── render/
    ├── index.js          # 레이아웃 분기·출력
    ├── colors.js         # ANSI 색상 헬퍼
    └── lines/
        ├── identity.js   # 컨텍스트 바 렌더링
        ├── project.js    # 모델명·프로젝트 렌더링
        └── usage.js      # 사용 한도 렌더링
```

---

## 핵심 개념 오해 해결

### 컨텍스트 윈도우 토큰 vs 5시간 사용 한도 토큰

처음에 "82,000 / 200,000으로 나오는데 5h 사용률이 83%면 숫자가 안 맞는다"는 의문이 생겼다.

**이유:** 완전히 다른 데이터 풀이다.

| 지표 | 의미 | 리셋 시점 | 형태 |
|------|------|-----------|------|
| 토큰 수 `82,000 / 200,000` | 현재 대화 1개의 토큰 | 새 대화 시작 시 | 실제 숫자 |
| 5h 사용률 `83%` | 지난 5시간 모든 대화 합산 | 5시간마다 | % 만 제공 |

`200,000`은 5시간 한도가 아닌 **컨텍스트 윈도우 크기** (한 대화에서 Claude가 기억할 수 있는 최대량).

**역산 불가 이유:** rate_limits API는 퍼센트만 내려온다. 총량을 모르니 역산 불가.
또한 역산해도 의미 없다. 5h 83%는 오늘 대화 3~4개의 합산인데, 현재 대화 토큰 82k로 나누면 틀린 값.

---

## 구현 내용

### 변경 파일 목록

| 파일 | 변경 내용 |
|------|-----------|
| `dist/config.js` | `validateLineLayout`에 `'dashboard'` 추가 |
| `dist/render/index.js` | `renderDashboardLines` import + `dashboard` 케이스 분기 |
| `dist/render/dashboard.js` | 신규 생성 — 대시보드 렌더러 전체 |

### 새 레이아웃 모드: `"dashboard"`

`config.json`에서 `"lineLayout": "dashboard"` 설정 시 활성화.

**출력 형태:**
```
현재 모델              컨텍스트 사용률         토큰 수                 5h 사용률       주간 사용률
claude-sonnet-4-6      ▓▓▓▓▓▓▓▓▓░░░░░ 62%      124,800 / 200,000       35% (2h)        18% (26h)
```

**구현 세부사항:**

1. **2줄 구조**: Row 1 = dim 라벨 행, Row 2 = 값 행
2. **컬럼 정렬**: 한국어는 2바이트 너비, 영문·숫자는 1바이트로 계산해 시각적 너비 기준 padding
3. **세그먼트 바**: `▓` (MEDIUM SHADE U+2593) 채움, `░` (LIGHT SHADE U+2591) 빈칸. 기존 `█/░` 대비 더 타일형 질감
4. **색상 임계값**: 0~69% 오렌지 → 70~84% 노랑 → 85%+ 빨강
5. **사용 정보 adaptive**: 5h/7d 데이터가 없으면 해당 컬럼 자동 숨김
6. **리셋 시간**: `resets_at` Unix timestamp → 남은 시간 `2h 30m` 형태 변환

### 빌드 없이 dist 직접 수정한 이유

플러그인 캐시 디렉토리(`~/.claude/plugins/cache/claude-hud/`)에 `node_modules`가 없어 `tsc` 실행 불가. dist JS 파일이 실제 실행 파일이므로 직접 수정. src 파일은 참조용으로 사용.

---

## AI 오케스트레이션 관점

이 작업 자체가 Claude Code 3-Agent 방식으로 진행됐다:

- **PO**: 이미지 한 장 제시 → "이렇게 만들어줘"
- **PM**: 플러그인 구조 분석, 데이터 소스 파악, 컨텍스트 윈도우 vs 5h 한도 개념 정리 후 PO 설명
- **Backend Dev**: dashboard.js 설계·구현, config.js 수정, render/index.js 분기 추가
- **Frontend Dev**: 해당 없음 (터미널 렌더러는 백엔드 성격)

PO는 "어떻게 구현할지" 전혀 지시하지 않았다. 단지 원하는 결과물 이미지를 제시했다.

---

## 기술적 의미

- AI가 자신의 개발 환경(터미널 상태바)을 AI 오케스트레이션 방식으로 직접 개선
- 외부 플러그인의 소스 코드를 분석·파악하고 빌드 파이프라인 없이 수정·검증
- 데이터 개념 오해(컨텍스트 윈도우 ≠ rate limit)를 코드 분석으로 정확히 설명
- Claude Code 생태계: 플러그인 구조, stdin 데이터, 렌더링 파이프라인 전체 파악

---

## 결과

- claude-hud v0.0.11 dist 파일 수정 (3개 변경, 1개 신규)
- `"lineLayout": "dashboard"` 설정으로 즉시 활성화
- 5시간 한도 있을 때: 5컬럼 표시 (모델·컨텍스트바·토큰수·5h·주간)
- 5시간 한도 없을 때: 3컬럼 표시 (모델·컨텍스트바·토큰수)
