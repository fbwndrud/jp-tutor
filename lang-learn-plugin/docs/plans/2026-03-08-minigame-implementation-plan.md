# 미니게임 시스템 구현 계획

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** 10개 미니게임을 일본어 학습 플러그인에 통합하여, 학습과학 기반 SRS 연동 + 스마트 추천 + 4단계 난이도 시스템을 구축한다.

**Architecture:** `/jp:game` 단일 진입점 → 추천 시스템이 최적 게임 선택 → 개별 HTML 템플릿으로 렌더링. 기존 progress.json에 minigames 필드 추가, correct 필드를 float로 확장. 게임 결과는 가중치(0.4~1.0)에 따라 SRS에 차등 반영.

**Tech Stack:** HTML5/CSS3/Vanilla JS (Kawaii Stationery 디자인 시스템), Canvas API (필기/그리기 게임), Web Speech API (TTS), Claude Cowork plugin 포맷 (YAML frontmatter + Markdown commands)

**Active Directory:** `jp-tutor/` (v0.7.0)

**Design Docs:**
- 통합 설계: `lang-learn-plugin/docs/plans/2026-03-08-minigame-system-design.md`
- 난이도/추천: `lang-learn-plugin/docs/plans/2026-03-08-minigame-progression-system.md`
- 학습과학: `lang-learn-plugin/docs/plans/2026-03-08-minigame-learning-science.md`

---

## Phase 0: 인프라 (추천 시스템 + 커맨드 + 스키마)

### Task 1: 게임 추천 알고리즘 레퍼런스 문서

**Files:**
- Create: `jp-tutor/skills/jp/references/game-recommendation.md`

**Context:** Claude가 게임 추천 시 참조하는 문서. 5가지 트리거, 신선도 감쇠, 프로필 보너스, 해금 테이블, 실패/성공 라우팅을 모두 포함. SKILL.md에서 `이 파일을 읽어라`로 참조됨.

**Step 1: 레퍼런스 파일 작성**

```markdown
---
# 게임 추천 알고리즘

## 해금 테이블

| 게임 | key | 해금 조건 |
|------|-----|-----------|
| 매칭 카드 | matching_card | Pre-N5 즉시 |
| 셔플 퍼즐 | shuffle_puzzle | Pre-N5 즉시 |
| 따라 쓰기 | stroke_practice | Pre-N5 즉시 |
| 스네이크 매치 | snake_match | N5 즉시 |
| 한자 조립 | radical_assembly | N5 즉시 |
| 파티클 배틀 | particle_battle | N5 즉시 |
| 끝말잇기 | shiritori | N5 즉시 |
| 듣고 그리기 | listen_draw | N5 + 따라쓰기 히라가나 20자 합격 |
| 경어 변환 | keigo_transformer | N4 즉시 |
| 문맥 추리 | context_detective | N4 즉시 |

## 게임별 가중치 (SRS 반영)

| 게임 | 가중치 | 인출 유형 |
|------|--------|-----------|
| matching_card | 0.4 | 재인(recognition) |
| snake_match | 0.4 | 재인 + 시각탐색 |
| shiritori | 0.6 | 자유회상 + 음운처리 |
| shuffle_puzzle | 0.7 | 순서 재구성 |
| radical_assembly | 0.7 | 분석적 재구성 |
| context_detective | 0.7 | 추론 + 단서회상 |
| stroke_practice | 0.8 | 운동기억 인출 |
| particle_battle | 0.8 | 생성(cued recall) |
| listen_draw | 0.9 | 청각인출 + 운동생성 |
| keigo_transformer | 1.0 | 변환 + 생성 |

## 추천 점수 계산

최종 점수 = 트리거 점수 + 신선도 점수 + 프로필 점수

### 트리거 1: 새 개념 학습 직후
발동: 신규 3개+ 학습 직후

| 학습 내용 | 추천 게임 | 점수 |
|-----------|-----------|------|
| 새 어휘 | matching_card | +80 |
| 새 어휘 | snake_match | +70 |
| 새 어휘 | shiritori | +50 |
| 새 문법(조사) | particle_battle | +80 |
| 새 문법(어순) | shuffle_puzzle | +70 |
| 새 한자 | radical_assembly | +90 |
| 새 한자 | stroke_practice | +75 |
| 새 경어 | keigo_transformer | +90 |

### 트리거 2: 복습 큐 과적
발동: nextReview ≤ 오늘 10개+

| 복습 대상 | 추천 게임 | 점수 |
|-----------|-----------|------|
| 어휘 10개+ | matching_card | +70 |
| 어휘 10개+ | snake_match | +65 |
| 어휘 15개+ | shiritori | +60 |
| 한자 5개+ | radical_assembly | +75 |
| 한자 5개+ | stroke_practice | +65 |
| 문법 5개+ | particle_battle | +70 |
| 문법 5개+ | context_detective | +65 |

복습 20개+ 시 전체 점수 +20

### 트리거 3: 반복 피로
발동: 최근 3세션 연속 같은 모드

| 조건 | 추천 게임 | 점수 |
|------|-----------|------|
| 어휘 퀴즈 3연속 | snake_match | +85 |
| 어휘 퀴즈 3연속 | shiritori | +80 |
| 문법 퀴즈 3연속 | context_detective | +85 |
| 문법 퀴즈 3연속 | shuffle_puzzle | +75 |
| 한자 플래시카드 3연속 | radical_assembly | +85 |
| 한자 플래시카드 3연속 | listen_draw | +75 |

### 트리거 4: 약점 타겟팅
발동: 특정 카테고리 빨강 배지 3개+

| 약점 | 추천 게임 | 점수 |
|------|-----------|------|
| 조사 정답률 40%- | particle_battle | +95 |
| 한자 읽기 40%- | radical_assembly | +90 |
| 한자 쓰기 40%- | stroke_practice | +90 |
| 경어 40%- | keigo_transformer | +95 |
| 어순 오류 빈번 | shuffle_puzzle | +90 |
| 어휘 recall 낮음 | matching_card | +85 |
| 문맥 파악 약함 | context_detective | +90 |
| 청취 약함 | listen_draw | +85 |

### 트리거 5: 데일리 마무리
/jp:daily 세션 마지막에 항상 1게임 추천.
1순위: 약점 게임 (+100), 2순위: 복습 게임 (+90), 3순위: 7일+ 미플레이 랜덤 (+70)

### 신선도 점수
- 7일+ 미플레이: +20
- 3-6일 전: +10
- 1-2일 전: +0
- 오늘 1회: -30
- 오늘 2회+: -60

### 프로필 보너스
| 프로필 | 보너스 게임 | 추가 점수 |
|--------|------------|-----------|
| jlpt | matching_card, particle_battle, context_detective | +15 |
| business | keigo_transformer, context_detective | +25 |
| textbook | shuffle_puzzle, stroke_practice | +15 |

## 추천 출력 형식

```
🎮 게임 추천
━━━━━━━━━━━━━━━━━━
추천: [게임 이름]
이유: [1줄 설명]
예상 소요: [시간]분
난이도: 티어 [N] ([난이도명])
━━━━━━━━━━━━━━━━━━
시작하시겠어요? (Y/다른 게임 보기)
```

## 실패 기반 라우팅

| 실패 게임 | 실패 패턴 | 보완 게임 |
|-----------|-----------|-----------|
| listen_draw | 유사도 50%- 반복 | stroke_practice |
| snake_match | 같은 쌍 반복 오류 | matching_card |
| shiritori | 5턴 이내 탈락 3회 | matching_card 또는 snake_match |
| context_detective | 조사 빈칸 반복 오답 | particle_battle |
| shuffle_puzzle | 어순 배열 반복 실패 | particle_battle |
| radical_assembly | 부수 위치 반복 오류 | stroke_practice |
| particle_battle | 문맥 파악 실패 (T3-4) | context_detective |
| keigo_transformer | 존경어/겸양어 혼동 | matching_card (경어 쌍) |

## 성공 기반 라우팅

| 성공 게임 | 조건 | 상위 게임 |
|-----------|------|-----------|
| matching_card | 티어3+ 3연승 | snake_match |
| stroke_practice | 티어3 성공률 80%+ | listen_draw |
| particle_battle | 티어3+ 정답률 85%+ | context_detective |
| matching_card + snake_match | 둘 다 티어2+ | shiritori |
| radical_assembly | 티어3 체인 5연속 | stroke_practice (한자) |

## 게이밍 방지

1. 동일 개념 동일 게임 24시간 쿨다운
2. 마스터(파랑) 개념 게임 XP 50% 감소
3. 세션 아이템 30%는 약점 개념 강제 배치
4. 배지 확정에 2가지+ 평가 방식 필요
5. SRS 7d+ 승급에 가중치 0.7+ 게임 필요 (승급 검증)
```

