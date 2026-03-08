---
description: "진도 동기화. 오래된 데이터 정리, 배지 재계산, STUDY-LOG.md 갱신, CLAUDE.md 핫캐시 갱신을 수행합니다."
allowed-tools: ["Read", "Write", "Edit", "Glob", "Grep", "Bash"]
model: sonnet
argument-hint: ""
---

# 진도 동기화

## 사전 준비

1. `skills/jp/SKILL.md` 스킬 파일을 읽어 규칙을 숙지하세요.
2. `jp-data/progress.json`을 읽으세요.
   - 파일이 없으면: "아직 초기 설정이 되어 있지 않습니다. `/jp:start`로 시작해주세요!" 라고 안내하고 종료합니다.

## Step 1: 오래된 데이터 정리

모든 카테고리(vocab, grammar, kanji)의 concepts를 순회합니다:

- `lastReview`가 90일 이상 전인 개념 → 재복습 대상으로 마킹:
  - `nextReview`를 오늘 날짜로 설정 (복습 큐에 다시 들어감)
  - 기존 correct/attempts는 유지 (학습 이력 보존)
- 해당 개념 수를 카운트합니다.

## Step 2: 배지 재계산

모든 개념의 배지를 다시 계산합니다:

| 비율 (correct/attempts) | 배지 |
|--------------------------|------|
| >= 80% (최소 3회 시도) | 파랑 (마스터) |
| >= 60% (최소 3회 시도) | 초록 (숙련) |
| >= 40% | 노랑 (학습중) |
| < 40% | 빨강 (보강 필요) |
| 0 attempts | 미학습 |
| < 3 attempts | 임시 배지 |

배지 분포 변화를 기록합니다 (전/후 비교용).

## Step 3: STUDY-LOG.md 갱신

`jp-data/STUDY-LOG.md`를 progress.json 기반으로 업데이트합니다:

### Active Learning
현재 진행 중인 학습 항목들:
- progress.json에서 attempts > 0이고 배지가 파랑이 아닌 개념들
- 형식: `- [ ] {카테고리} {레벨}: {개념명} ({attempts}회 시도, 배지: {badge})`

### Review Queue
향후 7일간 복습 예정:
- nextReview 날짜별로 정리
- 형식: `- [ ] {날짜}: {개념명} (간격: {interval}, 배지: {badge})`

### Completed Sessions
최근 10개 세션 기록:
- session_log에서 최근 10개
- 형식: `- [x] {날짜} {mode}: {topics} ({questions}Q, {correct}/{questions})`

### Milestones
주요 달성 기록:
- 레벨업, 새 배지 획득, 카테고리 완료 등
- 형식: `- {날짜}: {마일스톤 설명}`

## Step 4: CLAUDE.md 핫캐시 갱신

프로젝트 루트의 `CLAUDE.md`에서 `# Language Learning` 섹션을 찾아 업데이트합니다.

### Profile 업데이트
- Level, Profiles, Streak, XP, 레벨/칭호 최신화
- Textbook 정보 (있으면)

### Hot Weaknesses (max 15)
- progress.json에서 배지가 빨강/노랑인 개념 최대 15개
- 빨강 우선, 노랑 그 다음
- 형식: `- [RED] {개념명}: {error_notes의 첫 번째}`
- 형식: `- [YLW] {개념명}: {error_notes의 첫 번째}`

### Review Queue (next 7 days)
- 향후 7일간 날짜별 복습 예정 개수
- 형식: `- TODAY: {개념1}, {개념2}, ...`
- 형식: `- {날짜}: {개념1}, {개념2}, ...`

### Current Focus
- 프로필별 현재 학습 위치
- jlpt: `- JLPT: {level} Unit {unit} ({topic})`
- business: `- Business: Module {n} ({module_name}) {status}`
- textbook: `- Textbook: {name} {chapter}課 — {status}`

## Step 5: progress.json 저장

정리된 데이터를 `jp-data/progress.json`에 저장합니다.

## 동기화 요약 출력

모든 안내는 **한국어**로 출력합니다:

```
진도 동기화 완료!

정리된 항목:
- 90일 이상 미복습 → 재복습 대상: {staleCount}개

배지 현황:
- 어휘: 파랑 {n} / 초록 {n} / 노랑 {n} / 빨강 {n} / 미학습 {n}
- 문법: 파랑 {n} / 초록 {n} / 노랑 {n} / 빨강 {n} / 미학습 {n}
- 한자: 파랑 {n} / 초록 {n} / 노랑 {n} / 빨강 {n} / 미학습 {n}

STUDY-LOG.md 갱신 완료.
CLAUDE.md 핫캐시 갱신 완료.

향후 7일 복습 예정:
- 오늘: {n}개
- 내일: {n}개
- ...
```
