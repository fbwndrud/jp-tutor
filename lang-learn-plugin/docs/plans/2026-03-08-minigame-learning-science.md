# 미니게임 학습과학 통합 설계서

> 학습과학(Learning Science) 관점에서 10개 미니게임의 SRS 통합, 인지부하 관리,
> 최적 배치, 인코딩 깊이 분석, 안티패턴을 정의합니다.

---

## 1. 게임 결과 -> 간격반복(SRS) 통합

### 1.1 핵심 원칙: 인출 난이도에 따른 가중치(Retrieval Effort Weighting)

Bjork(1994)의 "바람직한 어려움(desirable difficulties)" 이론에 따르면, 더 힘들게 인출한 기억이 더 강화됩니다. 따라서 모든 게임의 정답을 동일하게 취급하면 안 됩니다. **인출 난이도(retrieval effort)**에 따라 가중치를 차등 적용합니다.

### 1.2 게임별 가중치 테이블

| 게임 | 인출 유형 | 가중치 | 근거 |
|------|-----------|--------|------|
| A. Matching Cards | 재인(recognition) + 연합(association) | **0.4** | 시각적 단서가 주어진 상태의 매칭. 재인은 회상보다 쉬움 |
| D. Shuffle Puzzle | 순서 재구성(sequencing) | **0.7** | 문장 구조의 절차적 지식 활성화. 부분적 생성 필요 |
| E. Snake Match | 재인(recognition) + 시각탐색 | **0.4** | Matching Cards와 동일한 인지 프로세스, UI만 다름 |
| F. Stroke Practice | 운동기억(motor recall) + 생성(generation) | **0.8** | 추적 후 기억에서 생성. 운동감각 인코딩 추가 |
| G. Listen & Draw | 청각인출 + 운동생성 | **0.9** | 단서 없이 청각만으로 생성. 이중 부호화(dual coding) |
| H. Radical Assembly | 분석적 재구성(analytical reconstruction) | **0.7** | 구성요소 분석 + 조합. 정교화(elaboration) 포함 |
| I. Particle Battle | 생성(cued recall) | **0.8** | 문맥 내 빈칸 채우기 = 단서회상. 표준 퀴즈와 유사 |
| J. Keigo Transformer | 변환(transformation) + 생성 | **1.0** | 규칙 적용 + 생성. 표준 퀴즈 이상의 인지 처리 |
| K. Context Detective | 추론(inference) + 단서회상 | **0.7** | 문맥에서 추론. 직접 회상은 아니지만 깊은 처리 요구 |
| L. shiritori | 자유회상(free recall) + 음운처리 | **0.6** | 자유연상이지만, 정확한 개념 매칭이 아닌 음운 기반 |

### 1.3 progress.json 반영 규칙

#### 정답 처리
```
concept.attempts += 1
concept.correct += weight  // 0.4~1.0 (소수점 허용)
```

**핵심 설계 결정**: `correct`를 정수가 아닌 **부동소수점**으로 변경합니다.

이유: 가중치 0.4인 Matching Cards에서 정답을 맞혀도 `correct`가 0.4만 올라갑니다.
배지 비율 계산(`correct/attempts`)에서 자연스럽게 게임의 낮은 인출 난이도가 반영됩니다.

#### 오답 처리
```
concept.attempts += 1
concept.correct += 0  // 오답은 게임 종류 무관하게 0
```

#### nextReview 갱신
- **가중 정답(weighted correct >= 0.7)**: 표준 SRS와 동일하게 다음 간격으로 이동
- **가중 정답(weighted correct < 0.7)**: 간격 유지(올리지도 리셋하지도 않음)
- **오답**: 간격 1d로 리셋 (기존과 동일)

이렇게 하면 Matching Cards(0.4)에서 맞혀도 SRS 간격이 올라가지 않습니다.
Particle Battle(0.8)에서 맞히면 간격이 올라갑니다.

```
// nextReview 갱신 의사코드
if (isCorrect) {
  if (gameWeight >= 0.7) {
    advanceToNextInterval();  // 1d->3d->7d->...
  } else {
    keepCurrentInterval();    // 간격 변동 없음, lastReview만 갱신
  }
} else {
  resetToOneDay();            // 기존과 동일
}
```

