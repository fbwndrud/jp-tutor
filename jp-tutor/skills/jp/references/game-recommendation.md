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
