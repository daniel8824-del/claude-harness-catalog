---
layout: default
title: Hook Chain
subtitle: 5종 Hook + RTK 60-90% 절감 상세
---

# Hook Chain — 5종 + RTK 상세

> 본 문서는 [Daniel's Claude Harness Catalog](./index.html) §6의 깊이 읽기 자료입니다.
> PreToolUse Hook chain 5종의 코드 위치·알고리즘·threshold·실측 매칭률을 자세히.

---

## 1. Hook chain 전체 흐름

```
사용자 발화
    ↓
┌──────────────────────────────────────────────────────────────┐
│ [1] force-detector.mjs                                       │
│     145개 키워드 frontmatter polish 매칭                       │
│     매칭? → Tier 1 즉시 실행                                  │
└──────────────────────────────────────────────────────────────┘
    ↓
┌──────────────────────────────────────────────────────────────┐
│ [2] mode-detector.mjs                                        │
│     OMC tier-0 force 8 + 실행 mode 4                         │
│     매칭? → Tier 1 즉시 실행                                  │
└──────────────────────────────────────────────────────────────┘
    ↓
┌──────────────────────────────────────────────────────────────┐
│ [3] embedding-daemon.mjs                                     │
│     bge-m3-q8_0 1024dim 로컬 임베딩                           │
│     cold 70-130s → warm 50-300ms                             │
│     Unix socket /tmp/skill-embedding.sock                    │
└──────────────────────────────────────────────────────────────┘
    ↓
┌──────────────────────────────────────────────────────────────┐
│ [4] embedding-suggester.mjs                                  │
│     threshold 0.50 · topK 3                                  │
│     짧은 한국어 16 패턴 augmentation                          │
│     메타 토론 4+8 패턴 필터                                   │
│     [SKILL SUGGESTION] 태그 생성                              │
└──────────────────────────────────────────────────────────────┘
    ↓
┌──────────────────────────────────────────────────────────────┐
│ [5] auto-recall.mjs                                          │
│     decision-moment learning · threshold 0.60                │
└──────────────────────────────────────────────────────────────┘
    ↓
┌──────────────────────────────────────────────────────────────┐
│ [5'] replay-learnings-recall.mjs                             │
│      30_Claude semantic · first-prompt recall                 │
│      per-session flag (한 세션 1회만)                          │
└──────────────────────────────────────────────────────────────┘
    ↓
Claude가 맥락 보고 Tier 2 호출 결정
```

---

## 2. Hook 1 — force-detector.mjs

### 코드 위치

```
~/.claude/hooks/force-detector.mjs
```

### 데이터 소스

`~/.claude/force-keywords.json` — 145개 키워드 단일 source.

이 키워드들은 **frontmatter polish 마이그레이션 결과**로, SKILL.md의 `description` 필드를 영문 표준으로 정리한 부산물입니다.

### 알고리즘

```
사용자 발화 입력
   ↓
정규식 매칭 (145 키워드 순회)
   ↓
매칭 발견? 
   YES → 해당 스킬 즉시 force 실행 ([MAGIC KEYWORD: SKILL] 태그)
   NO  → 다음 Hook으로 넘김
```

### 매칭 예시

```
"이 코드 리뷰해줘"        → code-review 매칭
"디자인 리뷰 가능?"        → design-review 매칭
"qa 테스트 돌려"           → qa 매칭
"빌드 헬스 체크"           → health 매칭
"보안 리뷰 부탁"           → security-review 매칭
```

### 강사 환경 잔재

145 keyword 매칭은 frontmatter polish triggers의 산물이며, 본 시스템에서는 `force-keywords.json`을 단일 source로 운영합니다.

---

## 3. Hook 2 — mode-detector.mjs

### 코드 위치

```
~/.claude/hooks/mode-detector.mjs
(이전 keyword-detector.mjs → mode-detector.mjs rename, 2026-05-01 Phase C 잔여)
```

### 데이터 소스

코드 내장 (deprecated SKILL.md frontmatter triggers 흡수 완료).

### 매칭 대상

**OMC tier-0 force skill 8종:**

| 키워드 | 스킬 |
|---|---|
| `ralph` | oh-my-claudecode:ralph |
| `autopilot` | oh-my-claudecode:autopilot |
| `ultrawork` | oh-my-claudecode:ultrawork |
| `ccg` | oh-my-claudecode:ccg |
| `ralplan` | oh-my-claudecode:ralplan |
| `deep-interview` | oh-my-claudecode:deep-interview |
| `ai-slop-cleaner` | oh-my-claudecode:ai-slop-cleaner |
| `cancelomc` / `stopomc` | oh-my-claudecode:cancel |

**실행 Mode 메시지 4종:**

| 메시지 | 모드 |
|---|---|
| `tdd` | Test-Driven Development |
| `code-review` | 코드 리뷰 |
| `security-review` | 보안 리뷰 |
| `ultrathink` | 깊은 추론 |

### 강사 운영 룰

사용자가 **정확한 동사형 단어**를 사용한 경우만 동작. 의도 추론은 X.

**폐기 (B6, 2026-05-02):**
- `deepsearch` → explore agent + Glob/Grep 직접 호출이 더 정확
- `analyze` → pdca skill의 analyze 키워드와 중복

---

## 4. Hook 3 — embedding-daemon.mjs

### 코드 위치

```
~/.claude/hooks/embedding-daemon.mjs
```

### 동작 메커니즘

**모델:** bge-m3-q8_0 (1024dim, multilingual)
**경로:** `~/.claude/models/bge-m3-q8_0.gguf`

**소켓 통신:** Unix socket `/tmp/skill-embedding.sock`

```
첫 실행 (cold start):
   1. GGUF 모델 메모리 로딩 (~70-130s)
   2. daemon process 시작
   3. socket bind
   
이후 실행 (warm):
   1. socket 연결 (50-300ms)
   2. 텍스트 → 1024dim 벡터 변환
   3. 결과 반환
```

### 자동 시작

`~/.bashrc`가 셸 시작 시 socket 부재면 daemon 자동 spawn (idempotent).

### 사전 계산된 임베딩

`~/.claude/hooks/build-skill-embeddings.mjs` 가 138개 스킬 description을 사전 계산하여 `~/.claude/skill-embeddings.json`에 저장 (mafia wrapper 7개 추가 포함).

### 강사 실측

- cold start: 70-130s
- warm 응답: 50-300ms
- 매 발화마다 임베딩 1회 호출

---

## 5. Hook 4 — embedding-suggester.mjs

### 코드 위치

```
~/.claude/hooks/embedding-suggester.mjs
```

### 핵심 파라미터

```
threshold:  0.50 (2026-05-02 0.45→0.50 상향)
topK:       3
boost:      도구함 내 스킬 +0.03 (Phase G)
```

### 라우터

`~/.claude/hooks/embedding-router.mjs`의 `findSkillSuggestions(prompt)` 함수가 핵심 라우팅을 담당합니다.

### 짧은 한국어 augmentation (16 패턴)

cutoff ≤24자 발화에 대해 16 패턴 augmentation 적용 (B7에서 16→24 확장). 짧은 한국어 발화는 임베딩 매칭률이 떨어지기 때문에 패턴 보강.

### 메타 토론 필터 (4+8 패턴)

메타 토론(스킬 자체에 대한 토론) vs 실제 작업 요청을 구분하기 위한 필터.

- 메타 4패턴 + cron prompt 3패턴 (추가)
- false positive 방지

### 매칭 알고리즘

```
1. 사용자 발화 → embedding-daemon으로 1024dim 벡터화
2. 138개 사전 계산된 스킬 description 벡터와 cosine similarity
3. threshold 0.50 이상인 결과 추출
4. topK 3 선정
5. 메타 토론 필터 적용
6. 짧은 한국어 augmentation 적용
7. 도구함 내 스킬 +0.03 boost
8. [SKILL SUGGESTION] 태그로 Claude에 전달
```

### 로그

매칭 결과는 `.router-matches.jsonl`에 기록되어 튜닝 데이터로 활용됩니다.

---

## 6. Hook 5 — auto-recall.mjs

### 코드 위치

```
~/.claude/hooks/auto-recall.mjs
```

### 데이터 소스

`~/.claude/auto-recall-rules.json` — 3 rule 등록 (Phase G).

### 동작

decision-moment에서 자동 회상. 사용자가 의사결정 시점에 도달하면 관련 과거 결정·교훈을 자동 인용.

**threshold:** 0.60 (embedding-suggester보다 높게 설정 — 회상은 보다 정확한 매칭 필요)

### 로그

매칭 결과는 `.auto-recall-matches.jsonl`에 기록.

---

## 7. Hook 5' — replay-learnings-recall.mjs

### 코드 위치

```
~/.claude/hooks/replay-learnings-recall.mjs
```

### 동작

세션 첫 발화에서 30_Claude의 410+ entries 중 semantic 유사 항목을 회상하여 컨텍스트에 주입.

**per-session flag:** 한 세션에 1회만 실행 (중복 회상 방지).

### 회상 대상

- 30_Claude/02_Learnings/ (310+ entries)
- 30_Claude/05_Research/
- 30_Claude/06_Designs/
- 30_Claude/01_Sessions/
- 30_Claude/04_Plans/

### 강사 환경 실측 사례

```
사용자 발화: "Hermes 에이전트 구축 시작하자"
    ↓
Trigger 감지: "Hermes" 고유명사 + "구축" 새 작업
    ↓
Tier 2a — replay-learnings 자동 발동
  L1: grep local learn-rule
  L2: qmd search "Hermes agent" -c obsidian -n 10
    ↓
결과 상위: research-2026-04-14-gbrain-ouroboros-analysis (0.92)
score ≥ 0.9 → 전문 Read
    ↓
Claude 응답:
"볼트에 이미 Hermes 관련 4개 자료 있습니다:
- research 문서: GBrain은 Hermes용 인프라, Ollama 루트 주목
- hermes-agent-install-guide: 18 LLM 지원
- 2026-04-14 세션: Ouroboros와 비교 분석 완료
- design 문서: 지식 시스템 활용 원칙
어느 것부터 보시겠어요?"
```

---

## 8. RTK (Rust Token Killer) — 60-90% 절감

### 코드 위치

별도 Rust 바이너리 — `rtk` 명령으로 호출.

### 동작 원리

Claude Code 환경의 Bash hook이 dev 명령(`git`, `npm`, `ls` 등)을 **자동으로 rtk 프록시**로 우회합니다.

```
사용자 또는 Claude가 명령 실행:
  git status

   ↓ (Bash hook 인터셉트, 0 토큰 오버헤드)
   
실제 실행:
  rtk git status
  
   ↓
   
rtk가 git status 출력을 압축·필터링:
  - 변경 없는 파일 제거
  - 긴 경로 단축
  - 색상 코드 제거
  - 요약 형식 출력
  
   ↓
   
Claude가 받는 출력: 원본의 10-40% 크기
```

### 메타 명령 (rtk 직접 사용)

```bash
rtk gain              # 토큰 절감 분석
rtk gain --history    # 명령별 사용 이력 + 절감
rtk discover          # Claude Code 히스토리에서 놓친 기회 분석
rtk proxy <cmd>       # 필터링 없이 원본 실행 (디버깅)
```

### 설치 확인

```bash
rtk --version         # 버전 확인
rtk gain              # 작동 확인 (not found 안 떠야)
which rtk             # 정확한 바이너리 경로 확인
```

### Name collision 주의

`reachingforthejack/rtk` (Rust Type Kit)이 동시 설치된 경우 충돌 가능. `rtk gain` 명령으로 구분.

### 강사 실측 효과

- dev 명령 출력의 60-90% 토큰 절감
- 매 세션 자동 적용 (사람 개입 0)
- `rtk gain --history`로 누적 통계 확인

---

## 9. 매칭 시스템 진화 — 2026-05-02 Phase 완료 시점

### Hook chain 정정

5 hook chain 최종:
1. `force-detector` (frontmatter polish 145)
2. `mode-detector` (OMC tier-0 force 8 + mode 4)
3. `embedding-suggester` (자연 발화 추천)
4. `auto-recall` (decision-moment learning)
5. `replay-learnings-recall` (30_Claude first-prompt)

### 임베딩 매칭률 — 47 자연 발화 bench

| 지표 | 결과 |
|---|---|
| Strict | **87.2%** |
| Lenient | **97.4%** |
| False Positive | **100% clean** |

Lenient 1건 회귀 — "UI 검토" 발화에 mafia-codereview-* over-match. 옵션 A 수용 (영문 only + 단일 벡터 임베딩의 본질적 한계, mafia wrapper 통합 효용 압도).

### 폐기 항목 (Phase C/D/E/F)

| 항목 | 폐기 이유 |
|---|---|
| `daniel-skill-router.mjs` | Phase C — force-detector + embedding-suggester로 분리 |
| `keyword-detector.mjs` | Phase C 잔여 — mode-detector.mjs로 rename |
| SKILL.md frontmatter `triggers.force` + `triggers.suggest` | 32 파일 — force-keywords.json 단일 source 정합 |
| `pre-tool-use.mjs` 도구함 enforce 블록 | Phase E — 임베딩 메인 시스템 효용 적음 |
| Tier 3 카테고리 9개 | Phase F — T2 통합 |

---

## 10. 강사 환경 디렉토리 구조

```
~/.claude/
├── hooks/
│   ├── force-detector.mjs
│   ├── mode-detector.mjs
│   ├── embedding-daemon.mjs
│   ├── embedding-suggester.mjs
│   ├── auto-recall.mjs
│   ├── replay-learnings-recall.mjs
│   ├── build-skill-embeddings.mjs
│   ├── build-skill-registry.mjs
│   └── embedding-router.mjs (helper)
├── models/
│   └── bge-m3-q8_0.gguf
├── force-keywords.json       (145 keyword)
├── auto-recall-rules.json    (3 rule)
├── skill-embeddings.json     (138 사전 계산)
├── .router-matches.jsonl     (튜닝 데이터)
└── .auto-recall-matches.jsonl
```

---

## 11. 관련 자료

- [architecture.md](./architecture.md) — 6 레이어 + 3 Tier 풀스택
- [token-saving.md](./token-saving.md) — 토큰 절감 12 기법 (RTK 포함)
- [memory-vault.md](./memory-vault.md) — 메모리·볼트 시스템
- [skill-ecosystem.md](./skill-ecosystem.md) — 134종 스킬 전체

---

**Last updated:** 2026-05-12 (v1)
