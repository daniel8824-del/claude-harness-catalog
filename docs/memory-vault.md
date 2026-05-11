---
layout: default
title: Memory & Vault
subtitle: 메모리 + 옵시디언 30_Claude 6 영역
---

# Memory & Vault — 메모리 + 옵시디언 30_Claude 6 영역

> 본 문서는 [Daniel's Claude Harness Catalog](./index.html) §8의 깊이 읽기 자료입니다.
> 사용자별 자동 메모리 + Obsidian 30_Claude 6 영역의 frontmatter·트리거·룰 누적 흐름을 자세히.

---

## 1. 두 가지 메모리 시스템

강사 환경에는 **두 가지 메모리 시스템**이 협력합니다.

```
┌─────────────────────────────────────────────────────────┐
│  1. ~/.claude/memory/          사용자별 자동 메모리       │
│     - Claude Code 내장 Auto Memory                       │
│     - MEMORY.md 인덱스 + 80+ 별도 .md                    │
│     - 매 세션 자동 로드                                  │
└─────────────────────────────────────────────────────────┘
                            +
┌─────────────────────────────────────────────────────────┐
│  2. /Zettelkasten/30_Claude/   Obsidian 볼트 6 영역      │
│     - 사용자 명시 저장 (session-handoff·learn-rule 등)  │
│     - 6 영역 분류 (Sessions, Learnings, Retros, ...)    │
│     - QMD 인덱싱 (988 파일)                              │
│     - 다음 세션 replay-learnings로 회상                  │
└─────────────────────────────────────────────────────────┘
```

---

## 2. ~/.claude/memory/ — 사용자별 자동 메모리

### 위치

```
~/.claude/projects/-home-daniel/memory/
```

### 구조

```
~/.claude/memory/
├── MEMORY.md                 200줄 인덱스 (매 세션 자동 로드)
└── 80+ 별도 .md              필요 시만 로드
    ├── user_*.md             사용자 프로필
    ├── feedback_*.md         피드백·룰
    ├── project_*.md          프로젝트 상태
    └── reference_*.md        외부 참조
```

### MEMORY.md 형식

200줄 인덱스만. 각 항목 한 줄 (≤150 chars). 본문은 별도 파일로 분리.

```markdown
# Memory Index

## User Profile
- Name: Daniel (daniel8824, 이성오)
- Windows: Claude Code for personal/company development
- Uses Obsidian for knowledge management (Zettelkasten vault)

## Work Rules - see [feedback_work_rules.md]
## Environment Setup - see [project_environment.md]
## OpenClaw Setup - see [openclaw.md]
## Zettelkasten Vault - see [project_zettelkasten.md]
## Yurisma 회사 정보 2026-05 변경 - see [feedback_yurisma_company_info_2026-05.md]
## "박제" 용어 금지 - see [feedback_no_seal_term.md]
...
```

### 4 메모리 타입

| 타입 | 용도 | 트리거 |
|---|---|---|
| **user** | 사용자 역할·목표·지식·선호 | 사용자 정보 학습 시 |
| **feedback** | 사용자 교정·승인 가이드 | "기억해", "다음엔", 사고 후 |
| **project** | 진행 중 작업·목표·이슈 | 누가/무엇을/왜/언제 학습 시 |
| **reference** | 외부 시스템 포인터 | 외부 시스템 언급 시 |

### feedback 메모리 구조

```markdown
---
name: 메모리 이름
description: 한 줄 설명 (관련성 판단용)
type: feedback
---

본문 (룰 자체)

**Why:** 이 룰이 도출된 이유 (사고·강한 선호)
**How to apply:** 언제·어디에 적용
```

### 자동 저장 트리거

| 상황 | 동작 |
|---|---|
| 사용자 교정 ("no not that", "don't", "stop doing X") | `learn-rule` 자동 저장 |
| 사용자 승인 ("yes exactly", "perfect, keep doing that") | 검증된 패턴으로 저장 |
| 새 사용자 정보 학습 | `user_*.md` 신규 |
| 외부 시스템 언급 | `reference_*.md` 신규 |

### 자동 회상 (매 세션 자동 로드)

`MEMORY.md`는 매 세션 자동 컨텍스트에 주입됩니다 (~200줄, ~12K 토큰).

본문은 필요 시만 Read — 인덱스에서 관련 메모리 발견 후 호출.

### 강사 현재 통계

- MEMORY.md 인덱스: 80+ 항목
- 별도 .md: 80+ 파일

---

## 3. Obsidian 30_Claude 볼트 6 영역

### 위치

```
/mnt/c/Users/daniel/Desktop/Zettelkasten/30_Claude/
```

### 6 영역 구조

```
30_Claude/
├── 00_Meta/              인덱스·로그·상태
│   ├── index.md         전체 항목 인덱스
│   └── log.md           이력 (상단 최신)
├── 01_Sessions/          session-handoff 저장
├── 02_Learnings/         learn-rule 교훈 (310+)
├── 03_Retros/            retro 주간 회고
├── 04_Plans/             writing-plans (prometheus 등)
├── 05_Research/          외부 리서치 결과
└── 06_Designs/           brainstorming 설계 결정

별도: 10_Raw/             원본 자료 (ingest, 988 파일)
```

### 파일명 형식

| 영역 | 형식 |
|---|---|
| 01_Sessions | `session-{YYYY-MM-DD}-{slug}.md` |
| 02_Learnings | `{YYYY-MM-DD}-{slug}.md` |
| 03_Retros | `retro-{YYYY-MM-DD}.md` |
| 04_Plans | `plan-{YYYY-MM-DD}-{slug}.md` |
| 05_Research | `research-{YYYY-MM-DD}-{slug}.md` |
| 06_Designs | `design-{YYYY-MM-DD}-{slug}.md` |
| 10_Raw | `{NN}_{type}/{YYYY-MM-DD}-{slug}.md` |

### Frontmatter 필수

```yaml
---
title: 제목
tags:
  - 개발/세션
  - 프로젝트/{name}
date: 2026-05-12
---
```

### 세션 노트 섹션 구조

```markdown
## Status
현재 상태 한 줄

## What's Done
완료된 작업 목록

## Key Decisions
의사결정 + 이유

## Resume Command
다음 세션 재개 명령
```

### 위키링크 규칙 (MANDATORY)

`[[링크]]`를 쓰기 전에 반드시 대상 파일 존재 여부 확인:

```bash
find /mnt/c/Users/daniel/Desktop/Zettelkasten -iname "파일명*.md" | head -1
```

- 존재하면 → `[[파일명]]` (확장자 제외)
- 존재하지 않으면 → **plain text로 작성** (링크 X)
- 스킬 파일은 위키 페이지 아니므로 절대 `[[스킬명]]` X → plain text + "(스킬)" 표기

이 규칙은 session-handoff, learn-rule, vault-save, brainstorming, writing-plans, retro 등 **볼트에 파일을 쓰는 모든 스킬에 동일 적용**.

### 저장 후 필수

```bash
# 1. 30_Claude/00_Meta/index.md 항목 추가
# 2. log.md 상단에 이력 추가
# 3. QMD 자동 인덱싱
qmd update && qmd embed
```

---

## 4. 자동 저장 트리거 (스킬별)

| 트리거 | 저장 경로 | 파일명 | 강제 |
|---|---|---|---|
| session-handoff 실행 | `01_Sessions/` | `session-{date}-{slug}.md` | **MANDATORY** |
| learn-rule 실행 | `02_Learnings/` | `{date}-{slug}.md` | **MANDATORY** |
| retro 실행 | `03_Retros/` | `retro-{date}.md` | **MANDATORY** |
| prometheus/writing-plans | `04_Plans/` | `plan-{date}-{slug}.md` | **MANDATORY** |
| 외부 리서치 3건+ | `05_Research/` | `research-{date}-{slug}.md` | 자동 |
| brainstorming 설계 도출 | `06_Designs/` | `design-{date}-{slug}.md` | 자동 |
| 중요 디버깅 (2시간+) | `02_Learnings/` | `{date}-{slug}.md` | 자동 |

---

## 5. 룰 누적 사이클 (복리 효과)

### 1주차

```
세션 1 → 작업 → 사고 → learn-rule "X 하지 마"
세션 2 → replay에서 X 룰 회상 → 안 함
세션 3 → 새 작업 → 새 사고 → learn-rule "Y 하지 마"
...
```

룰 0~3개 누적.

### 1개월

룰 10~15개 누적. AI가 회사 방식 정합 시작.

### 3개월

룰 30~50개 누적. AI가 회사처럼 일함.

### 6개월

룰 100+ 누적. **룰 자체가 자산**.

### 1년

룰 200+ 누적. 신입이 룰만 읽어도 회사 적응.

### 강사 현재 (1년+)

- MEMORY 인덱스: **80+**
- 30_Claude/02_Learnings: **310+** entries
- 30_Claude/01_Sessions: 다수 (세션별)
- 30_Claude/05_Research: 다수
- 30_Claude/06_Designs: 다수
- 10_Raw: **988 파일** (QMD 인덱싱)

---

## 6. 강사 실제 운영 룰 예시 (~/.claude/memory/MEMORY.md 발췌)

```
## 닫힌 기능은 다 여는 것이 원칙
   - disabled/readOnly/"준비 중"/mock-only 자의적 closure 금지
   - 사고 도출: 2026-04 Yurisma B-6 작업

## Yurisma fm 페이지 cream bg 거부
   - V.cream 섹션 bg 임의 추가 금지
   - 사용자 멘트: "그놈의 크림은 왜 그리 좋아하는거야"

## "박제" 용어 금지
   - 응답·commit·핸드오프·learn-rule 어디에도 사용 X
   - 대체: 기록·메모리 등록·캡처·자기 점검

## 알뜰카드 "포인트" 용어 금지
   - 적립 표시는 "원" 단위
   - UI label만 정합 (DB field 이름 유지)

## Yurisma 수정 후 즉시 Commit/Push
   - UI/DB/copy 수정 후 별도 OK 없이 즉시 commit+push
   - Daniel dogfooding cycle 효율
   - destructive 변경은 예외

## 오버액션 금지
   - 작업 chain 성공 확인되면 brief 보고만 + standby
   - 잔여 라벨/다른 modal/browser cache 류 자발적 추가 파헤치기 X

## Verify 직접 — 사용자에게 outsource 금지
   - URL fetch / deploy status / DB 등은 curl·gh api·convex run으로 직접
   - 단, dogfooding 화면 검수는 예외
```

각 룰은 별도 .md 파일로 본문 분리:
- `feedback_no_feature_closure.md`
- `feedback_no_default_cream_bg.md`
- `feedback_no_seal_term.md`
- `feedback_alddl_card_won_unit.md`
- `feedback_yurisma_immediate_commit_push.md`
- `feedback_no_over_action.md`
- `feedback_verify_directly.md`

---

## 7. 세션 라이프사이클 흐름

```
[세션 시작]
   ↓
replay-learnings 자동 실행
   - MEMORY.md 200줄 인덱스 자동 로드
   - 30_Claude/02_Learnings 310+ entries semantic 회상
   - per-session flag (한 세션 1회)
   ↓
[Tier 2 워크플로우]
   brainstorming → writing-plans → 구현
   ↓
[실수·교훈]
   사용자 교정 → learn-rule 자동 발동
   → ~/.claude/memory/feedback_*.md 작성
   → 30_Claude/02_Learnings/{date}-{slug}.md 저장
   → MEMORY.md 인덱스 한 줄 추가
   → qmd update && qmd embed (자동 인덱싱)
   ↓
[세션 종료]
   session-handoff 실행
   → 30_Claude/01_Sessions/session-{date}-{slug}.md 저장
   → "다음 세션 재개 명령" 포함
   ↓
[주간 회고]
   retro 실행
   → 30_Claude/03_Retros/retro-{date}.md
   → 일주일 패턴·인사이트·다음 주 계획
   ↓
[다음 세션]
   replay-learnings 자동 발동
   → 어제 session-handoff 자동 회상
   → 어제 learn-rule 자동 적용
   → "그대로 이어서" 가능
```

---

## 8. 메모리 vs 볼트 — 무엇을 어디에

### 메모리(~/.claude/memory)에 저장

- 사용자 정보 (user)
- 사용자 교정·승인 (feedback)
- 진행 중 작업·목표 (project)
- 외부 시스템 포인터 (reference)
- **매 세션 자동 회상 필요한 경량 정보**

### 볼트(30_Claude)에 저장

- 세션 핸드오프 (session-handoff)
- 학습된 교훈 본문 (learnings)
- 주간 회고 (retros)
- 플랜 문서 (plans)
- 외부 리서치 (research)
- 설계 결정 (designs)
- **세션 간 누적 + 검색 가능한 무거운 정보**

### 둘 다에 저장하는 경우

learn-rule은 양쪽 모두에 저장:
- 메모리: 인덱스 한 줄 (다음 세션 자동 회상)
- 볼트: 본문 풀 자료 (검색 가능 + 시간순 기록)

이 이중화로 **빠른 회상 + 깊은 검색** 모두 가능.

---

## 9. 무엇을 저장하지 말 것

CLAUDE.md `auto memory` 섹션 명시:

- **코드 패턴, 컨벤션, 아키텍처, 파일 경로, 프로젝트 구조** — 코드 읽으면 됨
- **git 히스토리, 최근 변경, who-changed-what** — git log/blame이 권위
- **디버깅 해결책, fix recipes** — 코드와 commit message가 답
- **CLAUDE.md에 이미 문서화된 것**
- **임시 작업 세부사항** — 작업 중 상태, 현재 대화 컨텍스트

사용자가 명시 요청해도 위 항목은 저장 X. 대신 **놀라운/비자명한 부분만** 저장.

---

## 10. 관련 자료

- [architecture.md](./architecture) — 6 레이어 + 3 Tier (Layer 6 pro-workflow)
- [hook-chain.md](./hook-chain) — replay-learnings-recall Hook
- [search-system.md](./search-system) — QMD 988 파일 검색
- [skill-ecosystem.md](./skill-ecosystem) — pro-workflow 스킬 명세

---

**Last updated:** 2026-05-12 (v1)
