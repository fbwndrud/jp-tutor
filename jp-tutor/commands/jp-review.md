---
description: "에빙하우스 스마트 복습. 복습 예정인 개념을 우선순위별로 퀴즈하고 간격을 갱신합니다."
allowed-tools: ["Read", "Write", "Edit", "Glob", "Grep", "Bash"]
model: sonnet
argument-hint: "[카테고리: vocab/grammar/kanji 또는 빈칸으로 전체]"
---

# 에빙하우스 스마트 복습

## 사전 준비

1. `skills/jp/SKILL.md` 스킬 파일을 읽어 튜터 역할을 숙지하세요.
2. `jp-data/progress.json`을 읽으세요.
   - 파일이 없으면: "아직 초기 설정이 되어 있지 않습니다. `/jp:start`로 시작해주세요!" 라고 안내하고 종료합니다.
3. `skills/jp/references/quiz-rules.md`를 읽어 퀴즈 규칙을 숙지하세요.

## 복습 대상 수집

오늘 날짜를 기준으로, progress.json의 모든 카테고리(vocab, grammar, kanji)에서 복습 대상을 수집합니다:

- 조건: `nextReview <= 오늘 날짜`
- 인자로 특정 카테고리가 지정되면 해당 카테고리만 필터링

## 우선순위 정렬

배지 기준으로 정렬합니다 (correct/attempts 비율 기반):

1. 빨강 (< 40%) — 가장 먼저 복습
2. 노랑 (40%~59%)
3. 초록 (60%~79%)
4. 파랑 (>= 80%) — 마지막

같은 배지 내에서는 nextReview가 더 오래된 것(더 밀린 것)을 우선합니다.
한 세션에 최대 20개 개념을 복습합니다.

## 복습 대상 없음

복습 대상이 없으면:
```
오늘은 복습할 항목이 없습니다!
다음 복습 예정: {가장 가까운 nextReview 날짜}에 {개수}개

`/jp:daily`로 새로운 학습을 시작해보세요.
```

## 복습 세션 실행

### 세션 시작

```
스마트 복습을 시작합니다!
오늘 복습 대상: {totalCount}개
- 보강 필요(빨강): {redCount}개
- 학습중(노랑): {yellowCount}개
- 숙련(초록): {greenCount}개
- 마스터(파랑): {blueCount}개
```

### 라운드 진행

4문제씩 한 라운드로 진행합니다:

각 문제:
1. 카테고리에 맞는 퀴즈 형식으로 출제
   - vocab: 의미, 읽기, 용례 중 랜덤
   - grammar: 빈칸 채우기, 문장 변환, 올바른 형태 선택
   - kanji: 읽기(음독/훈독), 의미, 쓰기
2. 사용자 답변 받기
3. 채점 및 피드백:
   - 정답: 간단한 격려 + 현재 배지 → 다음 배지 진행 상황
   - 오답: 정답 제시 + 핵심 설명 + 기억법 팁

N4 이하 학습자에게는 반드시 후리가나를 병기합니다.

### 라운드 사이

라운드 완료 후:
```
라운드 {n} 완료! ({correct}/{total} 정답)
남은 복습: {remaining}개

계속하시겠습니까? (Y/n)
```

## nextReview 갱신 로직

복습 간격: **1일 → 3일 → 7일 → 14일 → 30일 → 60일**

각 개념에 대해:

### 정답 시
1. 현재 간격 계산: `nextReview - lastReview` (일 수)
2. 다음 간격 결정:
   - 현재 간격 <= 1일 → 다음 간격 3일
   - 현재 간격 <= 3일 → 다음 간격 7일
   - 현재 간격 <= 7일 → 다음 간격 14일
   - 현재 간격 <= 14일 → 다음 간격 30일
   - 현재 간격 <= 30일 → 다음 간격 60일
   - 현재 간격 > 30일 → 다음 간격 60일 (최대)
3. `lastReview` = 오늘
4. `nextReview` = 오늘 + 다음 간격
5. `correct` += 1

### 오답 시
1. `lastReview` = 오늘
2. `nextReview` = 내일 (간격 1일로 리셋)

### 공통
- `attempts` += 1

## progress.json 업데이트

세션 완료 후:
1. 각 개념의 correct, attempts, lastReview, nextReview 업데이트
2. XP: 복습 세션 완료 +20 XP
3. 스트릭 업데이트: lastDate가 어제면 current +1, 오늘이면 유지, 그 외면 1로 리셋. best 갱신.
4. playerLevel 재계산: floor(XP / 100) + 1
5. lastSession: 오늘 날짜

## 복습 요약

```
복습 완료!
- 복습 개념: {total}개
- 정답률: {correctRate}%
- 간격 상승: {advancedCount}개
- 간격 리셋: {resetCount}개
- 획득 XP: +20 (총 {totalXP})

다음 복습 예정:
- 내일: {tomorrowCount}개
- 3일 후: {in3daysCount}개
- 7일 후: {in7daysCount}개
```