**Step 2: 검증**

파일이 생성되었는지, 마크다운 문법이 올바른지 확인합니다.

**Step 3: 커밋**

```bash
git add jp-tutor/skills/jp/references/game-recommendation.md
git commit -m "feat: add game recommendation algorithm reference"
```

---

### Task 2: 미니게임 규칙 레퍼런스 문서

**Files:**
- Create: `jp-tutor/skills/jp/references/minigame-rules.md`

**Context:** 10개 게임의 규칙, 4티어 난이도 곡선, XP 보상, 세션 시간, 인지부하 프로필을 정의. Claude가 게임 세션 진행 시 이 문서를 참조.

**Step 1: 레퍼런스 파일 작성**

설계서(`2026-03-08-minigame-system-design.md`)의 섹션 4(난이도 곡선)와 섹션 7(인지부하 관리)을 기반으로 작성합니다.

내용 포함:
- 10개 게임 각각의 4티어 파라미터 테이블 (카드 수, 시간, 특수 규칙)
- 티어 자동 배정 규칙 (최근 3판 승률 기준)
- XP 보상 테이블 (기본 + 보너스)
- 세션 시간 설계 (라운드 소요, 기본/최대 라운드)
- 자동 보호 장치 (15분, 30분, 3연패)
- 게임별 인지부하 프로필
- 세션 구성 NEW:REVIEW 비율 (레벨별)
- 안티패턴 방지 규칙 7가지

**Step 2: 검증**

**Step 3: 커밋**

```bash
git add jp-tutor/skills/jp/references/minigame-rules.md
git commit -m "feat: add minigame rules, difficulty curves, and XP tables"
```

---

### Task 3: `/jp:game` 커맨드 파일

**Files:**
- Create: `jp-tutor/commands/jp-game.md`

**Context:** 게임 진입 커맨드. 기존 커맨드 포맷(YAML frontmatter + 마크다운)을 따름. 추천 시스템을 호출하거나 유저가 직접 게임을 선택. `jp-vocab.md` 형식을 참조.

**Step 1: 커맨드 파일 작성**

