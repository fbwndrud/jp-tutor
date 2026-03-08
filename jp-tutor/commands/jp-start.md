---
description: "일본어 학습 온보딩. 학습 프로필 설정, 레벨 진단, 초기 progress.json 생성을 수행합니다."
allowed-tools: ["Read", "Write", "Edit", "Glob", "Bash"]
model: sonnet
argument-hint: ""
---

# 일본어 학습 온보딩

## 사전 준비

1. `skills/jp/SKILL.md` 스킬 파일을 읽어 튜터 역할을 숙지하세요.
2. `jp-data/progress.json` 파일 존재 여부를 확인하세요.
   - 이미 존재하면: "이미 학습 프로필이 설정되어 있습니다. 초기화하시겠습니까?" 라고 확인합니다.
   - 존재하지 않으면: 온보딩을 시작합니다.

## 5단계 온보딩 프로세스

모든 안내는 **한국어**로 진행합니다.

### Step 1: 환영 메시지

```
일본어 학습 튜터에 오신 것을 환영합니다!
맞춤형 학습 환경을 설정해 드리겠습니다.
```

### Step 2: 프로필 선택

사용자에게 학습 목적을 물어봅니다. **복수 선택 가능**합니다.

- **jlpt**: JLPT 시험 준비 (N5~N1)
- **business**: 비즈니스 일본어 (업무/경어/이메일)
- **textbook**: 교재 기반 학습 (특정 교과서 따라가기)

복수 선택 시 메인 프로필은 첫 번째 선택으로 설정합니다.

### Step 3: 레벨 평가

프로필에 따라 다르게 진행합니다:

**jlpt 선택 시:**
- "현재 일본어 실력은 어느 정도인가요?" 라고 물어봅니다.
- 선택지: 히라가나부터(완전 초보) / N5(입문) / N4(초급) / N3(중급) / N2(중상급) / N1(상급) / 모르겠어요
- **"히라가나부터" 선택 시**: 레벨을 `Pre-N5`로 설정합니다. 히라가나/카타카나부터 학습을 시작하며, 가나를 마스터하면 자동으로 N5로 승급합니다.
- "모르겠어요" 선택 시: 간단한 자가진단 퀴즈 5문제를 출제하여 레벨을 추정합니다. 히라가나를 읽지 못하면 Pre-N5로 설정합니다.

**business 선택 시:**
- 현재 일본어 기초 레벨을 확인합니다 (JLPT 기준 대략적 수준).

**textbook 선택 시:**
- 사용 중인 교재 이름과 현재 진도를 물어봅니다.

### Step 4: 비즈니스 세부 설정 (business 선택 시만)

- 업종/분야: IT, 제조, 금융, 무역, 서비스, 기타
- 주요 학습 포커스: 이메일, 전화, 회의, 프레젠테이션, 접객, 일상 업무

### Step 5: 교재 등록 (textbook 선택 시만)

- 교재명, 출판사, 현재 챕터 등을 기록합니다.

## progress.json 생성

온보딩 완료 후 `jp-data/` 디렉토리가 없으면 생성하고, 다음 구조의 `jp-data/progress.json`을 작성합니다:

```json
{
  "profile": "<메인 프로필>",
  "profiles": ["<선택한 프로필들>"],
  "level": "<설정된 레벨>",
  "created": "<오늘 날짜 YYYY-MM-DD>",
  "lastSession": "<오늘 날짜 YYYY-MM-DD>",
  "xp": 0,
  "playerLevel": 1,
  "streak": { "current": 0, "best": 0, "lastDate": null },
  "kana": { "hiraStage": 1, "kataStage": 0, "concepts": {} },
  "vocab": { "concepts": {} },
  "grammar": { "concepts": {} },
  "kanji": { "concepts": {} },
  "reading": { "sessions": [] },
  "writing": { "sessions": [] },
  "conversation": { "sessions": [] },
  "translation": { "sessions": [] },
  "business": { "modules": {}, "industry": "<업종>", "focus": ["<포커스들>"] },
  "textbook": { "chapters": {}, "name": "<교재명>", "publisher": "<출판사>" }
}
```

- business, textbook 관련 필드는 해당 프로필을 선택한 경우에만 값을 채웁니다.
- 선택하지 않은 프로필의 필드는 빈 기본값으로 둡니다.

## STUDY-LOG.md 초기화

`jp-data/STUDY-LOG.md`를 생성합니다:

```markdown
# Study Log

## Active Learning
- (아직 학습 항목 없음)

## Review Queue
- (복습 대상 없음)

## Completed Sessions
- (완료된 세션 없음)

## Milestones
- {오늘 날짜}: 학습 시작!
```

## CLAUDE.md 핫캐시 설정

프로젝트 루트의 `CLAUDE.md` 파일에 Language Learning 섹션을 추가합니다.
파일이 있으면 기존 내용 뒤에 추가하고, 없으면 새로 생성합니다.

추가할 내용:

```markdown
# Language Learning

## Profile
- Level: {선택한 레벨} | Profiles: {선택한 프로필들}
- Streak: 0 days | XP: 0 (Lv 1 초심자)

## Hot Weaknesses (max 15)
- (아직 약점 데이터 없음)

## Review Queue (next 7 days)
- (복습 대상 없음)

## Current Focus
- {프로필에 따른 첫 학습 목표}
```

## 온보딩 완료 메시지

```
설정이 완료되었습니다!
- 프로필: <프로필>
- 레벨: <레벨>

시작하려면:
- `/jp:kana` — 히라가나/카타카나 학습 (Pre-N5)
- `/jp:daily` — 오늘의 학습
- `/jp` — 자유 학습
- `/jp:dashboard` — 학습 현황 확인
```
