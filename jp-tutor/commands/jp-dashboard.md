---
description: "학습 대시보드. 현재 학습 현황, 배지 분포, 복습 예정, 추천 등을 요약하여 보여줍니다."
allowed-tools: ["Read", "Write", "Edit", "Glob", "Bash"]
model: haiku
argument-hint: ""
---

# 학습 대시보드

## 사전 준비

1. `jp-data/progress.json`을 읽으세요.
   - 파일이 없으면: "아직 초기 설정이 되어 있지 않습니다. `/jp:start`로 시작해주세요!" 라고 안내하고 종료합니다.

## 데이터 집계

progress.json에서 다음을 계산합니다:

### 배지 분포

각 카테고리(vocab, grammar, kanji)의 concepts를 순회하여 배지별 개수를 집계합니다:

| 비율 (correct/attempts) | 배지 |
|--------------------------|------|
| >= 80% (최소 3회) | 파랑 (마스터) |
| >= 60% (최소 3회) | 초록 (숙련) |
| >= 40% | 노랑 (학습중) |
| < 40% | 빨강 (보강 필요) |
| 0 attempts | 미학습 |

### 오늘 복습 대상

`nextReview <= 오늘`인 개념의 총 수를 계산합니다.

### 최근 세션

`lastSession` 날짜와 reading/writing/conversation/translation의 최근 sessions를 확인합니다.

## HTML 대시보드 생성 (템플릿 있는 경우)

`skills/jp/references/templates/dashboard.html` 파일이 존재하는지 확인합니다.

### 템플릿이 있는 경우

1. 템플릿 파일을 읽습니다.
2. 다음 데이터 객체를 조립합니다:
   ```javascript
   {
     profile: "<프로필>",
     level: "<레벨>",
     xp: <XP>,
     playerLevel: <플레이어레벨>,
     streak: { current: <현재>, best: <최고>, lastDate: "<날짜>" },
     badges: {
       vocab: { blue: N, green: N, yellow: N, red: N, untested: N },
       grammar: { blue: N, green: N, yellow: N, red: N, untested: N },
       kanji: { blue: N, green: N, yellow: N, red: N, untested: N }
     },
     reviewCount: <오늘 복습 대상 수>,
     recentSessions: [...],
     reviewQueue: { today: N, tomorrow: N, ... }
   }
   ```
3. 템플릿의 `const DATA = null;`을 `const DATA = <조립한 JSON>;`으로 치환합니다.
4. `jp-data/dashboard.html`로 저장합니다.
5. "HTML 대시보드가 `jp-data/dashboard.html`에 저장되었습니다." 라고 안내합니다.

### 템플릿이 없는 경우

HTML 생성을 건너뜁니다.

## 텍스트 대시보드 출력 (항상)

항상 텍스트 형식의 대시보드를 출력합니다. 모든 텍스트는 **한국어**로 작성합니다:

```
일본어 학습 대시보드
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
프로필: {profile} {level} | Lv.{playerLevel} {level_title} | XP: {xp}
연속 학습: {streak.current}일 (최고: {streak.best}일)
다음 레벨까지: {xp_remaining} XP
획득 배지: {badges_earned_count}개
일일 목표: {completed}/{target} 문제
주간 목표: {completed_days}/{target_days}일

어휘: 파랑x{blue} 초록x{green} 노랑x{yellow} 빨강x{red} 미학습x{untested}
문법: 파랑x{blue} 초록x{green} 노랑x{yellow} 빨강x{red} 미학습x{untested}
한자: 파랑x{blue} 초록x{green} 노랑x{yellow} 빨강x{red} 미학습x{untested}

오늘 복습 대상: {reviewCount}개

최근 학습:
- {lastSession}: {마지막 세션 정보}

추천:
- {복습 대상이 있으면 복습 추천, 없으면 다음 학습 추천}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
