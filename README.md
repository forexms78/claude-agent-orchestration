# Claude Agent Orchestration

> Claude CLI를 활용한 에이전트 오케스트레이션 — AI 엔지니어 취업을 위한 3가지 전문 에이전트 시스템

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

## 개요

이 프로젝트는 **Anthropic Claude CLI**를 오케스트레이션하여, 세션 시작 시 목적에 맞는 AI 에이전트 모드를 선택할 수 있는 시스템입니다.

단순히 Claude에게 질문하는 것을 넘어, **역할 기반 페르소나**를 설정함으로써 각 모드에 특화된 전문적인 피드백과 작업을 수행합니다. 현재 AI 엔지니어 취업 준비에 초점을 맞춰 세 가지 모드를 운영 중입니다.

---

## 작동 방식

Claude CLI를 실행하면 자동으로 모드 선택 프롬프트가 나타납니다:

```
안녕하세요! 오늘 어떤 모드로 시작할까요?

1. 채용 담당자 모드 — Google / Meta / Facebook 기준으로 냉철하게 이력서를 평가합니다
2. 이력서 첨삭가 모드 — Next.js 이력서 웹사이트를 직접 수정합니다
3. AI 프로젝트 기획자 모드 — AI 취업을 위한 포트폴리오 프로젝트를 함께 기획합니다

선택해주세요 (1, 2, 3 또는 일반 대화로 진행하려면 엔터)
```

사용자가 번호를 선택하면 Claude는 해당 페르소나로 전환되어 대화를 이어갑니다.

---

## 에이전트 모드

### 모드 1 — FAANG 채용 담당자

Google, Meta, Facebook의 시니어 엔지니어링 채용 담당자 역할로 이력서를 평가합니다.

**평가 기준:**
- 기술 스택의 깊이와 실무 적용 능력
- 프로젝트 임팩트의 수치화된 성과
- 문제 해결 능력과 사고 방식
- FAANG 레벨 기준 합격/불합격 판단

**특징:** 감정 없이 냉철하고 직접적으로 부족한 점을 먼저 지적합니다. 현실적인 조언을 통해 실제 채용 시장 기준을 파악할 수 있습니다.

---

### 모드 2 — 이력서 첨삭가 (Next.js 웹사이트 전문)

Next.js + Vercel로 배포된 이력서 웹사이트를 직접 수정하는 풀스택 개발자 역할입니다.

**이력서 사이트 스택:**
- Next.js 13 (App Router)
- TypeScript
- Tailwind CSS + MUI
- Zustand (상태 관리)
- Prisma + PostgreSQL
- Vercel 배포

**작업 범위:**
- 이력서 내용 개선 (문장, 키워드, 구조)
- FAANG 채용 기준 맞춤 콘텐츠 강화
- 실제 컴포넌트 코드 직접 수정

---

### 모드 3 — AI 프로젝트 기획자

AI 스타트업 공동창업자 + 실리콘밸리 PM 스타일로 AI 포트폴리오 프로젝트를 함께 기획합니다.

**기획 진행 방식:**
1. 문제 발굴 — 한국/글로벌 사회 문제 중 관심 영역 탐색 (또는 기획자가 3개 직접 제안)
2. 문제 검증 — 실재성, 규모, 해결 가능성 분석
3. 프로젝트 설계 — AI 에이전트 오케스트레이션 기반 솔루션 + MVP 범위 확정
4. 포트폴리오 가치 평가 — 채용 임팩트 점수 + 오케스트레이션 활용도 점수 (각 1-10)

**원칙:** 항상 문제 → 솔루션 순서. MVP는 2-4주 완성 가능한 범위로 제한.

---

## 기술 구현

이 시스템은 **`CLAUDE.md` 전역 설정 파일**을 통해 구현됩니다.

```
~/.claude/CLAUDE.md   ← Claude CLI가 모든 세션에서 참조하는 전역 설정
```

`CLAUDE.md`에 세션 시작 동작과 각 모드의 페르소나를 정의하면, Claude CLI는 새 대화를 시작할 때마다 이 설정을 자동으로 로드합니다. 별도의 코드나 서버 없이 **프롬프트 엔지니어링만으로** 에이전트 오케스트레이션을 구현한 것이 핵심입니다.

```
Claude CLI 시작
      │
      ▼
CLAUDE.md 로드
      │
      ▼
모드 선택 프롬프트 출력
      │
   ┌──┼──┐
   1  2  3
   │  │  │
   ▼  ▼  ▼
채용 첨삭 기획자
담당 가   페르소나
자
```

---

## 설치 및 사용

### 사전 요구사항
- [Claude Code (Claude CLI)](https://claude.ai/code) 설치

### 설정

1. 이 레포를 클론합니다:
```bash
git clone https://github.com/forexms78/claude-agent-orchestration.git
```

2. `.claude/CLAUDE.md` 내용을 전역 설정에 추가합니다:
```bash
cat .claude/CLAUDE.md >> ~/.claude/CLAUDE.md
```

3. Claude CLI를 실행합니다:
```bash
claude
```

세션 시작 시 자동으로 모드 선택 프롬프트가 나타납니다.

---

## 프로젝트 구조

```
claude-agent-orchestration/
├── .claude/
│   └── CLAUDE.md        # 에이전트 모드 설정 (전역 적용)
├── docs/
│   └── roadmap.md       # 로드맵 및 향후 계획
└── README.md
```

---

## 로드맵

향후 추가 예정인 기능들은 [docs/roadmap.md](./docs/roadmap.md)에서 확인할 수 있습니다.

주요 계획:
- 모의 기술 면접관 모드 (코딩 테스트 + 시스템 디자인)
- 커버레터 작성 도우미 모드
- 모드 간 컨텍스트 공유 (채용 담당자 피드백 → 첨삭가 전달)
- 다른 직군을 위한 범용 템플릿화

---

## 관련 프로젝트

- [이력서 웹사이트 (tailwind7)](https://github.com/forexms78/tailwind7) — Claude 첨삭가 모드가 직접 수정하는 Next.js 이력서 사이트

---

## License

MIT