```markdown
---
description: "미니게임으로 일본어를 재미있게 복습합니다. 매칭 카드, 셔플 퍼즐, 파티클 배틀 등 10가지 게임."
allowed-tools: ["Read", "Write", "Edit", "Glob", "Grep", "Bash"]
model: sonnet
argument-hint: "[게임이름 또는 빈칸으로 추천받기]"
---

# 미니게임 (/jp:game)

## 사전 준비

1. `skills/jp/SKILL.md` 스킬 파일을 읽어 튜터 역할을 숙지하세요.
2. `jp-data/progress.json`을 읽으세요.
   - 파일이 없으면: "아직 초기 설정이 되어 있지 않습니다. `/jp:start`로 시작해주세요!" 라고 안내하고 종료합니다.
3. `skills/jp/references/game-recommendation.md`를 읽으세요.
4. `skills/jp/references/minigame-rules.md`를 읽으세요.

## 게임 선택

### 직접 선택
인자가 있으면 해당 게임으로 바로 시작합니다:
- "매칭", "매칭카드", "matching" → matching_card
- "셔플", "퍼즐", "shuffle" → shuffle_puzzle
- "스네이크", "snake" → snake_match
- "따라쓰기", "쓰기", "stroke" → stroke_practice
- "듣고그리기", "listen" → listen_draw
- "한자조립", "radical" → radical_assembly
- "파티클", "조사", "particle" → particle_battle
- "경어", "keigo" → keigo_transformer
- "문맥", "추리", "context" → context_detective
- "끝말잇기", "しりとり", "shiritori" → shiritori

### 추천 모드 (인자 없음)
game-recommendation.md의 알고리즘에 따라 추천 점수를 계산합니다.
상위 2개 게임을 추천 형식으로 표시합니다.
유저가 선택하면 해당 게임을 시작합니다.

## 해금 확인

선택된 게임의 해금 조건을 game-recommendation.md의 해금 테이블에서 확인합니다.
- 해금되지 않았으면: "이 게임은 [조건]을 달성하면 해금됩니다!" 라고 안내.
- 해금되었으면: 게임 시작.

## 게임 세션 진행

### 첫 플레이 시
규칙 설명을 표시합니다. (minigame-rules.md 참조)

### 난이도 결정
progress.json의 minigames.stats에서 해당 게임의 current_tier를 확인합니다.
stats가 없으면 tier 1로 시작합니다.

### 아이템 선정
게임에 사용할 학습 아이템을 progress.json에서 선정합니다:
- 70%: 시스템 추천 (복습 대상, 약점 개념 우선)
- 30%: 현재 레벨 커리큘럼의 적절한 개념

### HTML 게임 렌더링
해당 게임의 HTML 템플릿을 읽어 DATA를 주입하고 렌더링합니다.
- 템플릿 경로: `skills/jp/references/templates/game-{key}.html`
- DATA 주입 패턴: `const DATA = null;` → `const DATA = { ... };`

### DATA 구조 (공통)
```json
{
  "gameKey": "matching_card",
  "tier": 2,
  "tierName": "기본",
  "items": [...],
  "timeLimit": 50,
  "maxMisses": 10,
  "roundNumber": 1,
  "totalRounds": 3,
  "playerLevel": 7,
  "levelTitle": "도전자"
}
```

각 게임별 items 형식은 게임 템플릿에서 정의됩니다.

### 텍스트 기반 게임 (HTML 템플릿 없는 게임)
끝말잇기(shiritori), 경어 변환(keigo_transformer), 문맥 추리(context_detective)는
HTML 대신 대화형으로 진행합니다. Claude가 직접 문제를 내고 채점합니다.

## 결과 처리

### progress.json 업데이트
게임 결과에 따라 progress.json을 업데이트합니다:

1. **개념별 SRS 업데이트**:
   - 정답: `concept.attempts += 1`, `concept.correct += gameWeight` (float)
   - 오답: `concept.attempts += 1`, `concept.correct += 0`
   - nextReview 갱신:
     - 정답 + 가중치 >= 0.7 → 다음 간격으로 (1d→3d→7d→14d→30d→60d)
     - 정답 + 가중치 < 0.7 → 간격 유지
     - 오답 → 1d로 리셋
   - 승급 검증: 간격 7d+ 승격에는 가중치 0.7+ 게임/퀴즈에서 최소 1회 정답 필요

2. **미니게임 통계**:
   ```json
   minigames.stats.{gameKey}.plays += 1
   minigames.stats.{gameKey}.last_played = 오늘
   minigames.stats.{gameKey}.recent_scores = [최근3판 승률]
   minigames.stats.{gameKey}.total_xp_earned += earnedXP
   ```

3. **티어 변동**:
   - recent_scores 3개 모두 90%+ → current_tier += 1 (최대 4)
   - recent_scores 3개 모두 50%- → current_tier -= 1 (최소 1)
   - best_tier 갱신

4. **XP**: minigame-rules.md의 XP 보상 테이블에 따라 XP 부여
   - 마스터(파랑) 개념은 XP 50% 감소

5. **게이밍 방지**:
   - today_plays, today_minutes 업데이트
   - 30분 초과 시 학습 복귀 권유

### 세션 결과 표시

```
🎮 게임 결과
━━━━━━━━━━━━━━━━━━
[게임 이름] - 티어 {N} ({난이도명})
정답: {correct}/{total} ({rate}%)
획득 XP: +{xp}
{티어 변동 메시지}
━━━━━━━━━━━━━━━━━━
```

### 다음 행동 추천
- 실패 패턴 감지 시: 보완 게임 추천 (game-recommendation.md 실패 라우팅 참조)
- 성공 패턴 감지 시: 상위 게임 추천 (game-recommendation.md 성공 라우팅 참조)
- 그 외: "계속 플레이" 또는 "학습으로 돌아가기" 선택지

## 언어 규칙
- 모든 설명과 피드백은 **한국어**로 제공합니다.
- 게임 내 일본어 콘텐츠에는 N4 이하 학습자에게 후리가나를 병기합니다.
```

**Step 2: 검증**

기존 커맨드 포맷과 일치하는지 확인 (YAML frontmatter, model, allowed-tools).

**Step 3: 커밋**

```bash
git add jp-tutor/commands/jp-game.md
git commit -m "feat: add /jp:game command with smart recommendation routing"
```

---

### Task 4: SKILL.md 수정 — 게임 라우팅 추가

**Files:**
- Modify: `jp-tutor/skills/jp/SKILL.md`

**Step 1: 세션 라우팅에 게임 추가**

