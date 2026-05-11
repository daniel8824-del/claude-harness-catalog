---
layout: default
title: Skill Ecosystem
subtitle: 134종 + project-bootstrap + 화이트리스트 27
---

# Skill Ecosystem — 134종 + project-bootstrap + 화이트리스트 27

> 본 문서는 [Daniel's Claude Harness Catalog](./index.html) §11의 깊이 읽기 자료입니다.
> 강사 보유 글로벌 스킬 134종 전체 명세 + project-bootstrap 시스템 + 도구함 운영.

---

## 1. 스킬 3 위치

스킬은 3 군데에 살 수 있습니다.

| 위치 | 경로 | 강사 환경 | 누가 만든 거? |
|---|---|---|---|
| **🌍 글로벌** | `~/.claude/skills/` | **134종** (top-level dirs 135) | Daniel 직접 클론/커스텀 |
| **📦 플러그인** | `~/.claude/plugins/cache/*/skills/` | **157종** (캐시 전체) | 플러그인 설치 시 자동 |
| **🏠 로컬** | `<project>/.claude/skills/` | 0 (현재 없음) | 프로젝트별 |

**플러그인 묶음 상세 (강사 환경):**
- OMC 4.13.0 → 38개 (autopilot, ralph, ultrawork, team, omc-* 등)
- Vercel 0.24.0 → 39개
- Obsidian → 5개 (obsidian-cli, markdown, json-canvas, bases, defuddle)
- Figma → 7개 (figma-use, figma-implement-design 등)
- Discord / Telegram → 2개씩 (configure, access)

**왜 분리?**
- 글로벌 = 어느 프로젝트에서나 호출
- 로컬 = 그 프로젝트만
- 플러그인 = 묶음으로 동시 설치/제거

---

## 2. 글로벌 스킬 14 카테고리 (134종)

### 1. 디자인 향상 — `imp-*` (18종)

만든 UI를 더 좋게 (색·타이포·레이아웃·애니메이션·접근성 등)

`imp-adapt` · `imp-animate` · `imp-audit` · `imp-bolder` · `imp-clarify` · `imp-colorize` · `imp-critique` · `imp-delight` · `imp-distill` · `imp-harden` · `imp-impeccable` · `imp-layout` · `imp-optimize` · `imp-overdrive` · `imp-polish` · `imp-quieter` · `imp-shape` · `imp-typeset`

**출처:** pbakaus/impeccable (Anthropic frontend-design 기반)

### 2. 디자인 시스템 — `taste-*` (9종)

디자인 결(brutalist · minimalist · soft 등) 미리 정해놓고 강제

`taste-brutalist` · `taste-gpt-tasteskill` · `taste-minimalist` · `taste-output` · `taste-redesign` · `taste-skill` · `taste-soft` · `taste-stitch` · `taste-taste`

**출처:** Leonxlnx/taste-skill (taste-gpt-tasteskill만 GPT-5.4 design taste 강의 영향)

### 3. 개발 파이프라인 — `phase-*` (9종)

새 프로젝트를 만들 때, 처음부터 끝까지 순서 잡아주는 단계들

`phase-1-schema` · `phase-2-convention` · `phase-3-mockup` · `phase-4-api` · `phase-5-design-system` · `phase-6-ui-integration` · `phase-7-seo-security` · `phase-8-review` · `phase-9-deployment`

**출처:** BKit 계열 (`agent: bkit:pipeline-guide` + `${PLUGIN_ROOT}/templates/pipeline/...`)

### 4. 플래닝/리뷰 (12 + 1)

코드 짜기 전에 "뭘 만들지" 합의·리뷰

`writing-plans` · `imp-shape` · `brainstorming` · `prometheus-planning` · `metis-review` · `momus-review` · `plan-ceo-review` · `plan-design-review` · `plan-devex-review` · `plan-eng-review` · `office-hours` · `autoplan` · `(참) autoresearch:plan`

**출처 (혼합):**
- `plan-*` · `autoplan` · `office-hours` = gstack 원본
- `writing-plans` · `brainstorming` · `imp-shape` = obra/superpowers 계열
- `prometheus` · `metis` · `momus` = code-yeongyu/oh-my-openagent ★54K 풀 변환 (한국어 + OMC 환경 적응)

### 5. 지식/메모리 (11종) ⭐ **오늘 핵심**

토큰 절약 · 세션 간 지식 누적 · 위키

| 스킬 | tier | 외부 원본 | Daniel 변형 |
|---|---|---|---|
| `vault-save` | 🟢 Custom | (옵시디언-클로드 패턴 영감) | **본문 자체 작성** — QMD 후처리 + 4영역 규칙 |
| `session-handoff` | 🟡 Adapted | rohitg00/pro-workflow ★2K | Obsidian 4영역 통합 + 본문 한국어 재작성 |
| `replay-learnings` | 🟡 Adapted | rohitg00/pro-workflow ★2K | 30_Claude 5폴더 + QMD 통합 |
| `qmd-search` · `qmd-manage` | 🟡 Wrapped | tobi/qmd (Shopify CEO) | wrapper |
| `wiki` | 🟢 Custom | Karpathy LLM Wiki + Obsidian-Claude 패턴 | **Daniel 9-verb 지식 엔진** |
| `learn-rule` | 🟡 Adapted | rohitg00/pro-workflow ★2K | **QMD 중복 체크 + 30_Claude/02_Learnings 저장 룰 추가** |
| `learn` · `retro` · `harness` | 🔴 Original | garrytan/gstack | 원본 |
| `remember` | 🔴 Original | Yeachan-Heo/oh-my-claudecode | 원본 |

🟢 = Custom by Daniel · 🟡 = Adapted/Wrapped · 🔴 = Original

### 6. 수집/리서치 (5종) — Daniel 본인 작성

외부 데이터 자동 수집

`blog-collector` · `news-collector` · `instagram-collector` · `youtube-collector` · `naver-scholar`

**출처:** 🟢 5개 모두 Daniel 작성 (네이버/구글뉴스/유튜브/인스타/네이버학술 API 직접 연동)

### 7. 콘텐츠 생성 (10종)

PPTX · PDF · 랜딩페이지 · 보고서 자동 생성

`slide-pptx` 🟢 · `landing-page` 🟢 · `biz-plan` 🟢 · `proposal-bid` 🟢 · `notebook-lm` 🟢 · `pptx-autofill` 🟢 · `hwp-analyzer` 🟢 · `report-image` 🟢 · `report-tone` 🟢 · `remotion` · `remotion-best-practices`

**출처:** 🟢 9개 Daniel 작성, remotion = 외부

### 8. QA/검증 (12종)

만든 거 진짜 작동하나 확인

`design-review` · `design-consultation` · `design-html` · `design-shotgun` · `qa` · `qa-only` · `browse` · `gstack` · `devex-review` · `visual-verdict` (oh-my-claudecode) · `benchmark` · `canary` · `connect-chrome`

**출처:** gstack 묶음 + 일부 OMC

### 9. 안전/하네스 (8종)

위험 명령 가드 · 권한 · 컨텍스트 보호

`careful` 🔴 · `freeze` 🔴 · `unfreeze` 🔴 · `guard` 🔴 · `fewer-permission-prompts` 🔴 · `permission-tuner` 🔴 · `compact-guard` 🔴 · `context-optimizer` 🟡 · `cost-tracker` 🟡

**출처:**
- gstack: careful · freeze · unfreeze · guard · fewer-permission-prompts
- rohitg00/pro-workflow ★2K 원본 100% 동일: permission-tuner · compact-guard
- pro-workflow 변형: context-optimizer · cost-tracker

### 10. 워크플로우 (대표 항목)

코딩 단계별 자동화 (커밋·배포·테스트·디버그·리뷰)

`smart-commit` 🔴 · `ship` 🔴 · `land-and-deploy` 🔴 · `health` 🔴 · `investigate` 🔴 · `ai-slop-cleaner` (omc) 🔴 · `document-release` 🔴 · `review` 🔴 · `security-review` 🔴

**Superpowers 계열:** test-driven-development · systematic-debugging · verification-before-completion · executing-plans · finishing-a-development-branch · requesting-code-review · receiving-code-review · dispatching-parallel-agents · subagent-driven-development · using-git-worktrees · using-superpowers · writing-skills

### 11. 멀티-AI (3종)

Claude만 쓰지 않고 Codex·Gemini도 동시 호출

`codex` (global) · `ccg` (Claude-Codex-Gemini, OMC plugin) · `ask` (oh-my-claudecode:ask, OMC plugin)

**호출 방법 3종:**
1. `codex` 직접 (CLI 단독)
2. `/ask codex "<질문>"` (OMC 라우팅)
3. `/ccg <task>` (3-AI 합의 모드)

### 12. 자율 반복 — `autoresearch:*` (9종)

목표 정해놓고 만족할 때까지 알아서 반복

`autoresearch` · `autoresearch:plan` · `autoresearch:debug` · `autoresearch:ship` · `autoresearch:scenario` · `autoresearch:fix` · `autoresearch:security` · `autoresearch:predict` · `autoresearch:learn`

**출처:** karpathy/autoresearch

### 13. 통신/외부 도구 (다수)

외부 서비스 연동

`discord:configure` · `discord:access` · `telegram:configure` · `telegram:access` · `figma:figma-use` · `figma:figma-implement-design` · `figma:*` (6) · `openclaw-ops` · `openclaw-skill` · `paperclip` · `claude-api` · `pair-agent` · `cmux-orchestrator` · `configure-notifications` (omc)

**출처:** claude-plugins-official + OMC + 본인 커스텀 (openclaw, paperclip)

### 14. 유틸리티/도메인 (12종)

1회용 스킬·도메인 특화

`brainstorming` · `feynman` · `seteuk` 🟢 · `data-analysis` 🟢 · `graphify` · `openboard` · `wire-framer` · `pdca` · `development-pipeline` · `init` · `setup-deploy` · `setup-browser-cookies` · `update-config` · `keybindings-help` · `simplify` · `schedule` · `loop` · `gstack-upgrade` · `cso` · `pptx-autofill`

**출처:** 혼합 (seteuk · data-analysis = Daniel 커스텀, 나머지 = gstack/OMC/Superpowers)

---

## 3. project-bootstrap 시스템

### 원칙

글로벌 `~/.claude/skills/` (134종)은 그대로. 프로젝트 폴더에 들어갔을 때 그 프로젝트에 맞는 스킬만 도구함에 노출. 글로벌 스킬은 사용자 명시 호출만 허용.

### 구성 요소

| 요소 | 위치 |
|---|---|
| 인터뷰 스킬 | `~/.claude/skills/project-bootstrap/SKILL.md` |
| 매핑 데이터 | `~/.claude/skills-intent-mapping.json` (135 스킬 × 10 의도 그룹) |
| 심링크 스크립트 | `~/.claude/skills/project-bootstrap/setup-symlinks.sh` |
| 프로젝트 도구함 | `{project}/.claude/skills/` (심링크) |
| 프로젝트 메타 | `{project}/.claude/skills-context.json` |
| 임베딩 boost | `~/.claude/hooks/embedding-suggester.mjs` 도구함 내 스킬 score +0.03 |

### 의도 그룹 10개 (한 단어 영어)

`plan` · `design` · `verify` · `review` · `deploy` · `debug` · `retro` · `docs` · `security` · `content`

각 그룹마다 사용자 인터뷰에서:
- **복수 답변** 가능
- **그룹 skip** 허용
- **한 도구가 N개 그룹 등장** 허용 (중복 매핑 정상)

### 운영 흐름

```
새 프로젝트 폴더 진입
   ↓
사용자: /project-bootstrap
   ↓
인터뷰 (10 의도 그룹 × 복수 답변 + skip)
   ↓
{project}/.claude/skills/ 심링크 생성 (화이트리스트 27 + 사용자 선택 N)
   ↓
{project}/.claude/skills-context.json 작성
   ↓
이후 세션에서 embedding-suggester가 도구함 boost (+0.03) 적용 + Claude 자율 판단
```

---

## 4. 화이트리스트 27종 (항상 활성)

인터뷰 외 항상 활성. 프로젝트 폴더 진입 시 자동 심링크.

### 글로벌 20종

| 스킬 | 카테고리 |
|---|---|
| `careful` | 안전 |
| `freeze` | 안전 |
| `unfreeze` | 안전 |
| `guard` | 안전 |
| `brainstorming` | 플래닝 |
| `writing-plans` | 플래닝 |
| `smart-commit` | 워크플로우 |
| `verification-before-completion` | 워크플로우 |
| `compact-guard` | 안전 |
| `systematic-debugging` | 워크플로우 |
| `replay-learnings` | 메모리 |
| `learn-rule` | 메모리 |
| `session-handoff` | 메모리 |
| `vault-save` | 메모리 |
| `learn` | 메모리 |
| `using-superpowers` | 메타 |
| `cost-tracker` | 비용 |
| `context-optimizer` | 컨텍스트 |
| `autoresearch` | 자율 반복 |
| `paperclip` | 통신 |

### OMC 플러그인 7종

| 스킬 | 용도 |
|---|---|
| `autopilot` | 워크플로우 자동 |
| `ralph` | 자율 반복 루프 |
| `ultrawork` | 병렬 작업 |
| `ccg` | 3-AI 합의 |
| `ralplan` | ralph + 플래닝 |
| `deep-interview` | 깊은 인터뷰 |
| `ai-slop-cleaner` | AI slop 정리 |

---

## 5. 자율 룰 (CLAUDE.md HARD-RULE)

프로젝트 폴더에서 `{project}/.claude/skills-context.json` 존재 시:

- **AI 자동 호출:** 도구함 내 스킬만 호출. 도구함 외 자동 호출 X.
- **사용자 명시 호출 (`/skillname`):** 도구함 외도 통과. 자유.
- **자주 호출하는 글로벌 스킬:** `/project-bootstrap` 재실행으로 정식 추가 권장.

PreToolUse hook 도구함 enforce는 폐기 (Phase E, 2026-05-01). 임베딩 메인 시스템에서 `embedding-suggester.mjs`가 도구함 내 스킬에 +0.03 boost 주는 방식으로 대체. Claude는 본 룰을 자율 판단으로 따른다.

---

## 6. 스킬 위치 이원화 (Claude Code vs OpenCode)

```
~/.claude/skills/                Claude Code 전용 (134종)
~/.config/opencode/skills/       OpenCode 전용 (분리 운영)
```

**환경변수:** `OPENCODE_DISABLE_CLAUDE_CODE_SKILLS=true`

이 환경변수 설정 시 OpenCode가 Claude Code 스킬을 로드하지 않음. 경로 격리로 두 시스템의 스킬 충돌 방지.

---

## 7. Daniel 본인 작성 자산 (Custom)

### 글로벌 스킬 (🟢)

- `vault-save` — Obsidian 볼트 4영역 저장 + QMD 인덱싱
- `wiki` 9-verb — Karpathy LLM Wiki + Obsidian-Claude 패턴 + qmd/feynman 경계 참고
- `seteuk` — 한국 고등학생 세부능력 및 특기사항 작성 지원
- `data-analysis` — 데이터 분석·시각화
- `blog-collector` · `news-collector` · `instagram-collector` · `youtube-collector` · `naver-scholar`
- `slide-pptx` · `landing-page` · `biz-plan` · `proposal-bid` · `notebook-lm` · `pptx-autofill` · `hwp-analyzer` · `report-image` · `report-tone`
- `openclaw-ops` · `openclaw-skill`
- `paperclip`

### Daniel 공개 GitHub 자산

- [daniel8824-del/claude-skill-catalog](https://daniel8824-del.github.io/claude-skill-catalog/) — 스킬 카탈로그 hub
- [daniel8824-del/claude-wiki-verbs](https://daniel8824-del.github.io/claude-wiki-verbs/) — 9-verb 위키 시스템
- [daniel8824-del/claude-feynman-research-skill](https://daniel8824-del.github.io/claude-feynman-research-skill/) — 출처 검증 리서치
- [daniel8824-del/claude-harness-catalog](https://daniel8824-del.github.io/claude-harness-catalog/) — 본 카탈로그

콘텐츠 스킬 레포:
- `slide-pptx-generator` · `landing-page-generator` · `bizplan-generator` · `proposal-bid-generator`
- `blog-collector-skill` · `youtube-collector-skill` · `news-collector-skill` · `instagram-collector-skill`
- `notebook-lm-slides` · `naver-blog-collector`

---

## 8. 스킬 통계

| 지표 | 강사 환경 |
|---|---|
| 글로벌 스킬 | 134종 |
| 플러그인 캐시 전체 | 157종 |
| 카테고리 | 14 |
| 화이트리스트 항상 활성 | 27 (글로벌 20 + OMC 7) |
| Daniel Custom (🟢) | 20+ |
| Daniel Adapted (🟡) | 10+ |
| Original (🔴) | 100+ |
| 메타 스캔 토큰 | ~100 |
| 활성화 토큰 | ~5000 |

---

## 9. 관련 자료

- [architecture.md](./architecture.md) — 6 레이어 + 3 Tier
- [hook-chain.md](./hook-chain.md) — embedding-suggester boost
- [memory-vault.md](./memory-vault.md) — 메모리 스킬
- [search-system.md](./search-system.md) — qmd/gbrain/graphify 스킬

---

**Last updated:** 2026-05-12 (v1)
