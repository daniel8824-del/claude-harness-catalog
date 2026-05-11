# Architecture — 6 레이어 + 3 Tier 풀스택 상세

> 본 문서는 [Daniel's Claude Harness Catalog](./index.html) §2-§3의 깊이 읽기 자료입니다.
> 강사 본인이 1년간 누적한 시스템의 각 레이어와 Tier 시스템을 메커니즘 단위로 자세히 풀어냅니다.

---

## 1. 아키텍처 진화 — 4 레이어에서 6 레이어로

초기 강사 환경은 **4 레이어 협력 모델**로 시작되었습니다.

```
Superpowers → OMC → gstack → autoresearch
```

이후 시스템 운영 중 두 가지 부족한 영역이 발견되었고, 외부 자산 2종을 추가 통합하면서 **6 레이어로 진화**했습니다.

| 진화 | 추가된 자산 | 부족했던 영역 |
|---|---|---|
| Layer 5 추가 | OMO (code-yeongyu/oh-my-openagent ★54K) | Planning 메타 — prometheus/metis/momus 미존재 |
| Layer 6 추가 | pro-workflow (rohitg00/pro-workflow ★2K) | Self-correcting memory — session 연속성 미흡 |

본 카탈로그는 **6 레이어 표준**으로 통일하며, 코어 4 + 확장 2의 구조로 명명합니다.

---

## 2. Layer 1 — Superpowers (언제 / 워크플로우 가드레일)

### 역할

사용자 발화를 읽고 **언제 어떤 워크플로우를 실행할지** 결정하는 가드레일. Claude가 맥락을 읽고 적시에 스킬을 추천하도록 안내합니다.

### 외부 출처

- GitHub: [obra/superpowers](https://github.com/obra/superpowers)
- Stars: ★167,056 (2026-04-25 기준)
- 라이센스: 원본 레포 정책 따름

### 핵심 스킬

| 스킬 | 시점 | 역할 |
|---|---|---|
| `brainstorming` | 기능·컴포넌트 생성 전 | 아이디어 흔들기 + 질문 도출 |
| `writing-plans` | 3+ 멀티파일 작업 | 구현 계획 작성 |
| `systematic-debugging` | 에러·테스트 실패 | 4단계 디버깅 구조화 |
| `verification-before-completion` | 완료 주장 직전 | 검증 강제 |
| `test-driven-development` | TDD 모드 | 테스트 우선 작성 |
| `executing-plans` | 플랜 실행 | 계획대로 진행 강제 |
| `finishing-a-development-branch` | 브랜치 마무리 | 마지막 정리 |
| `requesting-code-review` | 리뷰 요청 | 검토자 호출 |
| `receiving-code-review` | 리뷰 응답 | 피드백 반영 |
| `dispatching-parallel-agents` | 병렬 작업 | 멀티 에이전트 분산 |
| `subagent-driven-development` | 서브 에이전트 위임 | 역할 분담 |
| `using-git-worktrees` | git worktree | 격리 작업 |
| `using-superpowers` | 본 시스템 사용 안내 | 메타 가이드 |
| `writing-skills` | 스킬 작성 | 신규 스킬 패턴 |

### 동작 메커니즘

1. 사용자 발화 → embedding-suggester가 의미 매칭
2. 매칭 점수 ≥ 0.50 → `[SKILL SUGGESTION]` 태그 출력
3. Claude가 맥락 보고 자율 호출 결정
4. Tier 2 워크플로우로 진입

---

## 3. Layer 2 — OMC (누가/어떻게 / 에이전트 오케스트레이션)

### 역할

전문 에이전트 라우팅 + 모델 선택 + 상태 관리. 작업 성격에 맞는 에이전트를 호출하고 task_id 기반 세션 연속성을 유지합니다.

### 외부 출처

- GitHub: [Yeachan-Heo/oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode)
- Stars: ★31,195
- 강사 본인 환경에 v4.13.0 설치 (CLAUDE.md 명시)

### 전문 에이전트 명단

| 에이전트 | 모델 | 역할 |
|---|---|---|
| `executor` | Sonnet/Opus | 코드 작성 본편 |
| `architect` | Opus | 설계·디버깅 어드바이저 (READ-ONLY) |
| `planner` | Opus | 전략 플래닝 + 인터뷰 |
| `explore` | All | 코드베이스 검색 (READ-ONLY) |
| `code-reviewer` | Opus | 코드 리뷰 |
| `debugger` | Opus | 근본 원인 분석 |
| `verifier` | Opus | 검증 전략 |
| `security-reviewer` | Opus | OWASP Top 10 검사 |
| `test-engineer` | Opus | 테스트 전략 |
| `designer` | Sonnet | UI/UX 디자인 |
| `writer` | Haiku | README/API 문서 |
| `tracer` | Opus | 인과 추적 |
| `qa-tester` | All | tmux 기반 QA |
| `git-master` | All | 원자적 커밋·rebase |
| `document-specialist` | Sonnet | SDK 문서 전문 |
| `scientist` | Opus | 데이터 분석·연구 (READ-ONLY) |
| `simplifier` | Sonnet | 코드 단순화 |
| `analyst` | Opus | 사전 계획 컨설팅 (READ-ONLY) |
| `critic` | Opus | 다관점 비평 (READ-ONLY) |

각 에이전트는 task_id를 받아 세션 연속성을 유지하며, 모델 선택은 작업 복잡도에 따라 결정됩니다.

### OMC Tier-0 Force 키워드 8종

다음 키워드는 즉시 force 실행됩니다 (의도 추론 X):

| 키워드 | 스킬 | 용도 |
|---|---|---|
| `ralph` | ralph | 자율 반복 코드 작업 루프 |
| `autopilot` | autopilot | 워크플로우 자동 진행 |
| `ultrawork` | ultrawork | 병렬 작업 모드 |
| `ccg` | ccg | Claude+Codex+Gemini 3-AI 합의 |
| `ralplan` | ralplan | ralph + 플래닝 결합 |
| `deep-interview` | deep-interview | 깊은 인터뷰 워크플로우 |
| `ai-slop-cleaner` | ai-slop-cleaner | AI slop 코드 정리 |
| `cancel` | cancel | 모드 종료 |

### 실행 Mode 메시지 4종

| 메시지 | 모드 |
|---|---|
| `tdd` | Test-Driven Development 강제 |
| `code-review` | 코드 리뷰 모드 |
| `security-review` | 보안 리뷰 모드 |
| `ultrathink` | 깊은 추론 모드 |

### 상태 관리

OMC는 `.omc/state/` 디렉토리에 모드별 상태를 저장합니다.

```
.omc/state/
├── sessions/{sessionId}/
│   ├── autopilot-state.json
│   ├── ralph-state.json
│   ├── ultrawork-state.json
│   └── skill-active-state.json
└── (legacy files for backward compat)
```

세션별 격리로 다중 세션에서도 모드 충돌이 없습니다.

---

## 4. Layer 3 — gstack (안전 + 전문 도구)

### 역할

위험 명령 사전 차단 + 브라우저 QA + 성능 벤치마크 등 안전·검증 도구 제공.

### 외부 출처

- GitHub: [garrytan/gstack](https://github.com/garrytan/gstack)
- Stars: ★82,801

### 핵심 스킬

| 스킬 | 역할 |
|---|---|
| `careful` | 위험 명령 감지 (rm -rf · DROP TABLE · force-push 등) |
| `browse` | 헤드리스 브라우저 QA · 스크린샷 · 폼 검증 |
| `qa` | 브라우저 인터랙션 QA |
| `qa-only` | QA 전용 모드 |
| `connect-chrome` | 가시적 Chromium 실행 |
| `benchmark` | 성능 측정 |
| `canary` | 카나리 배포 검증 |
| `freeze` | 디렉토리 범위 제한 |
| `unfreeze` | freeze 해제 |
| `guard` | careful + freeze 통합 |
| `design-review` | 화면 가독성·UX 점검 |
| `design-consultation` | 디자인 상담 |
| `design-html` | HTML 디자인 생성 |
| `design-shotgun` | 다중 디자인 시안 |
| `visual-verdict` | 시각 판정 (OMC) |
| `devex-review` | DX 리뷰 |
| `ship` | 배포 워크플로우 |
| `land-and-deploy` | 머지·배포 |
| `health` | 헬스 체크 |
| `investigate` | 딥다이브 root cause |
| `document-release` | 릴리즈 노트 |
| `review` | 일반 리뷰 |
| `security-review` | 보안 리뷰 |

### 자동 발동 패턴 (Tier 1)

`careful`은 PreToolUse Bash inspect로 다음 패턴을 자동 감지합니다:

- `rm -rf` 류 destructive 명령
- `DROP TABLE` · `DELETE FROM ... WHERE 1=1` 등 SQL
- `git push --force` · `git reset --hard`
- `npm uninstall` 등 의존성 제거

감지 시 사용자 확인을 강제합니다.

---

## 5. Layer 4 — autoresearch (방법 / 자율 반복 프로토콜)

### 역할

명시 요청 시 목표 달성까지 반복 실행. `modify → verify → keep/discard → repeat` 사이클.

### 외부 출처

- GitHub: [karpathy/autoresearch](https://github.com/karpathy/autoresearch)
- Stars: ★76,395
- 작성: Andrej Karpathy

### 핵심 스킬 9종

| 스킬 | 용도 |
|---|---|
| `autoresearch` | 본편 자율 반복 |
| `autoresearch:plan` | 플래닝 단계 자율 반복 |
| `autoresearch:debug` | 디버깅 자율 반복 |
| `autoresearch:ship` | 배포 자율 반복 |
| `autoresearch:scenario` | 시나리오 검증 자율 반복 |
| `autoresearch:fix` | 수정 자율 반복 |
| `autoresearch:security` | 보안 검증 자율 반복 |
| `autoresearch:predict` | 예측 자율 반복 |
| `autoresearch:learn` | 학습 자율 반복 |

### 동작 프로토콜

```
[목표 정의]
   ↓
modify  ← 변경 시도
   ↓
verify  ← 검증
   ↓
keep/discard  ← 결과 유지 또는 폐기
   ↓
repeat  ← 목표 달성까지 반복
```

bounded iteration이라 max iteration 또는 시간 제한이 걸려 있어 무한 루프를 방지합니다.

### 강사 룰 (CLAUDE.md autoresearch protocol v2)

격차 진입 시 **Step 0 (시스템 정체성) · Step 1 (격차 출처) · Step 2 (메트릭 본질) 검증 강제**. 핸드오프·secondary 자료만으로 단정 금지. (영역 1·2 두 차례 오진단 사고 후 합의 2026-04-28)

---

## 6. Layer 5 — OMO (planning 메타 / Oh My OpenAgent)

### 역할 — **OpenCode Go 전용 병렬 오케스트레이션**

OMC가 Claude Code 전용 오케스트레이터라면, OMO는 OpenCode Go용 별도 오케스트레이터입니다. Kimi·DeepSeek 등 저렴 모델을 활용한 병렬 작업을 담당합니다.

### 외부 출처

- GitHub: [code-yeongyu/oh-my-openagent](https://github.com/code-yeongyu/oh-my-openagent)
- Stars: ★54,031
- 변형: 강사가 한국어 + OMC 환경 적응을 위해 풀 변환

### 핵심 스킬

| 스킬 | 역할 |
|---|---|
| `prometheus-planning` | 전략적 계획 수립 (5스텝 drill-down) |
| `metis-review` | 사전 계획 컨설팅 (계획 점검) |
| `momus-review` | 계획 검토 + 품질 게이트 (비판적 검토) |

### Planning 워크플로우

```
brainstorming (아이디어 흔들기)
   ↓
prometheus-planning (상세 플랜 작성)
   ↓
metis-review (빠진 제약·위험·질문 잡기)
   ↓
momus-review (비판적 마지막 검토)
   ↓
구현 진입
```

이 흐름은 강사가 1년간 운영하며 코드 짜기 전 합의 절차로 정착시킨 패턴입니다.

---

## 7. Layer 6 — pro-workflow (memory 보정)

### 역할

세션 간 지식 누적 + 자기 보정. session-handoff로 이어가고, replay-learnings로 복원, learn-rule로 룰 누적, vault-save로 영구 저장.

### 외부 출처

- GitHub: [rohitg00/pro-workflow](https://github.com/rohitg00/pro-workflow)
- Stars: ★2,007
- 변형: 강사가 Obsidian 4영역 통합 + 본문 한국어로 재작성

### 핵심 스킬

| 스킬 | 시점 | 역할 |
|---|---|---|
| `session-handoff` | 세션 종료 | 지금까지 한 일을 다음 세션용 메모로 저장 |
| `replay-learnings` | 세션 시작 (Tier 1 자동) | 과거 기억 복원 — 30_Claude 310+ entries |
| `learn-rule` | 사용자 교정 시 (Tier 1 자동) | 실수 → 한 줄 메모 → 다음 세션 자동 회상 |
| `vault-save` | 모든 핵심 저장 후 | Obsidian 30_Claude 6 영역에 영구 저장 |
| `learn` | 핵심 학습 발생 | 학습 사항 저장 |
| `retro` | 주간 회고 | 패턴 발견 + 다음 주 계획 |
| `harness` | 하네스 사용 안내 | 메타 가이드 |

### 강사 정정 (스킬 분류표 기준)

| 스킬 | tier | Daniel 변형 |
|---|---|---|
| `vault-save` | Custom by Daniel | QMD 후처리 + 4영역 규칙 |
| `session-handoff` | Adapted | Obsidian 4영역 통합 + 한국어 재작성 |
| `replay-learnings` | Adapted | 30_Claude 5폴더 + QMD 통합 |
| `learn-rule` | Adapted | QMD 중복 체크 + 30_Claude/02_Learnings 저장 룰 추가 |
| `learn` | Original (gstack) | 원본 |
| `retro` | Original (gstack) | 원본 |

---

## 8. 외부 CLI 도구 2종

### qmd (Quick Markdown Search)

- GitHub: [tobi/qmd](https://github.com/tobi/qmd)
- Stars: ★23,132
- 작성: Tobi Lütke (Shopify CEO)
- 역할: 로컬 마크다운 검색
- 강사 wrapper: `qmd-search` · `qmd-manage` 스킬

### feynman (Research Skill)

- GitHub: [getcompanion-ai/feynman](https://github.com/getcompanion-ai/feynman)
- Stars: ★5,846
- 역할: Pi CLI 학술 도구
- 강사 wrapper: `feynman` 스킬
- 포함 subskill 15종: literature-review · deep-research · paper-writing · peer-review · source-comparison · summarize · replicate · alpha-research 등

---

## 9. 3-Tier 스킬 실행 시스템 {#tiers}

### 🔴 Tier 1 — 완전 자동 (결정론적)

**조건:** 이벤트/명령 기반 — 의도 추론 없이 즉시 실행

| 시점 | 실행 |
|---|---|
| 세션 시작 | `replay-learnings` (SessionStart hook) |
| 위험 명령 감지 | `careful` (PreToolUse Bash inspect) |
| 컨텍스트 50% 도달 | `compact-guard` (시스템 이벤트) |
| 커밋 직전 | `smart-commit` (git commit 명령) |
| 사용자 교정 | `learn-rule` (명시 키워드 "기억해", "다음엔", "교훈") |

**Force 키워드 8 + Mode 메시지 4** (위 §3 OMC 참조)

### 🟡 Tier 2 — 맥락 추천 (Claude 자율 + Hook 추천)

**조건:** Hook이 `[SKILL SUGGESTION]` 태그로 추천 → Claude가 맥락 보고 자율 호출

**핵심 워크플로우 4 (위 §2 Superpowers 참조)** + 그 외 120+ 스킬.

**동작:**
- embedding-suggester가 도구함 스킬에 +0.03 boost
- threshold 0.50, topK 3
- Claude 맥락 자율 판단으로 호출

**거부된 추천은 같은 세션에서 반복 금지** (CLAUDE.md 명시).

### 🟢 Tier 3 — 폐기 (2026-05-01) → Tier 2 통합

기존 9종이 Tier 2로 흡수되었습니다:

`guard` · `retro` · `office-hours` · `investigate` · `health` · `codex` · `permission-tuner` · `freeze` · `unfreeze`

**폐기 이유:** 임베딩 기반 추천 시스템에서 manual/auto 구분이 description으로 자연 처리되어 별도 카테고리 유지의 효용이 사라짐. 사용자가 `/skillname`으로 직접 호출하면 언제든 실행 가능.

---

## 10. 매칭률 — 47 자연 발화 bench (2026-05-02 실측)

강사 환경에서 자연 발화 47건에 대한 매칭률:

- **Strict 87.2%** — 정확한 키워드·맥락 매칭
- **Lenient 97.4%** — 유사 의미 매칭 허용
- **FP 100% clean** — false positive 없음

Lenient 1건 회귀 — "UI 검토" 발화에 mafia-codereview-* over-match (옵션 A 수용 — 영문 only + 단일 벡터 임베딩 본질적 한계).

이행률은 **자동 100%가 아닌 80~90%**입니다 (정직 회수).

---

## 11. 관련 자료

- [hook-chain.md](./hook-chain.md) — Hook chain 5종 알고리즘 상세
- [token-saving.md](./token-saving.md) — 토큰 절감 12 기법
- [memory-vault.md](./memory-vault.md) — 메모리·볼트 시스템
- [search-system.md](./search-system.md) — QMD/GBrain/graphify
- [skill-ecosystem.md](./skill-ecosystem.md) — 134종 스킬 전체
- [issues.md](./issues.md) — 발견 이슈 4건
- [attribution.md](./attribution.md) — 외부 출처

---

**Last updated:** 2026-05-12 (v1)
