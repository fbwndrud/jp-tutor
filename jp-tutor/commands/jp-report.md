---
description: "학습 리포트. 주간/월간 학습 통계, 성장 추이, 지난 기간 대비 비교를 보여줍니다."
allowed-tools: ["Read", "Write", "Edit"]
model: sonnet
argument-hint: "[기간: 주간|월간|전체]"
---

# 학습 리포트

## 사전 준비

1. `jp-data/progress.json`을 읽으세요.
   - 파일이 없으면: "아직 학습 데이터가 없습니다. `/jp:start`로 시작해주세요!" 안내 후 종료.

## 기간 결정

- "주간", "이번 주": 최근 7일
- "월간", "이번 달": 최근 30일
- "전체", 미지정: 전체 기간

## 리포트 생성

### 학습 활동

progress.json의 session_log에서 해당 기간의 세션을 필터링합니다:

```
학습 리포트 ({기간})
━━━━━━━━━━━━━━━━━━━━━━━━━━━━

학습 활동
- 총 학습일: {days}일 / {period}일
- 총 세션 수: {sessions}회
- 총 문제 수: {questions}개 (정답: {correct}개)
- 전체 정답률: {rate}%

카테고리별 정답률
- 가나: {kana_rate}% ({kana_questions}문제) (Pre-N5인 경우)
- 어휘: {vocab_rate}% ({vocab_questions}문제)
- 문법: {grammar_rate}% ({grammar_questions}문제)
- 한자: {kanji_rate}% ({kanji_questions}문제)
```

### 성장 추이

```
성장 추이
- XP 획득: +{xp_earned} (Lv.{prev} → Lv.{current})
- 새 개념 학습: {new_concepts}개
- 배지 변동:
  - 빨강→초록/파랑: {improved}개
  - 새 파랑 배지: {new_blue}개
- 획득 배지: {new_badges}
```

### 스트릭

```
스트릭
- 현재: {current}일 연속
- 이 기간 최장: {period_best}일
- 역대 최장: {all_time_best}일
```

### 지난 기간 대비 (가능한 경우)

이전 동일 기간과 비교:
```
지난주 대비
- 학습일: {prev_days}일 → {curr_days}일 ({diff})
- 정답률: {prev_rate}% → {curr_rate}% ({diff})
- XP: {prev_xp} → {curr_xp} ({diff})
```

### 다음 학습 추천

리포트 데이터를 기반으로 다음 기간의 학습 방향을 추천합니다:
- 약점 카테고리 집중 제안
- 복습 밀린 개념 안내
- 새 모드 시도 제안 (사용하지 않은 학습 모드가 있으면)