SKILL.md의 `## 세션 라우팅` 섹션에 게임 라우팅을 추가합니다:

기존 라우팅 목록 (line 24~37 부근)에 추가:
```
   - "게임", "미니게임", "매칭", "퍼즐" → game 모드
```

**Step 2: progress.json 스키마에 minigames 필드 추가**

`### progress.json 구조` 섹션 (line 52~83)의 JSON에 추가:

```json
  "minigames": {
    "unlocked": [],
    "stats": {},
    "daily_game_played": false,
    "total_game_minutes_today": 0
  }
```

**Step 3: correct 필드 설명 수정**

기존 progress.json 예시의 `"correct": 3` 주석에 `(float, 게임 가중치 반영)` 추가.

**Step 4: 참조 파일에 게임 문서 추가**

`## 참조 파일` 섹션 (line 175~182)에 추가:
```
- **게임 추천 시**: `skills/jp/references/game-recommendation.md` 를 읽고 추천 알고리즘을 따릅니다.
- **게임 규칙 참조 시**: `skills/jp/references/minigame-rules.md` 를 읽고 난이도/XP를 적용합니다.
```

**Step 5: 게이미피케이션 XP 테이블에 게임 XP 추가**

`### XP 보상 테이블` (line 187~197)에 추가:
```
| 미니게임 완료 (티어 평균) | +15~50 (티어별) |
| 미니게임 퍼펙트/보너스 | +10~20 |
```

**Step 6: 검증**

SKILL.md가 500줄 이내인지 확인합니다.

**Step 7: 커밋**

```bash
git add jp-tutor/skills/jp/SKILL.md
git commit -m "feat: add minigame routing, schema, and references to SKILL.md"
```

---

### Task 5: jp-daily.md 수정 — 마무리 게임 추천 삽입

**Files:**
- Modify: `jp-tutor/commands/jp-daily.md`

**Step 1: 세션 흐름에 게임 추천 삽입**

`## 세션 종료 메시지` 섹션 직전에 추가:

```markdown
## 마무리 미니게임 추천

학습 세션이 완료되면 마무리 미니게임 1판을 추천합니다.

1. `skills/jp/references/game-recommendation.md`를 읽고 트리거 5(데일리 마무리) 로직을 적용합니다.
2. 추천 게임을 표시합니다:
   ```
   🎮 오늘의 마무리 게임!
   추천: [게임 이름]
   이유: [1줄]
   시작하시겠어요? (Y/N)
   ```
3. Y 선택 시 → `/jp:game [게임이름]`을 실행합니다.
4. N 선택 시 → 세션 종료 메시지로 이동합니다.
```

**Step 2: 커밋**

```bash
git add jp-tutor/commands/jp-daily.md
git commit -m "feat: add end-of-session minigame recommendation to jp-daily"
```

---

### Task 6: CLAUDE.md 업데이트 — 게임 템플릿 목록

**Files:**
- Modify: `CLAUDE.md` (루트)

**Step 1: 템플릿 목록 업데이트**

`### 3. 새 기능은 모든 관련 템플릿에 반영` 섹션의 템플릿 목록을 업데이트합니다.

기존 4개 템플릿 아래에 게임 템플릿 추가:
```
- 게임 템플릿 (Kawaii Stationery 디자인 공유):
  - `jp-tutor/skills/jp/references/templates/game-match.html`
  - `jp-tutor/skills/jp/references/templates/game-shuffle.html`
  - `jp-tutor/skills/jp/references/templates/game-snake.html`
  - `jp-tutor/skills/jp/references/templates/game-stroke.html`
  - `jp-tutor/skills/jp/references/templates/game-listen.html`
  - `jp-tutor/skills/jp/references/templates/game-radical.html`
  - `jp-tutor/skills/jp/references/templates/game-particle.html`
```

**Step 2: 프로젝트 구조에 게임 관련 추가**

```
  skills/jp/references/templates/ ← HTML 템플릿 (4개 기존 + 7개 게임)
  skills/jp/references/game-recommendation.md
  skills/jp/references/minigame-rules.md
```

**Step 3: 커밋**

```bash
git add CLAUDE.md
git commit -m "feat: update CLAUDE.md with game template list and structure"
```

---

## Phase 1: 코어 게임 3개 (매칭카드, 파티클배틀, 셔플퍼즐)

### Task 7: game-match.html — 매칭 카드 게임

**Files:**
- Create: `jp-tutor/skills/jp/references/templates/game-match.html`

**Context:** 가장 기본적인 게임. 카드를 뒤집어 짝을 맞추는 메모리 게임. 인코딩(재인) 단계에서 사용. 가중치 0.4.

**Design:** Kawaii Stationery 디자인 시스템 적용. `flashcard.html`의 CSS 변수, 폰트, 배경 패턴을 그대로 복사하여 시작.

**DATA 구조:**
```json
{
  "gameKey": "matching_card",
  "tier": 1,
  "tierName": "입문",
  "timeLimit": 60,
  "maxMisses": -1,
  "pairs": [
    { "front": "食べる", "back": "먹다", "concept": "食べる" },
    { "front": "飲む", "back": "마시다", "concept": "飲む" },
    { "front": "見る", "back": "보다", "concept": "見る" },
    { "front": "聞く", "back": "듣다", "concept": "聞く" }
  ],
  "showPreview": true,
  "previewTime": 1,
  "shuffleAfter": -1,
  "hideRandomFront": false,
  "roundNumber": 1,
  "totalRounds": 3
}
```

**티어별 파라미터 (DATA로 주입):**
| 티어 | pairs 수 | timeLimit | maxMisses | showPreview | shuffleAfter | hideRandomFront |
|------|----------|-----------|-----------|-------------|--------------|-----------------|
| 1 | 4 | 60 | -1 | true (1초) | -1 | false |
| 2 | 6 | 50 | 10 | false | -1 | false |
| 3 | 8 | 45 | 8 | false | 3 | false |
| 4 | 10 | 40 | 6 | false | 3 | true (일부) |