### 1.4 게이밍 방지(Anti-Gaming) 메커니즘

**문제**: 학습자가 쉬운 게임(Matching Cards)만 반복하여 XP를 축적하면서 실제 학습은 회피할 수 있습니다.

**대책**:

1. **동일 개념 동일 게임 쿨다운**: 같은 개념을 같은 게임으로 24시간 내 재시도 불가.
   progress.json에 `lastGameType` 필드를 추가합니다.

2. **게임 다양성 보너스**: 하나의 개념을 서로 다른 게임 유형으로 학습하면 XP 보너스 +5.
   이론적 근거: 인터리빙(interleaving) 효과 -- 동일 개념을 다양한 맥락에서 연습하면 전이(transfer)가 향상됩니다 (Rohrer & Taylor, 2007).

3. **배지 확정에 게임 다양성 요구**: 배지를 확정(3회 이상)하려면 최소 2가지 다른 평가 방식(게임 유형 또는 표준 퀴즈)에서의 시도가 필요합니다.

4. **승급 검증(Promotion Gate)**: 개념의 SRS 간격이 7d 이상으로 올라가려면, 가중치 0.7 이상인 게임이나 표준 퀴즈에서 최소 1회 정답이 필요합니다.
   - 낮은 가중치 게임만으로는 간격이 3d까지만 올라감
   - 이것은 Karpicke(2008)의 "인출 연습 효과"에 기반: 쉬운 재인으로는 장기기억 고정이 불충분

---

## 2. 인지부하 관리(Cognitive Load Management)

### 2.1 이론적 배경

Sweller(1988)의 인지부하 이론에 따르면:
- **내재적 부하(intrinsic load)**: 학습 자료 자체의 복잡도 (= 개념 난이도)
- **외재적 부하(extraneous load)**: 학습과 무관한 처리 비용 (= 게임 UI 조작)
- **본유적 부하(germane load)**: 스키마 구축에 기여하는 처리 (= 우리가 극대화할 것)

게임은 외재적 부하를 추가합니다. 따라서 내재적 부하가 높은 항목(새로운 개념)에서는 게임의 외재적 부하를 최소화해야 합니다.

### 2.2 세션 구성: NEW vs REVIEW 비율

| 학습자 레벨 | 플레이어 레벨 | 신규(NEW) | 복습(REVIEW) | 총 아이템 | 근거 |
|-------------|---------------|-----------|-------------|-----------|------|
| N5 초반 | 1-5 | 2-3 | 3-4 | **5-6** | 작업기억 용량(Miller, 7+-2)의 하한. 게임 UI 자체가 새로움 |
| N5 후반 | 6-10 | 3-4 | 4-5 | **7-8** | 게임 UI에 익숙해짐. 외재적 부하 감소 |
| N4 | 11-15 | 4-5 | 5-6 | **8-10** | 기본 패턴 숙지. 더 많은 항목 처리 가능 |
| N3 | 16-20 | 4-5 | 6-8 | **10-12** | 중급. 복습 비중 증가 (축적된 어휘) |
| N2-N1 | 21-30 | 5-6 | 8-10 | **12-15** | 상급. 정교화 연습에 집중 |

**핵심 규칙**: 신규 아이템 비율은 **전체의 40%를 초과하지 않습니다**.
근거: Atkinson(1972)의 최적 학습 모델에서, 새 항목 비율이 너무 높으면 간섭(interference)으로 학습 효율이 급감합니다.

### 2.3 세션 길이 제한

| 지표 | 임계값 | 조치 |
|------|--------|------|
| 시간 | **10분** (게임 시작 후) | 자연스러운 종료점 제안 |
| 연속 오답 | **3연속** | 난이도 하향 또는 복습 모드 전환 |
| 총 오답률 | **세션 내 60% 이상 오답** | 세션 중단 제안 + 복습 추천 |
| 아이템 수 | 위 테이블의 최대값 | 초과 시 "오늘은 여기까지" 메시지 |

**"10분 규칙"의 근거**: Oakley(2014)의 focused/diffuse 모드 이론. 10분 집중 후 짧은 휴식이 스키마 통합에 효과적입니다. 게임이라 몰입감이 높지만, 학습 효과는 10분 이후 체감 감소합니다.

