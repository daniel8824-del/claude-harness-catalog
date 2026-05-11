# Attribution — 외부 출처 명시

본 카탈로그(`claude-harness-catalog`)는 다음 외부 자산의 조합 위에 강사 본인의 운영 룰을 누적한 결과입니다. 각 자산의 원본 출처와 라이센스를 명시합니다.

---

## 6 레이어 외부 자산

### Layer 1 — Superpowers
- **GitHub**: [obra/superpowers](https://github.com/obra/superpowers)
- **Stars**: ★167,056 (2026-04-25 기준)
- **역할**: 워크플로우 가드레일 (언제 무엇을 할지)
- **포함 스킬**: brainstorming · writing-plans · systematic-debugging · verification-before-completion · test-driven-development · executing-plans · 기타 다수

### Layer 2 — OMC (oh-my-claudecode)
- **GitHub**: [Yeachan-Heo/oh-my-claudecode](https://github.com/Yeachan-Heo/oh-my-claudecode)
- **Stars**: ★31,195
- **역할**: 에이전트 오케스트레이션 (누가/어떻게)
- **포함 스킬**: autopilot · ralph · ultrawork · ccg · ralplan · deep-interview · ai-slop-cleaner · team · cancel 등 38종

### Layer 3 — gstack
- **GitHub**: [garrytan/gstack](https://github.com/garrytan/gstack)
- **Stars**: ★82,801
- **역할**: 안전 + 전문 도구
- **포함 스킬**: careful · freeze · unfreeze · guard · browse · qa · benchmark · investigate · health · ship · land-and-deploy · review · security-review 등

### Layer 4 — autoresearch
- **GitHub**: [karpathy/autoresearch](https://github.com/karpathy/autoresearch)
- **Stars**: ★76,395
- **역할**: 자율 반복 프로토콜
- **포함 스킬**: autoresearch:* 9종 (plan, debug, ship, scenario, fix, security, predict, learn 등)

### Layer 5 — OMO (oh-my-openagent)
- **GitHub**: [code-yeongyu/oh-my-openagent](https://github.com/code-yeongyu/oh-my-openagent)
- **Stars**: ★54,031
- **역할**: Planning 메타 (prometheus · metis · momus)
- **변형**: 강사가 한국어 + OMC 환경 적응을 위해 풀 변환

### Layer 6 — pro-workflow
- **GitHub**: [rohitg00/pro-workflow](https://github.com/rohitg00/pro-workflow)
- **Stars**: ★2,007
- **역할**: Self-correcting memory
- **포함 스킬**: session-handoff · replay-learnings · learn-rule · 5 utility
- **변형**: 강사가 Obsidian 4영역 통합 + 본문 한국어로 재작성

---

## 외부 CLI 도구

### qmd (Quick Markdown Search)
- **GitHub**: [tobi/qmd](https://github.com/tobi/qmd)
- **Stars**: ★23,132
- **작성**: Tobi Lütke (Shopify CEO)
- **역할**: 로컬 마크다운 검색 — `qmd-search` / `qmd-manage` 스킬이 wrapping

### feynman (Research Skill)
- **GitHub**: [getcompanion-ai/feynman](https://github.com/getcompanion-ai/feynman)
- **Stars**: ★5,846
- **역할**: Pi CLI 학술 도구 — `feynman` 스킬이 wrapping
- **포함 subskill**: literature-review · deep-research · paper-writing · peer-review · source-comparison 등 15종

---

## 스킬 카테고리별 외부 출처

### 디자인 향상 (imp-* 18종)
- **출처**: [pbakaus/impeccable](https://github.com/pbakaus/impeccable)
- **기반**: Anthropic frontend-design 표준
- **포함**: imp-shape · imp-impeccable · imp-polish · imp-typeset 등 18종

### 디자인 시스템 (taste-* 9종)
- **출처**: [Leonxlnx/taste-skill](https://github.com/Leonxlnx/taste-skill)
- **포함**: taste-brutalist · taste-minimalist · taste-soft · taste-redesign 등 9종

### 개발 파이프라인 (phase-* 9종)
- **출처**: BKit 계열 (원본 레포 미확정)
- **포함**: phase-1-schema ~ phase-9-deployment

### 플래닝 (12종)
- 혼합 출처:
  - `plan-*` · `autoplan` · `office-hours` = gstack 원본
  - `writing-plans` · `brainstorming` · `imp-shape` = obra/superpowers
  - `prometheus-planning` · `metis-review` · `momus-review` = OMO 풀 변환 (Daniel 한국어 적응)

### 지식/메모리 (11종)
- 혼합 출처:
  - `vault-save` · `wiki` = Daniel 본인 작성
  - `session-handoff` · `replay-learnings` · `learn-rule` = pro-workflow (Daniel Obsidian 적응)
  - `qmd-search` · `qmd-manage` = tobi/qmd wrapper
  - `learn` · `retro` · `harness` = gstack 원본
  - `remember` = OMC 원본

### 콘텐츠 생성 (10종) — Daniel 본인 작성
- `slide-pptx` · `landing-page` · `biz-plan` · `proposal-bid` · `notebook-lm` · `pptx-autofill` · `hwp-analyzer` · `report-image` · `report-tone` (Daniel)
- `remotion` · `remotion-best-practices` (외부)

### 수집/리서치 (5종) — Daniel 본인 작성
- `blog-collector` · `news-collector` · `instagram-collector` · `youtube-collector` · `naver-scholar`

---

## 5 도구 (코딩 3 + 수집 2)

### Claude Code (Anthropic)
- **공식**: [claude.ai/code](https://claude.ai/code)
- **모델**: Opus 4.x / Sonnet / Haiku
- **라이센스**: 상용 구독 (Pro / Max 5x / Max 20x)

### Codex CLI (OpenAI)
- **공식**: [openai.com/index/introducing-gpt-5-5/](https://openai.com/index/introducing-gpt-5-5/)
- **모델**: GPT-5.5 (2026-04-23 출시)
- **Terminal-Bench**: 82.7% (2.0)
- **버전**: codex-cli 0.125.0 (2026-05 기준)

### oh-my-opencode (Go)
- **기반**: opencode (오픈소스) + OMC 패턴
- **모델 라우팅**: Kimi (Moonshot AI) · DeepSeek 등 저렴 모델
- **용도**: 저렴 코딩 에이전트 fallback (PPTX 생성 · 요약 · boilerplate)

### Hermes (수집)
- **기반**: 18 LLM 지원 멀티 모델 수집 에이전트
- **연계**: GBrain 지식 그래프 (Supabase)
- **용도**: 외부 자료(논문 · 웹 · 뉴스) 자동 수집 → 옵시디언 볼트 적재

### OpenClaw (수집)
- **기반**: OAuth 자율 컴퓨터 사용 에이전트
- **인증**: ChatGPT Pro 연계
- **용도**: 외부 SaaS · vendor portal 자율 접근 수집

---

## 강사 본인 작성 (Custom)

다음은 강사 본인이 처음부터 작성한 자산입니다.

### 글로벌 스킬 (Daniel Custom)
- `vault-save` — Obsidian 볼트 4영역 저장 후 QMD 인덱싱
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
- [daniel8824-del/claude-feynman-research-skill](https://daniel8824-del.github.io/claude-feynman-research-skill/) — 출처 검증 리서치 스킬
- [daniel8824-del/claude-harness-catalog](https://daniel8824-del.github.io/claude-harness-catalog/) — 본 카탈로그
- `slide-pptx-generator` · `landing-page-generator` · `bizplan-generator` · `proposal-bid-generator`
- `blog-collector-skill` · `youtube-collector-skill` · `news-collector-skill` · `instagram-collector-skill`
- `notebook-lm-slides` · `naver-blog-collector`

---

## 라이센스 정합 안내

- 외부 자산은 각 원본 레포의 라이센스를 따릅니다.
- 강사 본인 작성 자산은 본 카탈로그 공개와 함께 자유 사용 허용.
- 본 카탈로그는 강사가 6개월~1년 운영 중인 시스템 가시화 목적으로, 강의 보조 자료로 활용됩니다.

---

## 변경 이력

| 일자 | 변경 |
|---|---|
| 2026-05-12 | 본 attribution.md v1 작성 (카탈로그 신규 배포) |

---

**문서 끝.**
