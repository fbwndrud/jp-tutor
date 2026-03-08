---
description: "일일 학습 세션. 복습 항목 + 새 개념을 혼합하여 오늘의 학습 세트를 제공합니다."
allowed-tools: ["Read", "Write", "Edit", "Glob", "Grep", "Bash"]
model: sonnet
argument-hint: "[카테고리: vocab/grammar/kanji 또는 빈칸으로 혼합]"
---

# 일일 학습 세션

## 사전 준비

1. `skills/jp/SKILL.md` 스킬 파일을 읽어 튜터 역할을 숙지하세요. (특히 TTS 섹션 확인)
2. TTS 환경을 감지합니다 (SKILL.md TTS 섹션 참조). 원격 환경이면 세션 종료 시 HTML TTS 플레이어를 생성합니다.
3. `jp-data/progress.json`을 읽으세요.
   - 파일이 없으면: "아직 초기 설정이 되어 있지 않습니다. `/jp:start`로 시작해주세요!" 라고 안내하고 종료합니다.
3. `skills/jp/references/quiz-rules.md`와 `skills/jp/references/exercise-types.md`를 읽으세요.
4. 프로필에 맞는 커리큘럼 파일을 읽으세요:
   - jlpt: `skills/jp/references/curricula/japanese-jlpt.md`
   - business: `skills/jp/references/curricula/business-japanese.md`

## Pre-N5 레벨인 경우

progress.json의 level이 "Pre-N5"인 경우, 일일 학습은 가나 중심으로 진행합니다:

1. `skills/jp/references/curricula/kana.md`와 `skills/jp/references/exercise-types-kana.md`를 읽습니다.
2. 가나 복습 대상 (kana.concepts에서 `nextReview <= 오늘`)을 우선 복습합니다.
3. 현재 스테이지의 새 문자를 소개하고 연습합니다.
4. 진행 방식은 `/jp:kana` 명령어와 동일하게 진행합니다.

## 오늘의 학습 세트 구성 (N5 이상)

오늘 날짜를 기준으로 학습 세트를 구성합니다. 목표: **복습 ~5개 + 신규 ~5개 = 약 10개 개념**

### Priority 1: 지연된 복습 항목

progress.json의 모든 카테고리(kana, vocab, grammar, kanji)에서 `nextReview <= 오늘`인 개념을 수집합니다.
배지 우선순위로 정렬합니다: 빨강 > 노랑 > 초록 > 파랑
상위 5개를 복습 세트로 선택합니다.

### Priority 2: 약점 개념

복습 대상이 5개 미만이면, 배지가 빨강/노랑인 개념 중 nextReview가 아직 안 된 것도 추가합니다.

### Priority 3: 새 개념

현재 레벨 커리큘럼에서 아직 학습하지 않은(progress.json에 없는) 개념을 순서대로 5개 선택합니다.
- jlpt: 현재 레벨(N5~N1)의 단어/문법/한자 커리큘럼에서 다음 항목
- business: 현재 모듈의 다음 학습 항목
- textbook: 현재 챕터의 다음 학습 항목

## 학습 세션 실행

### 세션 시작 메시지

```
오늘의 학습을 시작합니다!
복습: {reviewCount}개 | 신규: {newCount}개

연속 학습: {streak}일째
```

### 복습 파트

복습 항목이 5개 이상이면, **반드시** 퀴즈 전에 워밍업 미니게임 HTML을 생성합니다:
1. `skills/jp/references/templates/game-match.html` 등 적절한 템플릿을 읽고 `const DATA = null;`을 복습 항목 데이터로 치환
2. `jp-data/game.html`로 저장하고 "게임으로 먼저 워밍업 해볼까요!" 안내
3. **유저가 게임을 마친 후에만 퀴즈로 넘어갑니다**

퀴즈 형식:
- vocab: 일본어→한국어 또는 한국어→일본어 4지선다 / 직접 입력
- grammar: 빈칸 채우기, 문장 완성, 올바른 형태 선택
- kanji: 읽기(음독/훈독) 또는 의미 맞추기

각 문제 후:
- **TTS로 정답 단어/문장 발음 재생** (레벨별 속도 자동 조절)
- 정답 시: 격려 + 간단한 추가 예문
- 오답 시: 정답 설명 + 기억법 팁

### 신규 학습 파트

새 개념을 **듣기 → 게임 → 퀴즈** 순으로 단계적으로 학습합니다:

1. **소개** — 개념 소개 (의미, 읽기, 용례) + 예문 2~3개
2. **TTS 발음 학습** — Cowork이면 해당 개념들의 TTS 플레이어 HTML을 생성. 로컬이면 say로 재생. 유저가 발음을 충분히 듣도록 안내.
3. **⚠️ 미니게임으로 연습 (건너뛰기 금지)** — 신규 개념 3개 이상이면 **반드시** 게임 HTML을 생성:
   - 새 어휘 → `game-match.html` 또는 `game-shuffle.html`
   - 새 문법 → `game-particle.html` 또는 `game-shuffle.html`
   - 새 한자 → `game-radical.html` 또는 `game-match.html`
   - 템플릿의 `const DATA = null;`을 실제 데이터로 치환 → `jp-data/game.html`로 저장
   - 게임 티어는 1(입문)로 고정
   - **유저가 게임을 마친 후에만 퀴즈로 넘어갑니다**
4. **확인 퀴즈** — 게임으로 연습한 뒤 퀴즈로 정착 확인 (1문제씩)

N4 이하 학습자에게는 반드시 후리가나를 병기합니다.

## progress.json 업데이트

세션 완료 후 다음을 업데이트합니다:

1. **각 개념별**: `attempts` +1, 정답 시 `correct` +1, `lastReview`를 오늘로, `nextReview` 갱신
   - 정답: 다음 간격으로 이동 (1d→3d→7d→14d→30d→60d)
   - 오답: 간격을 1d로 리셋
   - 신규 개념: `attempts: 1`, `correct: 0 또는 1`, `lastReview: 오늘`, `nextReview: 내일`
2. **XP**: 퀴즈 정답당 +10 XP
3. **Daily 완료 보너스**: +50 XP
4. **스트릭 보너스**: 연속일수 x 5 XP
5. **스트릭**: lastDate가 어제면 current +1, 오늘이면 유지, 그 외면 1로 리셋. best 갱신.
6. **일일 목표 달성 확인**: daily_goal.completed >= daily_goal.target이면 +30 XP 보너스
7. **배지 체크**: 새로운 배지 달성 조건을 확인하고 부여
8. **playerLevel**: floor(XP / 100) + 1 계산, 레벨업 시 축하 메시지
9. **lastSession**: 오늘 날짜로 업데이트

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

## 세션 종료 메시지

```
오늘의 학습 완료!
- 학습 개념: {total}개 (복습 {review}개 + 신규 {new}개)
- 정답률: {correctRate}%
- 획득 XP: +{earnedXP} (총 {totalXP})
- 일일 목표: {completed}/{target} 문제
- 새 배지: {새로 획득한 배지가 있으면 표시}
- 다음 복습: {nextReviewDate}에 {nextReviewCount}개
```