### 2.4 게임별 인지부하 프로필

게임 선택 시, 현재 세션의 총 인지부하를 관리합니다:

| 게임 | 외재적 부하 | 적합한 내재적 부하 수준 |
|------|------------|----------------------|
| A. Matching Cards | 낮음 | 높은 내재적 부하 OK (새 개념도 가능) |
| D. Shuffle Puzzle | 중간 | 중간 이하 내재적 부하 |
| E. Snake Match | 낮음 | 높은 내재적 부하 OK |
| F. Stroke Practice | 중간 | 중간 이하 (한자 전용) |
| G. Listen & Draw | 높음 | 낮은 내재적 부하만 (이미 아는 글자) |
| H. Radical Assembly | 중간 | 중간 (부수 학습 후) |
| I. Particle Battle | 낮음 | 높은 내재적 부하 OK |
| J. Keigo Transformer | 높음 | 낮은 내재적 부하만 (숙지된 문법) |
| K. Context Detective | 중간 | 중간 이하 |
| L. shiritori | 중간 | 자유 (게임 자체가 복습) |

---

## 3. 학습 흐름 내 최적 게임 배치

### 3.1 학습 단계 모델

기존 시스템의 흐름:
```
[1단계] 도입(Introduction) → [2단계] 연습(Practice) → [3단계] 테스트(Quiz) → [4단계] 복습(Review)
```

게임을 통합한 확장 흐름:
```
[1] 도입 → [1.5] 인코딩 게임 → [2] 연습 → [2.5] 인출 게임 → [3] 테스트 → [4] 복습 게임/퀴즈
```

### 3.2 게임의 역할 분류

#### 인코딩 게임 (Encoding Games) -- 단계 1.5

새 개념을 처음 접한 직후, 초기 기억 흔적을 강화하는 게임.
**첫 노출을 보조**하지만 그 자체가 평가는 아닙니다.

| 게임 | 인코딩 역할 | 이유 |
|------|------------|------|
| A. Matching Cards | 의미-형태 연합 형성 | 짝 맞추기로 단어-뜻 연결 강화 |
| E. Snake Match | 의미-형태 연합 형성 | Matching Cards의 변형 |
| F. Stroke Practice (추적 단계) | 운동감각 인코딩 | 따라 쓰기는 인코딩. 기억에서 쓰기는 인출 |
| H. Radical Assembly | 구조적 분석 인코딩 | 한자를 부수로 분해/조합하며 구조 학습 |

#### 인출 연습 게임 (Retrieval Practice Games) -- 단계 2.5, 4

기억에서 정보를 끌어내야 하는 게임. 이것이 **진짜 학습**이 일어나는 곳입니다.
(Roediger & Karpicke, 2006: testing effect)

| 게임 | 인출 역할 | 이유 |
|------|----------|------|
| D. Shuffle Puzzle | 문법 구조 인출 | 문장 순서를 기억에서 재구성 |
| F. Stroke Practice (기억 단계) | 운동기억 인출 | 보지 않고 글자 재생산 |
| G. Listen & Draw | 청각-시각 교차 인출 | 소리만 듣고 글자 생성 = 강력한 인출 |
| I. Particle Battle | 문법 규칙 인출 | 빈칸 채우기 = cued recall |
| J. Keigo Transformer | 변환 규칙 인출 + 생성 | 가장 깊은 처리. 규칙 적용 + 생성 |
| K. Context Detective | 의미 추론 + 어휘 인출 | 문맥에서 역추론 |
| L. shiritori | 자유 어휘 인출 | 음운 단서 기반 자유연상 |

### 3.3 게임이 표준 퀴즈를 대체하는 조건 vs 보충하는 조건

#### 대체(REPLACE) 가능
- 가중치 >= 0.7인 인출 게임에서 정답 → 해당 개념의 표준 퀴즈를 건너뛸 수 있음
- 조건: 동일 세션 내에서만. 다음 SRS 복습 시점에서는 표준 퀴즈 또는 다른 인출 게임 필요

