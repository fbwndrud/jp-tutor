---
model: sonnet
description: "교재 기반 학습 -- 교재 등록, 단원별 예습/복습, 진도 추적"
argument-hint: "[교재명] [단원번호] [모드: preview|review|quiz|add]"
---

# 교재 기반 학습 (/jp:textbook)

## 데이터 로드

1. `jp-data/progress.json`을 읽습니다.
2. 파일이 없으면 → "/jp:start 명령어로 먼저 학습 프로필을 설정해주세요." 안내 후 종료.
3. `textbook` 섹션을 확인합니다.

## 교재 선택

progress.json의 `textbook.books` 배열을 확인:
- 교재 0권 → "등록된 교재가 없습니다. 교재를 추가하시겠습니까?" → add 모드 안내
- 교재 1권 → 자동 선택
- 교재 2권 이상 → 목록 표시, 사용자 선택 또는 인자에서 교재명 매칭

## 단원/모드 결정

인자에서 단원번호와 모드를 파싱합니다. 미지정 시 자동 결정:

### 자동 모드 결정
단원의 `status` 필드 기준:
- `not_started` → preview (예습)
- `in_progress` → review (복습) 또는 quiz
- `completed` → quiz (정기 복습)

## 모드별 진행

### add — 교재 등록
- 사용자에게 교재 정보 수집:
  - 교재명 (필수)
  - 총 단원 수 (필수)
  - 현재 진도 (선택, 기본값: 1)
  - 교재 유형: 종합/문법/회화/한자/독해 (선택)
- PDF나 이미지 드롭 시: 표지/목차에서 정보 추출 시도
- progress.json의 textbook.books에 새 교재 추가:
  ```json
  {
    "name": "교재명",
    "type": "종합",
    "totalUnits": 20,
    "currentUnit": 1,
    "units": {}
  }
  ```

### preview — 예습
1. 해당 단원의 핵심 내용 소개:
   - 새 어휘 목록 (읽기 + 의미 + 예문)
   - 새 문법 포인트 (설명 + 접속 + 예문 3개)
   - 새 한자 (해당 시)
2. 각 항목을 간결하게 설명
3. 플래시카드 형식으로 정리 (선택 시)
4. 단원 status를 `in_progress`로 변경

### review — 복습
1. 해당 단원 핵심 포인트 요약 (카드 형식)
2. 빈칸 채우기 연습 4문제:
   - 해당 단원의 어휘/문법에서 출제
   - 이전 단원에서 틀렸던 항목도 혼합
3. 틀린 항목 집중 해설
4. progress.json 업데이트

### quiz — 퀴즈
1. `skills/jp/references/quiz-rules.md` 규칙 적용
2. 해당 단원에서 4문제 1라운드 출제
3. 누적 테스트 옵션: "이전 단원도 포함할까요?" → 이전 3단원까지 범위 확장
4. 라운드 완료 후 결과 요약 + progress.json 업데이트
5. 80% 이상 정답 시 단원 status를 `completed`로 변경

## 수업 진도 동기화

사용자가 "이번 주 수업이 N과까지야" 형태로 입력하면:
- 해당 교재의 `currentUnit`을 N으로 업데이트
- currentUnit 이전의 미완료 단원 목록 표시
- "수업 진도에 맞춰 [미완료 단원] 복습을 추천합니다" 안내

## 교재별 독립 추적

- 각 교재는 독립적인 units 진행 상태 보유
- 교재 간 진행이 서로 영향을 주지 않음
- 대시보드에서 교재별 진행률 표시 가능

## 배지 및 진행 추적

- 단원별 개념은 textbook.books[i].units[n].concepts에 기록
- 퀴즈 결과: correct/attempts, nextReview 갱신
- XP: 퀴즈 정답 +10, preview 완료 +15, review 완료 +20
- 배지 계산은 `skills/jp/references/grading-rubrics.md` 기준 적용

## 세션 종료

1. progress.json 업데이트
2. 학습 요약: 교재명, 단원, 모드, 학습 항목 수, 정답률, 획득 XP
3. 다음 추천: 다음 단원 예습 또는 현재 단원 복습/퀴즈 안내