**기능 요구사항:**
1. 카드 그리드 (pairs x 2 장, 셔플)
2. 카드 뒤집기 애니메이션 (CSS 3D transform, flashcard.html 참조)
3. 2장 선택 후 매칭 판정 (0.5초 대기)
4. 매칭 성공: 카드 사라지기 (fade out + scale down)
5. 매칭 실패: 카드 다시 뒤집기 + 미스 카운터 증가
6. 타이머 (상단 프로그레스 바, 시간 경과에 따라 줄어듦)
7. showPreview: 게임 시작 전 모든 카드 앞면 previewTime초 표시 후 뒤집기
8. shuffleAfter: N매칭 성공 후 남은 카드 위치 셔플 (애니메이션)
9. hideRandomFront: 일부 카드 앞면을 "?" 표시 (티어4)
10. 결과 화면: 정답 수/총 수, 시간, 미스 수, XP
11. 키보드: 방향키로 카드 선택, Enter로 뒤집기

**Step 1: HTML 템플릿 작성**

- `flashcard.html`에서 CSS 변수 블록 (`:root`, `@media (prefers-color-scheme: dark)`, `body` 스타일)을 복사
- `const DATA = null;` 패턴 사용
- `;(function() { ... })();` IIFE 패턴
- 카드 그리드: CSS Grid `grid-template-columns: repeat(auto-fill, minmax(120px, 1fr))`
- 카드: `.card` > `.card-inner` > `.card-front` + `.card-back` (3D flip)
- 타이머: 상단 프로그레스 바 (accent 색상, 시간 비례 width)
- 미스 카운터: 하트 아이콘 (❤️) maxMisses개, 미스 시 하나씩 회색으로
- 결과 화면: 퍼펙트 시 별 이펙트 ⭐

**Step 2: 브라우저에서 DATA를 직접 넣어 동작 테스트**

`const DATA = null;`을 위의 예시 DATA로 교체하고 파일을 열어 게임이 동작하는지 확인.

**Step 3: 커밋**

```bash
git add jp-tutor/skills/jp/references/templates/game-match.html
git commit -m "feat: add matching cards minigame template"
```

---

### Task 8: game-particle.html — 파티클 배틀 게임

**Files:**
- Create: `jp-tutor/skills/jp/references/templates/game-particle.html`

**Context:** 문장 빈칸에 올바른 조사를 선택/입력하는 게임. 인출(cued recall) 단계. 가중치 0.8.

**DATA 구조:**
```json
{
  "gameKey": "particle_battle",
  "tier": 1,
  "tierName": "입문",
  "problems": [
    {
      "sentence": "私___学校___行きます",
      "blanks": [
        { "position": 0, "answer": "は", "choices": ["は", "が", "を", "に"], "concept": "は_topic" },
        { "position": 1, "answer": "に", "choices": ["に", "で", "へ", "を"], "concept": "に_destination" }
      ],
      "translation": "나는 학교에 갑니다",
      "explanation": "は: 주제 표시, に: 이동 목적지"
    }
  ],
  "timePerProblem": -1,
  "showExplanationOnWrong": true,
  "inputMode": "choice"
}
```

**티어별 파라미터:**
| 티어 | 문장 길이 | blanks 수 | inputMode | timePerProblem | choices 수 |
|------|-----------|-----------|-----------|----------------|-----------|
| 1 | 단문 5-8자 | 1 | "choice" | -1 | 4 |
| 2 | 단복문 8-15자 | 1-2 | "choice" | -1 | 4 |
| 3 | 복문 15-25자 | 2-3 | "choice" | 15 | 6 |
| 4 | 장문 25자+ | 3-4 | "input" | 12 | - |

**기능 요구사항:**
1. 문장 표시: 빈칸을 `___` 표시, 현재 활성 빈칸 하이라이트
2. 선택지 모드: pill 형태 버튼 4-6개 (accent 색상)
3. 자유 입력 모드 (티어4): 텍스트 입력 필드
4. 정답 시: 빈칸에 초록색으로 정답 삽입 + 짧은 성공 애니메이션
5. 오답 시: 빈칸 빨간 깜빡임 + showExplanationOnWrong이면 3초 해설 표시 (다음 버튼 3초 후 활성화)
6. 타이머 (티어 3-4): 문제당 카운트다운
7. 문제 간 전환: 슬라이드 애니메이션
8. 연속 정답 카운터: 5연속 +10 XP, 10연속 +25 XP
9. 결과 화면: 문제별 ⭕/❌, 총 정답률, XP

**Step 1: HTML 템플릿 작성**

game-match.html에서 CSS 변수/폰트/배경을 복사하여 시작합니다.

**Step 2: 테스트 DATA로 브라우저 동작 확인**

**Step 3: 커밋**

```bash
git add jp-tutor/skills/jp/references/templates/game-particle.html
git commit -m "feat: add particle battle minigame template"
```

---

### Task 9: game-shuffle.html — 셔플 퍼즐 게임

**Files:**
- Create: `jp-tutor/skills/jp/references/templates/game-shuffle.html`

**Context:** 흩어진 단어 타일을 올바른 순서로 배열하는 게임. 인출(어순 재구성). 가중치 0.7.

**DATA 구조:**
```json
{
  "gameKey": "shuffle_puzzle",
  "tier": 1,
  "tierName": "입문",
  "problems": [
    {
      "tiles": ["は", "私", "学生", "です"],
      "answer": ["私", "は", "学生", "です"],
      "dummies": [],
      "hiddenTiles": [],
      "translation": "저는 학생입니다",
      "concept": "basic_sentence_sov"
    }
  ],
  "previewTime": 5,
  "timeLimit": 30,
  "hintCount": 1
}
```