#### 보충(SUPPLEMENT)만 가능
- 가중치 < 0.7인 인코딩 게임 → 표준 퀴즈를 대체할 수 없음
- Matching Cards로 아무리 많이 맞혀도, 그 개념은 여전히 표준 퀴즈/인출 게임이 필요

#### 흐름 예시

**새 단어 "経済(けいざい, 경제)" 학습 시**:
```
1. [도입] 경제의 의미, 읽기, 예문 3개 제시
2. [인코딩 게임] Matching Cards에서 경제 포함 4쌍 매칭 (가중 0.4)
   → attempts +1, correct +0.4, nextReview = 내일
3. [표준 퀴즈] 4지선다 "경제는 일본어로?" (가중 1.0)
   → attempts +1, correct +1.0, nextReview 갱신
4. [3일 후 복습] Particle Battle "日本の___は成長している" (가중 0.8)
   → attempts +1, 정답 시 correct +0.8, 간격 7d로 상승
5. [7일 후 복습] Context Detective 또는 표준 퀴즈
```

**한자 "食" 학습 시**:
```
1. [도입] 食의 음독, 훈독, 의미, 부수 제시
2. [인코딩 게임] Radical Assembly로 食 조합 (가중 0.7)
3. [인코딩 게임] Stroke Practice 추적 단계 (가중치 없음 - 인코딩 전용)
4. [인출 게임] Stroke Practice 기억 단계 (가중 0.8)
5. [3일 후] Listen & Draw "しょく" 듣고 食 쓰기 (가중 0.9)
6. [7일 후] 표준 퀴즈 또는 인출 게임
```

### 3.4 카테고리별 추천 게임 매핑

| 카테고리 | 인코딩 단계 추천 | 인출 단계 추천 | 비추천 |
|----------|-----------------|---------------|--------|
| vocab | Matching Cards, Snake Match | Context Detective, shiritori | Stroke Practice (한자 전용) |
| grammar | - | Shuffle Puzzle, Particle Battle, Keigo Transformer | Matching Cards (문법에 부적합) |
| kanji | Radical Assembly, Stroke Practice(추적) | Stroke Practice(기억), Listen & Draw | shiritori (한자 특화 아님) |
| reading | - | Context Detective | 대부분 게임 부적합 (reading-tutor가 적합) |
| business | - | Keigo Transformer, Particle Battle | Matching Cards (비즈니스 깊이 부족) |

---

## 4. 인코딩 깊이 위계(Encoding Depth Hierarchy)

Craik & Lockhart(1972)의 처리 수준(Levels of Processing) 이론에 따라, 더 깊은 인지 처리를 요구하는 활동이 더 강한 기억 흔적을 형성합니다.

### 4.1 얕은 처리 -> 깊은 처리 순서

```
[얕은 처리] ─────────────────────────────── [깊은 처리]

A. Matching     E. Snake    L. shiritori   K. Context    D. Shuffle
Cards           Match                      Detective     Puzzle
(0.4)           (0.4)       (0.6)          (0.7)         (0.7)

        H. Radical     I. Particle    F. Stroke     G. Listen    J. Keigo
        Assembly       Battle         Practice      & Draw       Transformer
        (0.7)          (0.8)          (0.8)         (0.9)        (1.0)
```

### 4.2 상세 분석

#### 수준 1: 지각적 처리(Perceptual Processing) -- 가장 얕음

**A. Matching Cards (가중 0.4)**
- 인지 프로세스: 시각적 패턴 매칭, 재인(recognition), 작업기억 내 위치 기억
- 학습과학적 분석: 재인은 회상보다 쉬운 기억 과제(Mandler, 1980). 카드 위치의 공간 기억이 주요 과제이며, 어휘 의미 처리는 부차적. "알고 있다"는 **친숙성(familiarity)** 판단으로 충분하며 **회상(recollection)**이 불필요.
- 강점: 초기 연합 형성, 진입장벽 낮음, 동기부여
- 약점: 얕은 처리로 장기기억 전이 약함

