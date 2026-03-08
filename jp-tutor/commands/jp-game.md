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