**티어별 파라미터:**
| 티어 | tiles 수 | previewTime | timeLimit | dummies | hiddenTiles | hintCount |
|------|----------|-------------|-----------|---------|-------------|-----------|
| 1 | 3 | 5 | 30 | 0 | 0 | 1 |
| 2 | 5 | 4 | 25 | 0 | 0 | 0 |
| 3 | 7 | 3 | 25 | 1 | 0 | 0 |
| 4 | 9 | 2 | 20 | 2 | 일부 | 0 |

**기능 요구사항:**
1. 타일 영역: 상단에 셔플된 타일들 (드래그 가능), 하단에 정답 슬롯
2. 정답 미리보기: previewTime초간 올바른 순서 표시 후 셔플
3. 드래그 앤 드롭: 타일을 슬롯으로 끌어다 놓기 (터치 지원)
4. 클릭으로도 배치: 타일 클릭 → 다음 빈 슬롯에 배치
5. 슬롯에서 타일 제거: 배치된 타일 클릭 시 다시 풀로 복귀
6. 더미 타일 (티어 3-4): 정답에 포함되지 않는 타일 섞기
7. 숨김 타일 (티어 4): 일부 타일 뒷면만 보여줌 (클릭하면 잠시 보임)
8. 힌트 (티어 1): 첫 타일 위치 하이라이트
9. 제출 버튼: 모든 슬롯 채워지면 활성화
10. 정답 판정 후 결과 표시
11. 타이머: 프로그레스 바

**Step 1: HTML 템플릿 작성**

**Step 2: 테스트 DATA로 브라우저 동작 확인**

**Step 3: 커밋**

```bash
git add jp-tutor/skills/jp/references/templates/game-shuffle.html
git commit -m "feat: add shuffle puzzle minigame template"
```

---

### Task 10: dashboard.html 수정 — 미니게임 통계 섹션

**Files:**
- Modify: `jp-tutor/skills/jp/references/templates/dashboard.html`

**Context:** 기존 대시보드에 미니게임 통계 섹션 추가. 해금된 게임 목록, 각 게임의 티어/플레이 수/최근 성적, 오늘의 게임 시간.

**Step 1: DATA 구조 확장**

기존 DATA에 minigames 필드 추가:
```json
{
  "minigames": {
    "unlocked": ["matching_card", "shuffle_puzzle", "stroke_practice"],
    "stats": {
      "matching_card": {
        "plays": 15,
        "current_tier": 3,
        "best_tier": 3,
        "recent_scores": [85, 90, 75],
        "total_xp_earned": 340,
        "last_played": "2026-03-08"
      }
    },
    "total_game_minutes_today": 12
  }
}
```

**Step 2: 미니게임 통계 섹션 HTML/CSS/JS 추가**

대시보드의 기존 섹션들(배지 분포, 복습 알림, 약점) 아래에 새 섹션:

- 제목: "🎮 미니게임"
- 해금된 게임 카드 그리드 (각 게임: 이름, 현재 티어 ★표시, 최근 3판 미니 차트, 플레이 수)
- 미해금 게임: 잠금 아이콘 + 해금 조건 툴팁
- 오늘의 게임 시간: 프로그레스 바 (30분 기준)

**Step 3: 커밋**

```bash
git add jp-tutor/skills/jp/references/templates/dashboard.html
git commit -m "feat: add minigame stats section to dashboard"
```

---

### Task 11: Phase 1 통합 커밋 — 버전 범프

**Files:**
- Modify: `jp-tutor/.claude-plugin/plugin.json` — version 0.7.0 → 0.8.0
- Modify: `.claude-plugin/marketplace.json` — version 0.7.0 → 0.8.0
- Modify: `README.md` — 미니게임 섹션 추가

**Step 1: 버전 업데이트 + README 업데이트**

**Step 2: 커밋**

```bash
git add jp-tutor/.claude-plugin/plugin.json .claude-plugin/marketplace.json README.md
git commit -m "feat: v0.8.0 — Phase 1 minigames (matching, particle, shuffle)"
```

---

## Phase 2: Canvas 게임 (따라 쓰기, 듣고 그리기)

### Task 12: game-stroke.html — 따라 쓰기 게임

**Files:**
- Create: `jp-tutor/skills/jp/references/templates/game-stroke.html`

**Context:** Canvas API로 글자 필기 연습. 운동기억 인코딩→인출. 가중치 0.8.

**DATA 구조:**
```json
{
  "gameKey": "stroke_practice",
  "tier": 1,
  "tierName": "따라쓰기",
  "characters": [
    {
      "char": "あ",
      "reading": "a",
      "strokes": [[...]],
      "concept": "hiragana_a"
    }
  ],
  "guideMode": "dotted",
  "strokeOrderCheck": false,
  "similarityThreshold": 60,
  "showPreviewTime": -1
}
```

**티어별 파라미터:**
| 티어 | guideMode | strokeOrderCheck | similarityThreshold | showPreviewTime |
|------|-----------|------------------|---------------------|-----------------|
| 1 | "dotted" (점선+번호) | false | 60 | -1 (항상) |
| 2 | "translucent" (반투명) | true | 70 | -1 |
| 3 | "blank" (빈 캔버스) | true | 75 | 3초 |
| 4 | "memory" (글자 없이 뜻만) | true | 80 | 0 |

**기능 요구사항:**
1. Canvas 영역 (300x300px): 필기 입력
2. 포인터/터치 이벤트로 획 그리기 (부드러운 선)
3. 가이드 모드별 배경 표시 (점선, 반투명, 없음)
4. 획순 번호 표시 (티어 1)
5. "다시 쓰기" 버튼: Canvas 초기화
6. 유사도 판정: 사용자 획과 정답 획의 경로 비교 (간단한 DTW 또는 영역 비교)
7. 합격/불합격 표시 + 정답 획순 애니메이션 재생
8. 10자 연속 성공 보너스 표시