**E. Snake Match (가중 0.4)**
- 인지 프로세스: Matching Cards와 동일한 재인 기반. 시각 탐색(visual search) 추가
- 학습과학적 분석: 그리드에서 쌍을 찾는 것은 주의(attention) 자원을 시각 탐색에 소모. 의미 처리 깊이는 Matching Cards와 동등.
- 강점: 시각적 변형으로 인터리빙 효과 (같은 개념을 다른 형식으로)
- 약점: 시각 탐색의 외재적 부하가 의미 처리를 방해할 수 있음

#### 수준 2: 음운적/구조적 처리(Phonological/Structural Processing)

**L. shiritori (가중 0.6)**
- 인지 프로세스: 음운 분석(phonological analysis), 어휘 접근(lexical access), 자유연상(free association)
- 학습과학적 분석: 단어의 마지막 음절로 새 단어를 찾는 것은 음운 루프(phonological loop)를 활용. 자유회상이지만 음운 제약 하의 연상이므로 의미적 깊이는 제한적. 그러나 어휘 네트워크(lexical network)의 활성화 확산(spreading activation, Collins & Loftus, 1975)을 촉진.
- 강점: 어휘 접근 속도(fluency) 향상, 재미있음
- 약점: 이미 아는 쉬운 단어만 반복 사용 가능. 특정 개념 타겟팅 어려움

#### 수준 3: 의미적 처리(Semantic Processing)

**K. Context Detective (가중 0.7)**
- 인지 프로세스: 문맥 추론(contextual inference), 의미 분석, 단서 기반 회상
- 학습과학적 분석: 빠진 단어를 문맥에서 추론하는 것은 schema-driven processing. 학습자가 문맥적 단서를 통합하여 의미를 구성해야 하므로 정교화(elaboration)가 발생. Nation(2001)의 어휘 학습 모델에서 "contextual guessing"은 중간 수준의 효과적 전략.
- 강점: 실제 읽기 전략과 연결, 의미 네트워크 강화
- 약점: 추론에 성공해도 그 단어를 생성(production)할 수 있다는 보장 없음

**H. Radical Assembly (가중 0.7)**
- 인지 프로세스: 분석(decomposition), 구조적 인코딩, 재구성(reconstruction)
- 학습과학적 분석: 한자를 부수로 분해하고 재조합하는 것은 "구성요소 분석(componential analysis)"으로, 전체를 그대로 외우는 것보다 깊은 처리. Heisig(2007)의 RTK(Remembering the Kanji) 방법론과 유사하게 구조적 이해를 촉진.
- 강점: 한자 구조 이해, 유사 한자 변별, 부수 지식 강화
- 약점: 의미/읽기 인출은 포함되지 않음. 구조만 안다고 읽을 수 있는 것은 아님

**D. Shuffle Puzzle (가중 0.7)**
- 인지 프로세스: 통사적 분석(syntactic analysis), 순서 재구성, 절차적 지식 활용
- 학습과학적 분석: 문장의 어순을 재구성하는 것은 통사 규칙의 절차적 지식(procedural knowledge)을 활성화. DeKeyser(2007)의 skill acquisition theory에서 "연습을 통한 절차화"에 해당. 단순 재인보다 깊지만, 단어들이 모두 제시되어 있으므로 어휘 인출은 불필요.
- 강점: 문법 구조의 내재화, 어순 감각 향상
- 약점: 선택지가 모두 보이므로 "생성" 부담이 낮음

#### 수준 4: 생성적 처리(Generative Processing)

**I. Particle Battle (가중 0.8)**
- 인지 프로세스: 문법 규칙 인출, 단서 회상(cued recall), 문맥 분석
- 학습과학적 분석: 빈칸에 적절한 조사를 채우는 것은 "generation effect"(Slamecka & Graf, 1978)를 활용. 학습자가 직접 답을 생성해야 하므로 재인보다 깊은 처리. 일본어 조사는 미세한 의미 차이를 가지므로 의미적 정교화도 필요.
- 강점: 조사/문법의 정확한 사용 연습, 높은 전이 가능성
- 약점: 조사에 특화되어 범용성 제한

