---
layout: default
title: Token Saving
subtitle: 토큰 절감 12 기법 + RTK 풀스택
---

# Token Saving — 12 기법 + RTK 풀스택

> 본 문서는 [Daniel's Claude Harness Catalog](./index.html) §7의 깊이 읽기 자료입니다.
> 강사 본인이 매일 굴리고 있는 12 기법의 메커니즘·실측·운영 룰을 자세히.

---

## 1. 합산 효과 (강사 환경 실측)

| 카테고리 | 누적 효과 |
|---|---|
| [A] Hook 자동 (4종) | 50-70% 절감 |
| [B] 구조 설계 (3종) | 컨텍스트 윈도우 효율 2~3배 |
| [C] 라우팅 (3종) | 단순 작업 비용 1/5 ~ 1/10 |
| [D] 워크플로우 (2종) | 검색 비용 거의 0 (사내 지식 쌓일수록) |

→ 12 기법 모두 적용 시 **연간 비용 절감 X배**. 사용자별로 다르지만 강사 환경에서는 RTK Hook 단독으로도 60-90% 절감.

---

## [A] Hook 기반 자동 절감 (사람 개입 0)

### 기법 1 — RTK (Rust Token Killer) Hook

**원리:** Bash hook이 dev 명령(`git`, `npm`, `ls` 등)을 자동으로 rtk 프록시로 우회. 명령 출력을 압축·필터링하여 토큰 폭감.

**메커니즘 상세:**

```
사용자/Claude가 명령 실행:
   git status

   ↓ Bash hook (PreToolUse) 인터셉트 (0 토큰 오버헤드)

실제 실행:
   rtk git status

   ↓

rtk가 git status 출력 변환:
   • 변경 없는 파일 제거
   • 긴 경로 단축 (~/.../ 축약)
   • ANSI 색상 코드 제거
   • 요약 형식 (예: "3 modified, 2 untracked" + 핵심만)

   ↓

Claude가 받는 출력:
   원본의 10-40% 크기
```

**프록시되는 명령 (강사 환경):**

| 명령 | 절감률 (강사 실측) |
|---|---|
| `git status` | ~70% |
| `git diff` | ~80% |
| `git log` | ~85% |
| `npm install` | ~90% |
| `npm test` | ~60% |
| `ls -la` (대규모 디렉토리) | ~75% |
| `find` (큰 결과) | ~80% |

**메타 명령 (rtk 직접 사용):**

```bash
rtk --version         # 설치 확인 (rtk X.Y.Z)
rtk gain              # 누적 절감 통계
rtk gain --history    # 명령별 사용 이력 + 절감
rtk discover          # Claude Code 히스토리에서 놓친 기회 분석
rtk proxy <cmd>       # 필터링 없이 원본 실행 (디버깅)
which rtk             # 정확한 바이너리 경로
```

**Name collision 주의:**

`reachingforthejack/rtk` (Rust Type Kit)이 동시 설치된 경우 충돌 가능. `rtk gain` 명령으로 구분.

**강사 운영 룰:**

- "git을 직접 안 쓰고 rtk 자동 프록시 신뢰" — Claude/Daniel 모두 git 직접 호출 X
- `rtk gain` 매주 확인 — 절감 추이 모니터링
- 새 dev 명령 추가 시 `rtk discover`로 후보 발견

---

### 기법 2 — compact-guard Hook

**원리:** 컨텍스트 50% 도달 시 자동 압축 타이밍 제어. auto-compact의 정보 손실 회피.

**메커니즘:**

```
Claude Code 기본 동작:
   컨텍스트 90~95% 도달 → auto-compact 자동 발동
   → 정보 손실 큼 (오래된 메시지 마구 잘림)

compact-guard 적용 후:
   컨텍스트 50% 도달 → 직접 /compact 타이밍 제어
   → 중요 메시지 유지하며 압축
   → 정보 손실 최소화
```

**효과:**

- 자동 8% 압축 vs 50% 수동 압축 — 토큰 윈도우 효율 ~2배
- 작업 중 컨텍스트 폭발 방지
- 사용자가 직접 `/compact` 안 쳐도 됨

**강사 룰:** "auto-compact 타이밍 나쁘다 — 적절한 시점에 직접 compact 실행"

---

### 기법 3 — embedding daemon

**원리:** bge-m3-q8_0 1024dim 로컬 모델 재사용. 매 발화마다 모델 로딩 X.

**메커니즘:**

```
Unix socket: /tmp/skill-embedding.sock

첫 실행 (cold start):
   1. GGUF 모델 메모리 로딩 (~70-130s)
   2. daemon process 시작
   3. socket bind

이후 실행 (warm):
   1. socket 연결 (50-300ms)
   2. 텍스트 → 1024dim 벡터 변환
   3. 결과 반환
```

**자동 시작:** `~/.bashrc`가 셸 시작 시 socket 부재면 daemon spawn (idempotent).

**효과:**

- 매 발화마다 모델 로딩 = ~70s × N
- daemon 재사용 = ~300ms × N
- → 응답 시간 200배 단축 = 사용자 대기 시간 절감 → 효율적 작업 → 토큰 효율

---

### 기법 4 — 프롬프트 캐싱

**원리:** Anthropic 자동 활성화. 같은 컨텍스트 반복 시 input의 30-50%가 cache_read로 처리.

**메커니즘:**

```
입력 토큰 유형:
   input_tokens         사용자 프롬프트 + 시스템
   cache_write_tokens   캐시 기록 (약간 비쌈, 1회만)
   cache_read_tokens    캐시 읽기 (저렴, 5분 TTL)

같은 시스템 프롬프트 + 도구 정의 반복 시:
   첫 호출: cache_write_tokens 발생 (1.25x 가격)
   이후 5분 내: cache_read_tokens 사용 (0.1x 가격)
```

**TTL:** 5분 (Anthropic 기본). 5분 지나면 cache 만료 → 다시 cache_write.

**강사 룰:**

- 같은 작업 연속 시 5분 내 처리 — 캐시 활용
- 5분 넘어가는 작업은 새 세션으로 분리하지 말고 한 세션에서 압축
- "5분 timeout 인지 → 효율적 작업 패턴"

**실측:** 강사 환경 input의 약 40%가 cache_read로 처리됨.

---

## [B] 구조 설계로 절감 (한 번 만들면 영구)

### 기법 5 — 3계층 CLAUDE.md

**원리:** 메인 컨텍스트는 작게, 도메인 진입 시만 추가 로드.

**구조:**

```
~/.claude/CLAUDE.md                개인 글로벌
  • 모든 프로젝트 공통 룰
  • OMC 설정, MCP 기본
  • 본인 작업 스타일 (존댓말, 톤, 출력 규칙)
  • 매 세션 자동 로드 (강사 환경: ~12K 토큰)

/project/CLAUDE.md                 프로젝트 단위
  • 프로젝트 구조, 명령어, 도메인 용어
  • 팀 규칙 (커밋 컨벤션, PR 룰)
  • 프로젝트 진입 시 자동 로드 (~2-5K 토큰)

/project/src/CLAUDE.md             폴더 단위 (선택)
  • 특정 폴더 진입 시만 로드
  • 모듈별 깊은 규칙
  • CLAUDE.md 비대화 방지
```

**효과:**

- 모든 룰을 ~/.claude/CLAUDE.md에 다 넣으면 80K 토큰
- 3계층 분리 시 매 세션 ~12K + 필요 시만 추가
- → **토큰 절감 5배 이상**

**강사 실측:**

| 위치 | 토큰 |
|---|---|
| ~/.claude/CLAUDE.md | 12K |
| Yurisma/CLAUDE.md | 3K |
| BeautyDecode/CLAUDE.md | 2K |
| proposal-bid/CLAUDE.md | 2K |
| 총 (Yurisma 작업 시) | 15K (전체 다 로드 시 19K) |

---

### 기법 6 — MEMORY.md 200줄 인덱스 패턴

**원리:** MEMORY.md = 200줄 인덱스만. 본문은 별도 .md 분리.

**구조:**

```
~/.claude/memory/
├── MEMORY.md                 200줄 인덱스 (매 세션 자동 로드)
└── 80+ 별도 .md              필요 시만 로드
    ├── user_*.md             사용자 프로필
    ├── feedback_*.md         피드백·룰
    ├── project_*.md          프로젝트 상태
    └── reference_*.md        외부 참조
```

**MEMORY.md 형식 예시:**

```markdown
## 닫힌 기능은 다 여는 것이 원칙 - see [feedback_no_feature_closure.md]
## 알뜰카드 "포인트" 용어 금지 - see [feedback_alddl_card_won_unit.md]
## Yurisma 회사 정보 2026-05 변경 - see [feedback_yurisma_company_info.md]
```

**효과:**

- 메타데이터 스캔 ~100 토큰 (인덱스 한 줄)
- 활성화 시 본문 ~500-2000 토큰만 로드
- 80+ 항목 등록되어 있어도 매 세션은 ~12K (인덱스 + ~/.claude/CLAUDE.md)

**강사 룰:** MEMORY.md 200줄 초과 시 → 별도 .md로 분리 + 인덱스 한 줄로 압축.

---

### 기법 7 — Skills 메타데이터 스캔

**원리:** Skills는 메타데이터 스캔 ~100 토큰. 활성화 시에만 본문 ~5000 토큰 로딩.

**메커니즘:**

```
Skills 134종 등록 상태:
   매 세션 메타 스캔: 134 × ~100 = 13.4K 토큰 (사실 sample만)
   활성화된 1-2개: 2 × ~5000 = 10K 토큰
   총 ~20K 토큰

만약 모든 본문 로딩한다면:
   134 × ~5000 = 670K 토큰 → 불가능
```

**활성화 시점:**

- 사용자 명시 `/skillname` 호출
- Hook이 `[SKILL SUGGESTION]` 태그로 추천 → Claude 자율 호출
- Tier 1 force 키워드 매칭

**강사 보유 134종 명세:** [skill-ecosystem.md](./skill-ecosystem.md) 참조.

---

## [C] 라우팅으로 절감 (어느 모델·도구가 적합한가)

### 기법 8 — 모델 라우팅

**원리:** 작업 복잡도에 맞는 모델 선택.

**라우팅 표:**

| 모델 | 작업 유형 | 비용 (상대) |
|---|---|---|
| **Haiku** | grep · lookup · 파일 요약 · 단순 분류 | 0.2x |
| **Sonnet** | 일반 작업 · 표준 코딩 | 1x (기준) |
| **Opus** | 설계 · 깊은 분석 · 멀티스텝 · 복잡 디버깅 | 5x |

**미스라우팅 사례 (실패):**

- "파일에서 함수 이름 찾아줘" 같은 단순 grep 작업에 **Opus 사용**
- 결과: 비용 5배 + 결과는 Haiku와 동일
- 룰 도출: "작업 복잡도 평가 먼저, 모델 선택 그 다음"

**강사 라우팅 룰 (CLAUDE.md `<model_routing>`):**

```
haiku   quick lookups, 파일 요약
sonnet  standard, 일반 코딩
opus    architecture, deep analysis
```

`<delegation_rules>` 명시: "Route code to `executor` (use `model=opus` for complex work). Uncertain SDK usage → `document-specialist`."

---

### 기법 9 — oh-my-opencode (Go 구독)

**원리:** 저렴 코딩 에이전트 라우팅. Kimi (Moonshot) · DeepSeek 등 저렴 모델로 단순 작업 위임.

**비용 비교:**

| 모델 | 입력 토큰 단가 (상대) |
|---|---|
| Claude Opus | 1x (기준) |
| Claude Sonnet | 0.2x |
| GPT-5.5 (Codex) | 0.3x |
| Kimi (Moonshot) | 0.05x ~ 0.1x |
| DeepSeek | 0.05x ~ 0.1x |

→ Kimi/DeepSeek = Opus의 **1/10 ~ 1/20** 비용.

**강사 실 사용:**

| 작업 | 라우팅 |
|---|---|
| PPTX 생성 (proposal-bid) | oh-my-opencode + DeepSeek |
| 요약·boilerplate 코드 | oh-my-opencode + Kimi |
| 단순 변환 작업 | oh-my-opencode |
| 복잡 설계 | Claude Code (Opus 직접) |
| 교차 검증 | Codex CLI |

**환경 격리:** `OPENCODE_DISABLE_CLAUDE_CODE_SKILLS=true` 환경변수로 OpenCode 전용 스킬 경로(`~/.config/opencode/skills/`) 격리.

---

### 기법 10 — Hermes / OpenClaw 수집 분리

**원리:** 수집은 코딩 컨텍스트 안 먹음. 메인 작업과 비용·맥락 분리.

**구조:**

```
[코딩 작업 컨텍스트]
   Claude Code (Opus) — 코딩 메인
   Codex CLI — 교차 검증
   oh-my-opencode — 저렴 fallback

[수집 작업 컨텍스트] — 별도 분리
   Hermes — 외부 자료 (논문·웹·뉴스) 자동 수집
            18 LLM 중 비용 적합 선택 (강사는 저렴 모델 선호)
   OpenClaw — OAuth 자율 컴퓨터 사용
              외부 vendor portal 접근
```

**효과:**

- 수집 작업이 코딩 세션 컨텍스트를 안 먹음
- 야간 자동 수집 → 코딩 비용 X
- 작업 흐름 분리로 메인 세션 효율 ↑

**강사 운영:** Hermes 수집은 cron 야간 실행 → 옵시디언 볼트에 자동 적재. 다음 날 작업 시 볼트에서 검색만 하면 됨.

---

## [D] 워크플로우로 절감 (Tier 게이트)

### 기법 11 — /clear + 세션 분리

**원리:** 새 작업 시 컨텍스트 초기화. 한 세션에서 작업 섞지 않기.

**강사 룰 (CLAUDE.md):**

> "clear를 자주 사용하세요. 새 작업을 시작할 때 컨텍스트를 비워야 할루시네이션이 줄고 윈도우를 최대로 활용할 수 있습니다."

**세션 분리 가이드:**

- 작업 A 끝 → `session-handoff` 실행 → 다음 세션 메모 저장
- `/clear` → 새 작업 B 시작
- 작업 B 끝 → 다시 `session-handoff` → `/clear`

**효과:**

- 작업별 컨텍스트 격리 → 할루시네이션 감소
- 윈도우 최대 활용 → 깊은 작업 가능
- `session-handoff` 통해 다음 세션 연결 보장

---

### 기법 12 — Tier 게이트 (vault → 외부)

**원리:** 사내 지식(vault) 무비용 검색을 외부 API 비용 검색보다 우선.

**Tier 게이트 룰 (CLAUDE.md Knowledge Hierarchy Protocol):**

```
Tier 1   세션 내 (CLAUDE.md · memory · 읽은 파일)     무비용
Tier 2a  30_Claude replay-learnings (310+ entries)    무비용
Tier 2b  20_Wiki qmd-search (988 파일)                무비용
Tier 2c  GBrain query (Supabase 그래프)               거의 무비용
Tier 3   context7 · brave · firecrawl                [비용 발생]

원칙: Tier 3 진입 전 Tier 2 통과
```

**발동 트리거 (Strong):**

- 프로젝트명 언급 (harness, Yurisma, BeautyDecode 등)
- 외부 기술/도구/사람 고유명사
- 과거 참조 ("이전에", "예전에", "우리 ~했잖아")
- 작업 재개 ("이어서", "다시", "계속")
- Tier 3 사용 전 (gate)

**Skip 조건:**

- 사용자가 "바로", "검색 없이", "vault 무시" 명시
- 단순 수학·문법·인사·명확한 사실
- 같은 세션 동일 쿼리 이미 검색
- QMD 오류·미설치

**Score 기반 활성화:**

- score ≥ 0.9 (High): 파일 전문 Read 후 핵심 인용
- score 0.7-0.9 (Medium-High): 요약 + "읽어볼까요?" 제안
- score 0.5-0.7 (Medium): 제목만 + "관련 있어 보여요"
- score < 0.5 (Low): 조용히 skip

**효과:** 사내 지식이 988 파일까지 쌓이면서 외부 API 호출 거의 0. 토큰비 폭감.

**이행률:** 80~90% (자동 100% X — 정직 회수).

---

## 2. 토큰 4분류 기초

```
input_tokens          사용자 프롬프트 + 시스템 컨텍스트
output_tokens         모델이 생성한 응답
cache_read_tokens     프롬프트 캐시에서 읽은 입력 (저렴, 0.1x)
cache_write_tokens    캐시에 기록한 입력 (약간 비쌈, 1.25x, 1회만)
```

**플랜별 비용 구조:**

| 플랜 | 가격 | 사용량 |
|---|---|---|
| Pro | $20/월 | 기본 사용량 |
| Max 5x | $100/월 | Pro 5배 |
| Max 20x | $200/월 | Pro 20배 |
| API | per-token | 토큰 단위 직접 |

---

## 3. 실패 사례 3건 (Day 5 본편 예고)

### 사례 1 — auto-accept 무한루프 $50/일 폭주

**상황:** 종료 조건 없이 auto-accept 모드 + 자리 비움.
**결과:** 8시간 동안 같은 작업 변형 반복 → $50/일 비용.
**도출 룰:**
- 종료 조건 명시 + 시간 제한
- 비용 알람 (일일 한도 알림)
- auto-accept 모드 시 자리 비움 금지

### 사례 2 — Max 20x 한도 24시간 차단

**상황:** 무거운 작업 연속 3시간 → 일일 한도 초과.
**결과:** 24시간 차단 → 다음 날까지 대기.
**도출 룰:**
- 사용량 모니터링 + 휴식 사이클
- 한도 80% 도달 시 alert
- 무거운 작업은 분할 진행

### 사례 3 — Opus 미스라우팅 (단순 grep)

**상황:** "파일에서 함수 이름 찾아줘" 같은 단순 grep에 Opus 사용.
**결과:** 비용 5배 / 결과는 Haiku와 동일.
**도출 룰:**
- 모델 라우팅 — 작업 복잡도 평가 먼저
- 강사 CLAUDE.md `<model_routing>` 룰 명시

---

## 4. 본인 손익분기 계산

```
입력:
  주당 코딩 시간: ____ 시간
  AI 보조 비중 (Copilot 기준): ____ %
  평균 작업 복잡도 (1~5): ____

추천 플랜:
  5시간 미만:   API per-token (사용량만큼만)
  5~15시간:     Pro $20
  15~30시간:    Max 5x $100
  30시간+:      Max 20x $200
```

12 기법 모두 적용 시 위 추천보다 **한 단계 아래 플랜으로도 가능**.

---

## 5. 관련 자료

- [architecture.md](./architecture.md) — 6 레이어 + 3 Tier
- [hook-chain.md](./hook-chain.md) — Hook chain 5종 (RTK 메커니즘 포함)
- [memory-vault.md](./memory-vault.md) — MEMORY.md 200줄 패턴
- [search-system.md](./search-system.md) — Tier 게이트 룰

---

**Last updated:** 2026-05-12 (v1)
