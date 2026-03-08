---
description: "일일 학습 세션. 복습 항목 + 새 개념을 혼합하여 오늘의 학습 세트를 제공합니다."
allowed-tools: ["Read", "Write", "Edit", "Glob", "Grep", "Bash"]
model: sonnet
argument-hint: "[카테고리: vocab/grammar/kanji 또는 빈칸으로 혼합]"
---

# 일일 학습 세션

## 사전 준비

1. `skills/jp/SKILL.md` 스킬 파일을 읽어 튜터 역할을 숙지하세요.
2. `jp-data/progress.json`을 읽으세요.
   - 파일이 없으면: "아직 초기 설정이 되어 있지 않습니다. `/jp:start`로 시작해주세요!" 라고 안내하고 종료합니다.
3. `skills/jp/references/quiz-rules.md`와 `skills/jp/references/exercise-types.md`를 읽으세요.
4. 프로필에 맞는 커리큘럼 파일을 읽으세요:
   - jlpt: `skills/jp/references/curricula/japanese-jlpt.md`
   - business: `skills/jp/references/curricula/business-japanese.md`

## 오늘의 학습 세트 구성

오늘 날짜를 기준으로 학습 세트를 구성합니다. 목표: **복습 ~5개 + 신규 ~5개 = 약 10개 개념**

### Priority 1: 지연된 복습 항목

progress.json의 모든 카테고리(vocab, grammar, kanji)에서 `nextReview <= 오늘`인 개념을 수집합니다.
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

복습 항목을 하나씩 퀴즈 형식으로 출제합니다:
- vocab: 일본어→한국어 또는 한국어→일본어 4지선다 / 직접 입력
- grammar: 빈칸 채우기, 문장 완성, 올바른 형태 선택
- kanji: 읽기(음독/훈독) 또는 의미 맞추기

각 문제 후:
- 정답 시: 격려 + 간단한 추가 예문
- 오답 시: 정답 설명 + 기억법 팁

### 신규 학습 파트

새 개념을 소개하고 바로 연습합니다:
1. 개념 소개 (의미, 읽기, 용례)
2. 예문 2~3개 제시
3. 확인 퀴즈 1문제

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