**F. Stroke Practice -- 기억 단계 (가중 0.8)**
- 인지 프로세스: 운동기억 인출(motor memory retrieval), 시각-운동 변환, 순서 재생산
- 학습과학적 분석: 추적(tracing) 후 기억에서 쓰는 것은 이중 부호화(Paivio, 1986)의 실현. 시각적 형태 + 운동 감각이 결합되어 두 가지 기억 흔적을 형성. Longcamp et al.(2005)의 연구에서 필기가 타이핑보다 글자 인식에 효과적임을 입증.
- 강점: 운동감각 기억 형성, 한자 형태의 정밀한 인코딩
- 약점: 의미/읽기 처리는 포함하지 않음

#### 수준 5: 변환적 처리(Transformative Processing) -- 가장 깊음

**G. Listen & Draw (가중 0.9)**
- 인지 프로세스: 청각 입력 → 음운 분석 → 의미 접근 → 시각 형태 인출 → 운동 출력
- 학습과학적 분석: 4단계 변환이 필요한 과제. Baddeley(2000)의 작업기억 모델에서 음운 루프 → 시공간 스케치패드 → 일화적 버퍼의 세 구성요소를 모두 활용. 외부 시각 단서가 전혀 없으므로 완전한 생성(pure generation). 이것이 "바람직한 어려움"의 전형적 예시.
- 강점: 가장 강력한 다중감각 인코딩, 높은 인출 난이도 = 강한 기억
- 약점: 인지부하가 매우 높아 초보자에게 좌절감 유발 가능

**J. Keigo Transformer (가중 1.0)**
- 인지 프로세스: 규칙 분석 → 형태 변환 → 맥락 적합성 판단 → 생성
- 학습과학적 분석: Anderson(1983)의 ACT-R 이론에서 선언적 지식(경어 규칙)을 절차적 지식(실제 변환)으로 전환하는 과정. 단순 회상이 아닌 **변환(transformation)**을 요구하므로 가장 깊은 처리. 경어는 화용적(pragmatic) 지식까지 필요하여 사회언어학적 처리도 포함.
- 강점: 표준 퀴즈 이상의 인지 깊이. 실제 사용 능력(proficiency)에 직결
- 약점: N3 이상에서만 의미 있음. 초보자에게는 불가능

---

## 5. 안티패턴(Anti-Patterns): 하지 말아야 할 것들

### 안티패턴 1: "유창성 환상(Fluency Illusion)"

**증상**: Matching Cards에서 빠르게 쌍을 맞추면서 "나 잘하고 있다"고 느끼지만, 실제로는 재인만 하고 있어서 정작 말하거나 쓸 때 인출이 안 됨.

**학습과학**: Kornell & Bjork(2008)의 "판단과 실제 학습의 괴리(metacognitive illusion)". 재인의 유창성을 학습의 증거로 착각하는 현상.

**방지책**:
- 인코딩 게임 후 반드시 인출 게임/퀴즈 연결
- 대시보드에서 "인출 연습 비율" 표시 (인출 게임 비율이 50% 이하면 경고)
- "쉬운 게임만 하고 있어요. 도전해볼까요?" 넛지 메시지

### 안티패턴 2: "XP 파밍(XP Farming)"

**증상**: 이미 마스터한 쉬운 개념으로 게임을 반복하여 XP만 축적. 레벨은 올라가지만 실력은 정체.

**학습과학**: Bjork의 "저장 강도(storage strength) vs 인출 강도(retrieval strength)" 구분. 이미 저장 강도가 높은 항목을 반복하는 것은 인출 강도만 일시적으로 올릴 뿐, 새로운 학습이 없음.

**방지책**:
- 파랑 배지(마스터) 개념은 게임 XP의 50%만 부여
- 게임 아이템 선정 시 복습 대상/약점 개념 우선 배치
- "마스터한 개념이에요! 새로운 개념에 도전해보세요" 메시지

```
// XP 계산 의사코드
if (concept.badge === "blue" && concept.attempts >= 10) {
  xp = baseXP * 0.5;  // 마스터 개념 XP 반감
}
```

### 안티패턴 3: "맥락 없는 반복(Decontextualized Drilling)"

**증상**: 단어 카드만 반복적으로 매칭하면서, 실제 문장이나 맥락에서 그 단어를 만나면 의미를 파악하지 못함.

**학습과학**: 전이 적절 처리(Transfer-Appropriate Processing, Morris et al., 1977). 학습 시의 처리 방식과 테스트 시의 처리 방식이 일치할 때 기억이 가장 잘 인출됨. 고립된 단어 매칭은 실제 읽기/듣기/말하기의 맥락적 처리와 불일치.

