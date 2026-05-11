# Daniel's Claude Harness Catalog

> 강사 본인이 6개월~1년 운영 중인 Claude Code 하네스 시스템 전체 카탈로그.
> 5 도구 + 6 레이어 + 3-Tier + 134 스킬 + MCP 풀스택 + Hook chain + 토큰 절감 12 기법을 한 장에 가시화.

**Live**: https://daniel8824-del.github.io/claude-harness-catalog/

---

## 본 카탈로그의 목적

본 페이지는 강사가 진행하는 **AI 실무 생산성 혁신** 교육의 보조 자료로, 수강생이 강의 후 시스템 전체 구조를 한 장으로 다시 찾아볼 수 있게 만든 카탈로그입니다.

기존 강사 GitHub Pages 3종 (`claude-skill-catalog`, `claude-wiki-verbs`, `claude-feynman-research-skill`) 을 보완하여, 시스템 전체 아키텍처 + 운영 사례 + 토큰 절감 기법까지 한 페이지에 집약한 것이 본 카탈로그의 차별점입니다.

---

## 12 섹션 구성

| # | 섹션 | 내용 |
|---|---|---|
| §1 | 5 도구 포지셔닝 | 코딩 3 (Claude/Codex/oh-my-opencode) + 수집 2 (Hermes/OpenClaw) |
| §2 | 6 레이어 협력 모델 | Superpowers / OMC / gstack / autoresearch / OMO / pro-workflow |
| §3 | 3-Tier 스킬 시스템 | 자동 / 추천 / 통합 |
| §4 | Skills 14 카테고리 | 글로벌 134종 분류 |
| §5 | MCP 풀스택 | 회사 6 + 강사 추가 8 |
| §6 | Hook chain 5종 + RTK | 60-90% 토큰 절감 |
| §7 | 토큰 절감 12 기법 | [A] Hook · [B] 구조 · [C] 라우팅 · [D] 워크플로우 |
| §8 | 메모리·볼트 | ~/.claude/memory + Obsidian 30_Claude 6 영역 |
| §9 | QMD / GBrain / graphify | 사내 지식 무비용 검색 3종 |
| §10 | 세션 라이프사이클 | replay → 작업 → learn → handoff → retro |
| §11 | 운영 사례 6건 | harness · Yurisma · BeautyDecode · proposal-bid · Hermes · QMD |
| §12 | 4 hub 링크 | 강사 GitHub Pages 4종 |

---

## 강사 운영 자산 (수치)

```
글로벌 ~/.claude/skills/         134종
플러그인 캐시                    157종
MEMORY.md 인덱스                 80+ 항목
30_Claude/02_Learnings 누적      310+ entries
QMD 인덱싱                       988 파일
graphify 그래프                  3386 nodes / 247 communities
RTK Hook 절감                    60-90% (강사 실측)
```

---

## 본 카탈로그가 기반하는 외부 자산

본 시스템은 다음 외부 자산 위에 강사의 운영 룰을 누적한 결과입니다. 자세한 출처는 [attribution.md](./attribution.md) 참조.

| 레이어 | GitHub | Stars |
|---|---|---|
| Superpowers | obra/superpowers | ★167K |
| OMC | Yeachan-Heo/oh-my-claudecode | ★31K |
| gstack | garrytan/gstack | ★82K |
| autoresearch | karpathy/autoresearch | ★76K |
| OMO | code-yeongyu/oh-my-openagent | ★54K |
| pro-workflow | rohitg00/pro-workflow | ★2K |

**외부 CLI**: tobi/qmd ★23K · getcompanion-ai/feynman ★5.8K

---

## 강의 활용 안내

본 카탈로그는 강의 후 수강생에게 1장 인쇄물로도 배포됩니다. 강의 1일차 4차시 마무리에 카탈로그 hub로 안내됩니다.

**관련 강의 자료**:
- `AI_실무_생산성_혁신_Day1_수업자료.md` (강사용 진행 스크립트)
- `회사_협의용_5일_매트릭스.md` (강의 1주 전 사전 협의문)

---

## 배포 방법

```bash
# 로컬에서 git 초기화
cd /mnt/c/Users/daniel/Desktop/claude-harness-catalog
git init
git add .
git commit -m "init: claude-harness-catalog v1"

# GitHub repo 생성 + push
gh repo create claude-harness-catalog --public --source . --push

# GitHub Pages 활성화 (docs/ 폴더 사용)
gh api -X POST repos/daniel8824-del/claude-harness-catalog/pages \
  -f 'source[branch]=main' -f 'source[path]=/docs'

# 약 3분 후 라이브:
# https://daniel8824-del.github.io/claude-harness-catalog/
```

---

## 라이센스 및 출처

본 카탈로그는 외부 6 레이어 + 2 CLI의 조합 위에 강사 본인의 운영 룰을 누적한 결과입니다. 각 외부 자산의 라이센스와 출처는 [attribution.md](./attribution.md)에 명시되어 있습니다.

본 카탈로그 자체는 강사 본인의 운영 자산 가시화이며, 본인 GitHub Pages에서 자유 열람 가능합니다.

---

**v1 · 2026-05-12 · Daniel (daniel8824)**