**핵심 기술 구현 — 필기 유사도 판정:**
Canvas에서 사용자가 그린 이미지와 정답 이미지의 픽셀 오버랩 비율로 간단 판정:
```js
function calculateSimilarity(userCanvas, answerCanvas) {
  var userCtx = userCanvas.getContext('2d');
  var answerCtx = answerCanvas.getContext('2d');
  var userPixels = userCtx.getImageData(0, 0, 300, 300).data;
  var answerPixels = answerCtx.getImageData(0, 0, 300, 300).data;
  var overlap = 0, total = 0;
  for (var i = 3; i < userPixels.length; i += 4) {
    if (answerPixels[i] > 128) { total++; if (userPixels[i] > 128) overlap++; }
  }
  return total > 0 ? (overlap / total) * 100 : 0;
}
```

**Step 1: HTML 템플릿 작성**

**Step 2: 테스트 — 히라가나 "あ" DATA로 필기 동작 확인**

**Step 3: 커밋**

```bash
git add jp-tutor/skills/jp/references/templates/game-stroke.html
git commit -m "feat: add stroke practice minigame template with canvas"
```

---

### Task 13: game-listen.html — 듣고 그리기 게임

**Files:**
- Create: `jp-tutor/skills/jp/references/templates/game-listen.html`

**Context:** TTS로 발음을 듣고 Canvas에 글자를 쓰는 게임. 이중부호화(dual coding). 가중치 0.9.

**DATA 구조:**
```json
{
  "gameKey": "listen_draw",
  "tier": 1,
  "tierName": "입문",
  "characters": [
    {
      "char": "あ",
      "reading": "a",
      "strokes": [[...]],
      "row_hint": "あ행",
      "concept": "hiragana_a"
    }
  ],
  "maxPlays": 3,
  "showRowHint": true,
  "similarityThreshold": 60,
  "preferredVoice": "Kyoko"
}
```

**기능 요구사항:**
1. game-stroke.html의 Canvas 코드를 재사용
2. TTS 재생 버튼 (tts-player.html의 speakGen + sortVoices 로직 재사용)
3. 재생 횟수 제한 (maxPlays, 남은 횟수 표시)
4. 행 힌트 (티어 1): "あ행이에요" 표시
5. 정답 공개 시 글자 표시 + 획순 애니메이션
6. preferredVoice 자동 선택 (tts-player.html 패턴)

**핵심:** game-stroke.html과 TTS 로직의 결합. Canvas 부분은 game-stroke.html에서 복사.

**Step 1: HTML 템플릿 작성**

**Step 2: 테스트 — TTS 재생 + 필기 동작 확인**

**Step 3: 커밋**

```bash
git add jp-tutor/skills/jp/references/templates/game-listen.html
git commit -m "feat: add listen-and-draw minigame template with TTS + canvas"
```

---

### Task 14: Phase 2 버전 범프

**Files:**
- Modify: `jp-tutor/.claude-plugin/plugin.json` — 0.8.0 → 0.9.0
- Modify: `.claude-plugin/marketplace.json` — 0.8.0 → 0.9.0

```bash
git commit -m "feat: v0.9.0 — Phase 2 canvas games (stroke practice, listen & draw)"
```

---

## Phase 3: 그리드 + AI 게임 (스네이크, 한자조립, 끝말잇기)

### Task 15: game-snake.html — 스네이크 매치 게임

**Files:**
- Create: `jp-tutor/skills/jp/references/templates/game-snake.html`

**Context:** 그리드에서 관련 쌍을 선으로 연결하는 게임. 인코딩(재인+시각탐색). 가중치 0.4.

**DATA 구조:**
```json
{
  "gameKey": "snake_match",
  "tier": 1,
  "tierName": "입문",
  "gridSize": 4,
  "pairs": [
    { "a": "食べる", "b": "먹다", "concept": "食べる" },
    { "a": "飲む", "b": "마시다", "concept": "飲む" },
    { "a": "見る", "b": "보다", "concept": "見る" },
    { "a": "聞く", "b": "듣다", "concept": "聞く" }
  ],
  "timeLimit": 60,
  "highlightFirst": 2,
  "pathConstraint": false,
  "obstacles": 0
}
```

**기능 요구사항:**
1. NxN 그리드: 셀에 텍스트 배치 (짝이 되는 아이템 랜덤 배치)
2. 선 그리기: 셀 클릭 → 드래그 → 짝 셀 도달 시 매칭 (SVG 또는 Canvas 오버레이)
3. pathConstraint (티어 3-4): 선이 다른 선과 교차하면 안 됨
4. obstacles (티어 4): 지나갈 수 없는 셀 3개 (어두운 색)
5. highlightFirst (티어 1): 처음 2개 매칭 가능 쌍을 살짝 하이라이트
6. 매칭 성공: 선 고정 + 셀 비활성화 (초록색 변환)
7. 매칭 실패: 선 사라짐 + 흔들림 애니메이션

**Step 1: HTML 템플릿 작성**

**Step 2: 테스트**

**Step 3: 커밋**

```bash
git add jp-tutor/skills/jp/references/templates/game-snake.html
git commit -m "feat: add snake match minigame template"
```

---

### Task 16: game-radical.html — 한자 조립 게임

**Files:**
- Create: `jp-tutor/skills/jp/references/templates/game-radical.html`

**Context:** 부수 조각을 조합하여 한자를 완성하는 게임. 인코딩(구조분석). 가중치 0.7.