**방지책**:
- 모든 게임에서 가능하면 예문/문맥 포함
- Matching Cards도 "단어-뜻" 뿐 아니라 "문장-해석" 매칭 변형 제공
- Context Detective, Particle Battle 등 문맥 기반 게임을 적극 추천

### 안티패턴 4: "과잉 게이미피케이션(Over-Gamification)"

**증상**: XP, 뱃지, 스트릭, 레벨업, 리더보드... 외재적 동기(extrinsic motivation)에만 의존하여 학습하다가, 보상이 사라지면(또는 익숙해지면) 학습 동기도 사라짐.

**학습과학**: Deci & Ryan(1985)의 자기결정이론(Self-Determination Theory). 외재적 보상의 과도한 사용은 내재적 동기(intrinsic motivation)를 잠식할 수 있음 (과정당화 효과, overjustification effect).

**방지책**:
- 게이미피케이션 요소는 **보조적** 역할만 수행
- "오늘 새로 알게 된 것" 요약을 XP 보고보다 먼저 표시
- 레벨보다 "마스터한 개념 수"를 더 눈에 띄게 표시
- 스트릭 압박 완화: 1일 빠져도 스트릭 유지 (freeze 1회/주)

### 안티패턴 5: "난이도 회피(Difficulty Avoidance)"

**증상**: 학습자가 자유 선택 시 항상 쉬운 게임/쉬운 개념만 선택. 어려운 개념을 계속 회피하여 약점이 고착화.

**학습과학**: Metcalfe(2009)의 "근접 학습 영역(region of proximal learning)" 이론. 너무 쉬운 것은 학습이 없고, 너무 어려운 것은 좌절만 줌. 최적 난이도는 "약간 어렵지만 노력하면 가능한" 수준.

**방지책**:
- 게임 세션의 아이템 중 최소 30%는 시스템이 약점 기반으로 강제 배치
- 자유 선택 + 강제 배치 혼합: 학습자 선택 70% + 시스템 추천 30%
- 약점 개념 정답 시 추가 XP 보너스 (+5 "도전 보너스")

### 안티패턴 6: "벼락치기 세션(Massed Practice)"

**증상**: 일주일에 1번, 2시간씩 게임. 매일 10분보다 효율이 떨어지지만 학습자는 "많이 했다"고 느낌.

**학습과학**: 분산 효과(Spacing Effect, Ebbinghaus, 1885; Cepeda et al., 2006). 동일 총 학습 시간이라도 분산하여 학습한 것이 집중적으로 학습한 것보다 장기 기억에 효과적. 이것은 학습과학에서 가장 robust한 발견 중 하나.

**방지책**:
- 1회 게임 세션 10분 제한 (하드캡)
- 10분 초과 시 XP 효율 50% 감소 (diminishing returns 명시)
- "오늘 10분 완료! 내일 다시 만나요" 메시지로 분산 학습 유도
- 연속 3세션 이상 시 쿨다운 1시간

### 안티패턴 7: "피드백 무시(Feedback Neglect)"

**증상**: 오답 후 해설을 읽지 않고 바로 다음 문제로 넘어감. 게임의 빠른 템포가 피드백 처리를 방해.

**학습과학**: Butler et al.(2008)의 연구에서, 피드백을 처리할 시간이 주어지지 않으면 테스트 효과(testing effect)가 크게 감소함을 입증.

**방지책**:
- 오답 시 최소 3초 피드백 표시 강제 (게임 UI에서 "다음" 버튼 3초 후 활성화)
- 피드백에 "왜 틀렸는지" 1줄 핵심 설명 필수 포함
- 세션 종료 시 "오늘의 오답 복습" 코너 (틀린 문제 재출제)

---

## 6. 구현 우선순위 요약

### Phase 1: 핵심 통합 (필수)
- [ ] progress.json의 `correct` 필드를 float로 변경
- [ ] 게임별 가중치 테이블 적용
- [ ] nextReview 갱신 로직에 가중치 임계값(0.7) 반영
- [ ] 세션 길이 제한 (10분, 아이템 수 상한)
- [ ] 오답 피드백 3초 강제 표시

