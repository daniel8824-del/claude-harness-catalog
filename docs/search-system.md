---
layout: default
title: Search System
subtitle: QMD / GBrain / graphify 위키 시스템
---

# Search System — QMD / GBrain / graphify 위키 시스템

> 본 문서는 [Daniel's Claude Harness Catalog](./index.html) §9의 깊이 읽기 자료입니다.
> 사내 지식 무비용 검색 3종 + Tier 게이트 + 발동 트리거 + score 활성화 룰 풀스택.

---

## 1. 위키 시스템 아키텍처

```
[외부 자료 — 논문·뉴스·웹·SaaS·사내 시스템]
            ↓
┌──────────────────────────────────────────────┐
│  수집 레이어                                  │
│   Hermes        18 LLM · 외부 자료 자동 수집  │
│   OpenClaw      OAuth · 자율 컴퓨터 사용     │
│   firecrawl     URL 본문 추출               │
│   brave         웹 검색 (유료)               │
└──────────────────────────────────────────────┘
            ↓
┌──────────────────────────────────────────────┐
│  영구 저장 레이어                             │
│   Obsidian 30_Claude (6 영역 분류)           │
│   ├─ 01_Sessions   세션 핸드오프              │
│   ├─ 02_Learnings  교훈 (310+)               │
│   ├─ 03_Retros     주간 회고                  │
│   ├─ 04_Plans      플랜                       │
│   ├─ 05_Research   리서치                     │
│   └─ 06_Designs    설계 결정                  │
│   10_Raw/          원본 자료 (988 파일)       │
└──────────────────────────────────────────────┘
            ↓
┌──────────────────────────────────────────────┐
│  검색 레이어 (무비용 → 비용 게이트)            │
│   QMD       단순 의미·키워드 (default · 988)  │
│   GBrain    백링크·타임라인 (Supabase RRF)    │
│   graphify  community·god_nodes (3386 nodes) │
│   ↓ 결과 score                                │
│   ≥0.9 전문 Read / 0.5-0.9 요약 / <0.5 skip   │
└──────────────────────────────────────────────┘
            ↓
┌──────────────────────────────────────────────┐
│  활용 레이어                                  │
│   Claude Code 답변 컨텍스트에 자동 주입       │
│   Codex / oh-my-opencode 도 동일 게이트       │
│   Tier 3 외부 API 진입 전 게이트              │
└──────────────────────────────────────────────┘
```

---

## 2. QMD (Quick Markdown Search)

### 외부 출처

- GitHub: [tobi/qmd](https://github.com/tobi/qmd)
- Stars: ★23,132
- 작성: Tobi Lütke (Shopify CEO)
- 라이센스: 원본 레포 정책

### 역할

볼트 988 파일에 대한 **무비용 로컬 검색**. 단순 의미·키워드 매칭. Tier 2b 게이트의 default 도구.

### 강사 wrapper 스킬

- `qmd-search` — 검색 + 결과 표시
- `qmd-manage` — 인덱스 관리

### 사용 명령

```bash
qmd search "Hermes agent" -c obsidian -n 10
```

`-c obsidian` = 옵시디언 카테고리 (강사 볼트 전체).
`-n 10` = 상위 10개 결과.

### 인덱싱

```bash
qmd update    # 새 파일 인덱싱
qmd embed     # 임베딩 갱신
```

볼트에 새 파일 저장 후 자동 실행 (스킬들이 마지막에 호출).

### 강사 환경 실측

- 인덱싱 파일: 988
- 검색 응답: <1초 (로컬)
- 비용: 0 (네트워크 X, API X)

### 알려진 한계

**한글 chunk 영구 실패 사례** (project_qmd_embed_issue.md):

- QMD 기본 CHUNK_SIZE가 영문 토큰 기준
- 한글 chunk가 2048 토큰 context 초과
- 재시도해도 영구 실패
- **해결:** 모델/chunk 크기 교체만 유효

---

## 3. GBrain (지식 그래프)

### 역할

볼트의 **백링크·타임라인·엔티티 그래프** 검색. QMD가 단순 키워드라면 GBrain은 관계 그래프 기반.

### 백엔드

- Supabase (네트워크 경유)
- RRF 하이브리드 검색 (tsvector + embedding)

### MCP 도구

```
mcp__gbrain__query              일반 쿼리
mcp__gbrain__get_backlinks      백링크 조회
mcp__gbrain__get_timeline       타임라인
mcp__gbrain__get_page           페이지 조회
mcp__gbrain__put_page           페이지 저장
mcp__gbrain__search             검색
mcp__gbrain__traverse_graph     그래프 순회
mcp__gbrain__get_links          링크 조회
mcp__gbrain__add_link           링크 추가
mcp__gbrain__add_tag            태그 추가
mcp__gbrain__add_timeline_entry 타임라인 항목 추가
mcp__gbrain__get_chunks         청크 조회
mcp__gbrain__file_upload        파일 업로드
mcp__gbrain__sync_brain         브레인 동기화
```

### 언제 GBrain 호출?

QMD가 만족하지 못하는 질문:

- "X 페이지 뭐가 link 걸렸지?" → 백링크 조회
- "최근 작성 흐름?" → 타임라인
- "X 엔티티는?" → 엔티티 검색

### 강사 실측

- 응답: ~500ms (네트워크 경유)
- 비용: 거의 0 (자체 호스팅 Supabase)

---

## 4. graphify (모듈 의존성 + 그래프 분석)

### 역할

볼트를 **그래프**로 분석. community detection · god nodes · shortest path 등.

### 데이터

- 로컬 graph.json
- 3386 nodes / 4139 edges / 247 communities
- 주1회 빌드

### MCP 도구

```
mcp__graphify__query_graph      그래프 쿼리
mcp__graphify__get_node         노드 정보
mcp__graphify__get_neighbors    이웃 노드
mcp__graphify__get_community    커뮤니티
mcp__graphify__god_nodes        중심성 높은 노드
mcp__graphify__graph_stats      그래프 통계
mcp__graphify__shortest_path    최단 경로
```

### 언제 graphify 호출?

- "X와 Y 어떻게 연결?" → shortest_path
- "vault 핵심 노드?" → god_nodes
- "이 vault 어떤 cluster 있지?" → get_community
- "노드 중심성?" → graph_stats
- "X 의미 비슷한 페이지?" → qmd 1차 → graphify get_community 2차

### 강사 환경 적용

- POS 시스템 모듈 의존성 시각화
- "어느 모듈이 핵심인가" + "shortest path" 자동 추적
- 새 사고 발생 → graphify shortest_path → 영향받는 모듈 자동 추적

---

## 5. Tier 게이트 룰 (CLAUDE.md Knowledge Hierarchy Protocol)

### 5단계 게이트

```
Tier 1   세션 내                                    무비용
   - CLAUDE.md 자동 주입
   - ~/.claude/memory MEMORY.md
   - 이번 세션 읽은 파일

Tier 2a  30_Claude (협업자 기억) — 자율 발동       무비용
   - replay-learnings 스킬
   - 01_Sessions / 02_Learnings / 04_Plans / 05_Research / 06_Designs

Tier 2b  20_Wiki (복리 지식) — 자율 발동           무비용
   - qmd-search (default) 단순 의미·키워드
   - 988 파일 로컬 임베딩

Tier 2c  20_Wiki (백링크·타임라인) — opt-in        거의 무비용
   - gbrain query
   - graphify (community / god_nodes / shortest_path)

Tier 3   외부 검색 — 비용 발생                     [BILL]
   - context7 (SDK 문서)
   - brave_web_search (일반 웹)
   - firecrawl (특정 페이지)
```

### 진입 원칙

**무비용 → 비용** (qmd → gbrain → graphify). 첫 도구가 만족하면 다음 단계 진입 X.

**Tier 3 진입 전 Tier 2 반드시 통과** (gate).

---

## 6. 발동 트리거

### Strong triggers (반드시 검색)

| 트리거 | 예시 |
|---|---|
| 프로젝트명 언급 | "harness", "Yurisma", "BeautyDecode", "proposal-bid" |
| 외부 기술·도구·사람 고유명사 | "Graphify", "Ouroboros", "Ollama", "카파시" |
| 과거 참조 표현 | "이전에", "예전에", "우리 ~했잖아", "기억나는데" |
| 작업 재개 | "이어서", "다시", "계속" |
| 새 프로젝트/기능 시작 | "Hermes 에이전트 구축 시작하자" |
| Tier 3 사용 전 (gate) | 외부 API 호출 직전 |

### Medium triggers (검색 고려)

| 트리거 | 시점 |
|---|---|
| 설계 결정 직전 | 06_Designs 확인 |
| 계획 수립 시 | 04_Plans 재사용 |
| 리서치 시작 | 05_Research 중복 방지 |
| Task 도구 위임 직전 | 에이전트 위임 전 |

### Skip 조건

| 조건 | 행동 |
|---|---|
| 사용자 "바로", "검색 없이", "vault 무시" 명시 | skip |
| 단순 수학·문법·인사·명확한 사실 | skip |
| 같은 세션 동일 쿼리 이미 검색 | skip |
| QMD 오류·미설치 | silent fallback |

---

## 7. Token Budget (Tier 2 최대 주입량)

| 영역 | 최대 토큰 |
|---|---|
| Tier 2a (30_Claude / replay-learnings) | 1500 |
| Tier 2b (20_Wiki / qmd-search) | 1000 |
| Tier 2c (10_Raw) | 사용자 요청 시만 Read |
| **합계** | **최대 2500 tokens / session start** |

---

## 8. Score 기반 활성화 룰

검색 결과 score에 따른 액션:

| Score | 행동 |
|---|---|
| **≥ 0.9** (High) | 파일 전문 Read 후 핵심 인용 |
| **0.7 ~ 0.9** (Medium-High) | 요약 주입 + "읽어볼까요?" 제안 |
| **0.5 ~ 0.7** (Medium) | 제목만 주입 + "관련 있어 보여요" |
| **< 0.5** (Low) | 조용히 skip, "특별히 관련 지식 없음" 명시 |

---

## 9. 실전 예시 — Hermes 에이전트 구축

```
사용자: "Hermes 에이전트 구축 시작하자"
    ↓
[Trigger 감지: "Hermes" 고유명사 + "구축" 새 작업]
    ↓
[Tier 2a — replay-learnings 자동 발동]
   L1: grep local learn-rule
   L2: qmd search "Hermes agent" -c obsidian -n 10
    ↓
[결과 상위: research-2026-04-14-gbrain-ouroboros-analysis (0.92)]
[score ≥ 0.9 → 전문 Read]
    ↓
Claude 응답:
   "볼트에 이미 Hermes 관련 4개 자료 있습니다:
    - research 문서: GBrain은 Hermes용 인프라, Ollama 루트 주목
    - hermes-agent-install-guide: 18 LLM 지원
    - 2026-04-14 세션: Ouroboros와 비교 분석 완료
    - design 문서: 지식 시스템 활용 원칙
    어느 것부터 보시겠어요?"
```

**효과:** Tier 3 외부 검색 (context7, brave) 진입 X → 비용 0.

---

## 10. 질문 → 도구 자동 라우팅 (B4 룰)

| 질문 유형 | 첫 도구 |
|---|---|
| "이 주제 뭘 알고 있지?" / "X 정의는?" / 단순 의미·키워드 | **qmd-search** (default, 무비용) |
| "X 페이지 뭐가 link 걸렸지?" / "최근 작성 흐름?" | **gbrain query** (백링크·timeline) |
| "X와 Y 어떻게 연결?" / "vault 핵심 노드?" / "최단 경로?" | **graphify** (shortest_path·god_nodes) |
| "이 vault 어떤 cluster 있지?" / "노드 중심성?" | **graphify** (get_community / graph_stats) |
| "X 의미 비슷한 페이지?" | **qmd-search 1차** → graphify get_community 2차 |

---

## 11. 회사 환경 적용 시나리오

### Confluence → QMD 인덱싱

```
Confluence export → 로컬 QMD 인덱싱 → 토큰 0 검색
   1. Confluence 페이지 마크다운 export (스크립트)
   2. qmd update 인덱싱
   3. 사내 지식 988 파일 → 1초 내 검색
   4. Tier 3 외부 API 안 부르고도 답 추출
```

### Jira + Gitlab + Confluence → GBrain 통합

```
Jira 티켓 + Gitlab commit + Confluence 페이지 → GBrain 통합 그래프
   1. 각 소스에서 변경 사항 수집 (Hermes·OpenClaw)
   2. GBrain에 페이지·링크·타임라인 적재
   3. 백링크로 재발 사고 자동 추적
```

### 모듈 의존성 → graphify

```
POS 시스템 모듈 분석:
   - graphify로 community detection → "어느 모듈이 핵심인가" 시각화
   - shortest_path → 영향받는 모듈 자동 추적
   - god_nodes → 변경 시 최대 영향 모듈 식별
```

---

## 12. 이행률 정직 회수

**80~90% 이행** — 자동 100% X.

본 라이브 페이지 작성 세션에서도 QMD/GBrain/graphify를 직접 호출하지 않은 답변이 있었습니다. AI 자율 판단의 한계입니다. CLAUDE.md에 룰 명시 + 시스템 누적으로 이행률은 점진적으로 올라갑니다.

---

## 13. 관련 자료

- [architecture.md](./architecture) — 6 레이어 + 3 Tier
- [memory-vault.md](./memory-vault) — 30_Claude 6 영역 (검색 대상)
- [token-saving.md](./token-saving) — Tier 게이트 = 기법 12
- [hook-chain.md](./hook-chain) — replay-learnings-recall

---

**Last updated:** 2026-05-12 (v1)