**DATA 구조:**
```json
{
  "gameKey": "radical_assembly",
  "tier": 1,
  "tierName": "입문",
  "problems": [
    {
      "answer": "休",
      "meaning": "쉬다",
      "reading": "きゅう/やすむ",
      "parts": ["亻", "木"],
      "dummies": [],
      "layout": "left-right",
      "concept": "kanji_休"
    }
  ],
  "showPreview": true,
  "previewTime": 3,
  "showMeaning": true,
  "showReading": false,
  "timeLimit": 30,
  "chainMode": false
}
```

**티어별 파라미터:**
| 티어 | parts 수 | dummies | showPreview | showMeaning | showReading | timeLimit | chainMode |
|------|----------|---------|-------------|-------------|-------------|-----------|-----------|
| 1 | 2 | 0 | true (3초) | true | true | 30 | false |
| 2 | 2-3 | 1 | false | true | false | 25 | false |
| 3 | 3-4 | 2 | false | true | true (만) | 20 | true (3연속) |
| 4 | 4-5 | 3 | false | false | true (만) | 15 | true (5연속) |

**기능 요구사항:**
1. 부수 조각 영역: 드래그 가능한 부수 타일들
2. 조합 영역: 빈 한자 틀 (layout에 따라 좌우/상하/포함 구조)
3. 드래그 앤 드롭: 부수를 틀의 올바른 위치에 배치
4. 미리보기: 완성된 한자를 previewTime초 보여준 후 분해
5. 더미 부수: 정답에 포함되지 않는 부수 섞기
6. 체인 모드 (티어 3-4): 연속 문제, 체인 보너스 표시
7. 정답 판정: 모든 부수가 올바른 위치에 배치되었는지 확인
8. 정답 시: 완성된 한자 표시 + 뜻/읽기 + 축하 애니메이션
9. 오답 시: 올바른 조합 애니메이션 표시

**Step 1: HTML 템플릿 작성**

**Step 2: 테스트**

**Step 3: 커밋**

```bash
git add jp-tutor/skills/jp/references/templates/game-radical.html
git commit -m "feat: add radical assembly minigame template"
```

---

### Task 17: 끝말잇기 (shiritori) — 대화형 게임

**Context:** 끝말잇기는 HTML 템플릿 없이, Claude가 대화형으로 진행합니다. jp-game.md 커맨드에서 직접 처리.

**이 Task는 별도 파일이 필요 없습니다.** jp-game.md의 "텍스트 기반 게임" 섹션에서 이미 커버됨.
Claude가 minigame-rules.md의 끝말잇기 규칙을 읽고 직접 진행합니다:
- 어휘 범위 제한 (레벨별)
- 응답 시간 체크 (Claude가 안내)
- ん으로 끝나는 단어 경고 (티어 1)
- AI 난이도 조절 (쉬운/어려운 글자로 연결)

**Step 1: minigame-rules.md에 끝말잇기 상세 규칙이 포함되었는지 확인**

**Step 2: 커밋 (필요 없음 — Task 2에서 이미 포함)**

---

### Task 18: Phase 3 버전 범프

```bash
git commit -m "feat: v0.10.0 — Phase 3 games (snake match, radical assembly, shiritori)"
```

---

## Phase 4: LLM 채점 게임 (경어 변환, 문맥 추리)

### Task 19: 경어 변환 (keigo_transformer) — 대화형 게임

**Context:** 경어 변환은 자유 입력 + LLM 채점이 필요하므로 HTML 템플릿 없이 Claude가 대화형으로 진행합니다.

**별도 파일 불필요.** jp-game.md + minigame-rules.md로 커버.

Claude가 진행하는 방식:
1. 보통체 문장 제시 → 학습자가 경어로 변환
2. Claude가 정답 판정 (부분 점수 70% 이상이면 정답)
3. 오답 시 올바른 변환 + 해설 제공
4. 티어 3-4: 자유 입력, 복합 채점

---

### Task 20: 문맥 추리 (context_detective) — 대화형 게임

**Context:** 문맥에서 빈칸 단어를 추측. LLM 채점이 필요하므로 대화형.

**별도 파일 불필요.** jp-game.md + minigame-rules.md로 커버.

---

### Task 21: Phase 4 버전 범프 + 최종 정리

**Files:**
- Modify: `jp-tutor/.claude-plugin/plugin.json` — → 1.0.0
- Modify: `.claude-plugin/marketplace.json` — → 1.0.0
- Modify: `README.md` — 미니게임 전체 목록 업데이트

```bash
git commit -m "feat: v1.0.0 — Complete minigame system (10 games, smart recommendation, SRS integration)"
```

---

## 요약

| Phase | Tasks | 파일 | 핵심 내용 |
|-------|-------|------|-----------|
| 0 | 1-6 | 6개 (2 new, 4 modify) | 인프라: 추천 알고리즘, 규칙, 커맨드, SKILL.md, daily, CLAUDE.md |
| 1 | 7-11 | 5개 (3 new, 2 modify) | 코어 게임 3개 + 대시보드 + 버전 범프 |
| 2 | 12-14 | 3개 (2 new, 1 modify) | Canvas 게임 2개 + 버전 범프 |
| 3 | 15-18 | 3개 (2 new, 1 modify) | 그리드 게임 2개 + 대화형 1개 + 버전 범프 |
| 4 | 19-21 | 1개 (1 modify) | 대화형 2개 + 최종 버전 1.0.0 |

**총: 21 Tasks, 9 new files + 8 modify = 17 file operations**

**HTML 게임 템플릿 7개:**
1. game-match.html (매칭 카드)
2. game-particle.html (파티클 배틀)
3. game-shuffle.html (셔플 퍼즐)
4. game-stroke.html (따라 쓰기)
5. game-listen.html (듣고 그리기)
6. game-snake.html (스네이크 매치)
7. game-radical.html (한자 조립)

**대화형 게임 3개** (HTML 없음):
8. shiritori (끝말잇기)
9. keigo_transformer (경어 변환)
10. context_detective (문맥 추리)
