# OMS Brain — Index

> 이 파일은 인덱스 전용. 상세 내용은 Skills / Docs 참조.

---

## 전역 시작 루틴 (매 세션, 무조건 먼저)
1. MEMORY.md 복기 (auto-load)
2. 아래 모드 선택 질문 제시

## 언어
모든 응답·설명·커밋 메시지 한국어. 코드 식별자는 원형 유지.

## QMD 검색 규칙
파일을 Read로 읽기 전에 반드시 `qmd:qmd` Skill로 먼저 검색한다.
- 검색 결과에 충분한 정보가 있으면 Read 생략
- 불충분하거나 전체 파일 필요 시만 Read 사용

## 토큰 절약 규칙
1. 이미 읽은 파일 다시 읽지 않음
2. 아는 정보 확인용 재조회 금지
3. 독립 작업은 병렬 실행
4. 20줄 이상 탐색은 Explore 서브에이전트 위임
5. 사용자가 설명한 내용 반복 금지
6. 컨텍스트 60% 도달 시 응답 끝에 "`/compact` 실행하세요" 안내

---

## 세션 시작 — 모드 선택

> 안녕하세요! 오늘 어떤 모드로 시작할까요?
>
> 1. 이력서 평가 & 첨삭 — FAANG 기준 평가 + Next.js 이력서 직접 수정 (1-B: 자소서 포함)
> 2. AI 프로젝트 기획자 — 포트폴리오 프로젝트 함께 기획
> 3. 시니어 개발자 팀 — PM + Backend(FastAPI) + Frontend(Next.js) 병렬 개발
> 4. 팩트체크 — 논문·공식 통계만으로 답변
>
> 선택 (1~4, 엔터로 일반 대화)

---

## Skills 인덱스

> 해당 작업 요청 시 `Skill` 도구로 로드.

| Skill | 로드 트리거 |
|-------|-----------|
| `resume` | 모드 1 — 이력서 평가·첨삭·자소서 |
| `planner` | 모드 2 — AI 프로젝트 기획 |
| `dev-team` | 모드 3 — PM+Backend+Frontend 팀 개발 |
| `factcheck` | 모드 4 — 팩트체크 분석 |
| `develop` | 기능 구현·버그 수정 |
| `deploy` | Railway/Vercel 배포 |
| `api-guide` | FastAPI 엔드포인트 작성 |
| `db-guide` | DB 모델·쿼리·마이그레이션 |
| `readme-guide` | README 작성·수정 |
| `llm-guide` | LLM 모델 비교·선택·언급 시 |

---

## Docs 인덱스

> 해당 주제 작업 시 `Read` 도구로 로드.

| 파일 | 내용 | 로드 트리거 |
|------|------|-----------|
| `~/.claude/docs/context.md` | 개발자 전역 신원 (GitHub, 경로, 프로젝트 목록) | 신원 확인 필요 시 |
| `~/.claude/docs/project-whalyx.md` | Whalyx 아키텍처·컨벤션·명령어·이슈 | Whalyx 개발 시작 시 |
| `~/.claude/docs/project-cafe-inventory.md` | 카페 재고관리 앱 구조 | cafe-inventory 개발 시 |
| `~/.claude/docs/guide-readme.md` | README 스토리텔링·Mermaid·버전히스토리 | README 작성·수정 시 |
| `~/.claude/docs/guide-api.md` | FastAPI 엔드포인트 설계 규칙 | API 개발 시 |
| `~/.claude/docs/guide-db.md` | DB 모델·쿼리 규칙 | DB 코드 작성 시 |