### Phase 2: 게이밍 방지 (중요)
- [ ] 동일 개념 동일 게임 24시간 쿨다운
- [ ] 승급 검증(Promotion Gate): 간격 7d 이상은 가중치 0.7+ 필요
- [ ] 마스터 개념 XP 반감
- [ ] 약점 개념 30% 강제 배치

### Phase 3: 최적화 (권장)
- [ ] 게임 다양성 보너스
- [ ] 배지 확정 시 2가지 이상 평가 방식 요구
- [ ] 인코딩 -> 인출 자동 흐름 구성
- [ ] 분산 학습 넛지 (10분 초과 XP 감소)
- [ ] 대시보드에 "인출 연습 비율" 표시

---

## 참고문헌

- Anderson, J. R. (1983). *The Architecture of Cognition*. Harvard University Press.
- Atkinson, R. C. (1972). Optimizing the learning of a second-language vocabulary. *Journal of Experimental Psychology*, 96(1), 124-129.
- Baddeley, A. D. (2000). The episodic buffer: a new component of working memory? *Trends in Cognitive Sciences*, 4(11), 417-423.
- Bjork, R. A. (1994). Memory and metamemory considerations in the training of human beings. In J. Metcalfe & A. Shimamura (Eds.), *Metacognition*.
- Butler, A. C., Karpicke, J. D., & Roediger, H. L. (2008). Correcting a metacognitive error. *Journal of Experimental Psychology: Learning, Memory, and Cognition*, 34(4), 918-928.
- Cepeda, N. J., et al. (2006). Distributed practice in verbal recall tasks. *Psychological Bulletin*, 132(3), 354-380.
- Collins, A. M., & Loftus, E. F. (1975). A spreading-activation theory of semantic processing. *Psychological Review*, 82(6), 407-428.
- Craik, F. I. M., & Lockhart, R. S. (1972). Levels of processing. *Journal of Verbal Learning and Verbal Behavior*, 11(6), 671-684.
- Deci, E. L., & Ryan, R. M. (1985). *Intrinsic Motivation and Self-Determination in Human Behavior*. Plenum.
- DeKeyser, R. (2007). Skill acquisition theory. In B. VanPatten & J. Williams (Eds.), *Theories in Second Language Acquisition*.
- Ebbinghaus, H. (1885). *Memory: A Contribution to Experimental Psychology*.
- Karpicke, J. D., & Roediger, H. L. (2008). The critical importance of retrieval for learning. *Science*, 319(5865), 966-968.
- Kornell, N., & Bjork, R. A. (2008). Learning concepts and categories. *Psychological Science*, 19(6), 585-592.
- Longcamp, M., et al. (2005). The influence of writing practice on letter recognition in preschool children. *Acta Psychologica*, 119(1), 67-79.
- Mandler, G. (1980). Recognizing: The judgment of previous occurrence. *Psychological Review*, 87(3), 252-271.
- Metcalfe, J. (2009). Metacognitive judgments and control of study. *Current Directions in Psychological Science*, 18(3), 159-163.
- Morris, C. D., Bransford, J. D., & Franks, J. J. (1977). Levels of processing versus transfer appropriate processing. *Journal of Verbal Learning and Verbal Behavior*, 16(5), 519-533.
- Nation, I. S. P. (2001). *Learning Vocabulary in Another Language*. Cambridge University Press.
- Oakley, B. (2014). *A Mind for Numbers*. TarcherPerigee.
- Paivio, A. (1986). *Mental Representations: A Dual Coding Approach*. Oxford University Press.
- Roediger, H. L., & Karpicke, J. D. (2006). Test-enhanced learning. *Psychological Science*, 17(3), 249-255.
- Rohrer, D., & Taylor, K. (2007). The shuffling of mathematics problems improves learning. *Instructional Science*, 35(6), 481-498.
- Slamecka, N. J., & Graf, P. (1978). The generation effect. *Journal of Experimental Psychology: Human Learning and Memory*, 4(6), 592-604.
- Sweller, J. (1988). Cognitive load during problem solving. *Cognitive Science*, 12(2), 257-285.
