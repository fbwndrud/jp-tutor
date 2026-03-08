# jp-tutor

일본어 학습 튜터 Claude Cowork 플러그인.

`/jp` 하나로 JLPT 시험 준비, 비즈니스 일본어, 교재 기반 학습까지 — 개념별 마스터리 추적과 에빙하우스 복습으로 체계적으로 학습합니다.

## 주요 기능

- **3가지 학습 프로필**: JLPT 자율 학습 / 비즈니스 일본어 / 교재 기반 커스텀
- **15개 명령어**: 어휘, 문법, 한자, 독해, 작문, 회화, 번역, 문화, 비즈니스, 교재, 복습 등
- **3개 전문 에이전트**: 독해 튜터, 작문 튜터, 회화 파트너
- **에빙하우스 복습**: 1일→3일→7일→14일→30일→60일 간격 자동 스케줄링
- **게이미피케이션**: XP/30단계 레벨, 스트릭, 업적 배지
- **대시보드 & 플래시카드**: 인터랙티브 HTML (다크모드, 모바일 지원)

## 설치

Claude Cowork에서:

1. **설정** → **플러그인** → **URL로 마켓플레이스 추가**
2. URL 입력:
   ```
   https://github.com/fbwndrud/jp-tutor.git
   ```
3. `jp-tutor` 플러그인 설치

## 시작하기

```
/jp:start
```

3가지 프로필 중 선택 (복수 선택 가능):
1. **자율 학습** — JLPT 시험 준비 / 자유 학습
2. **비즈니스** — 업무용 일본어 (경어, 메일, 회의)
3. **수업 병행** — 교재 기반 예습/복습

## 명령어

### 일상 학습

| 명령어 | 설명 |
|--------|------|
| `/jp` | 범용 진입점 — 자연어로 말하면 자동 라우팅 |
| `/jp:daily` | 오늘의 학습 세트 (복습 + 새 개념 혼합) |
| `/jp:review` | 에빙하우스 기반 스마트 복습 |
| `/jp:dashboard` | 학습 현황 대시보드 |

### 학습 모드

| 명령어 | 모델 | 설명 |
|--------|------|------|
| `/jp:vocab` | haiku | 어휘 4지선다 퀴즈 |
| `/jp:grammar` | sonnet | 문법 패턴 설명 + 연습 문제 |
| `/jp:kanji` | haiku | 한자 플래시카드 + 퀴즈 |
| `/jp:translate` | sonnet | 한일/일한 번역 연습 |
| `/jp:culture` | sonnet | 문화/관용표현 학습 |

### 특화 모드

| 명령어 | 설명 |
|--------|------|
| `/jp:biz` | 비즈니스 일본어 (경어/메일/회의/전화/업종별) |
| `/jp:textbook` | 교재 기반 예습/복습/확인 테스트 |

### 관리

| 명령어 | 설명 |
|--------|------|
| `/jp:start` | 온보딩 + 프로필 설정 |
| `/jp:update` | 진도 동기화 + CLAUDE.md 핫캐시 갱신 |
| `/jp:export` | CSV/Anki 형식 내보내기 |
| `/jp:report` | 주간/월간 학습 리포트 |

## 에이전트

멀티턴 학습이 필요한 모드는 전문 에이전트가 담당합니다:

| 에이전트 | 모델 | 트리거 예시 |
|---------|------|------------|
| **reading-tutor** | sonnet | "독해 연습", "지문 읽기" |
| **writing-tutor** | opus | "작문 교정", "일기 써볼래" |
| **conversation-partner** | opus | "회화 연습", "편의점 상황 연습" |

## 마스터리 추적

개념 단위(단어, 문법, 한자)로 숙련도를 추적합니다:

| 배지 | 정답률 | 의미 |
|------|--------|------|
| 🟦 | >= 80% | 마스터 |
| 🟩 | >= 60% | 숙련 |
| 🟨 | >= 40% | 학습중 |
| 🟥 | < 40% | 보강 필요 |
| ⬜ | 미측정 | 아직 테스트 안 함 |

## 에빙하우스 복습

정답/오답에 따라 복습 간격이 자동 조절됩니다:

```
정답 → 1일 → 3일 → 7일 → 14일 → 30일 → 60일 → 마스터 유지
오답 → 1일로 리셋
```

`/jp:review`로 오늘 복습할 개념을 자동으로 모아줍니다.

## 게이미피케이션

| 행동 | XP |
|------|-----|
| 정답 | +10 |
| 오답 (시도 보상) | +3 |
| Daily 완료 | +50 |
| 스트릭 보너스 | 연속일수 x 5 |
| 새 개념 학습 | +15 |

30단계 레벨 시스템: 초심자(Lv.1) → 센세이(Lv.30)

업적 배지: 스트릭(3/7/30/100/365일), 마스터리(10/50/100개), 약점 극복, 퍼펙트 라운드 등

## 비즈니스 일본어 (`/jp:biz`)

6개 모듈로 구성된 비즈니스 커리큘럼:

1. 기본 비즈니스 매너 (인사, 명함, 경어 기초)
2. 비즈니스 메일 (의뢰/사과/보고 메일)
3. 회의/보고 (ほうれんそう, 프레젠테이션)
4. 전화 응대 (받기/걸기, 부재 대응)
5. 업종별 용어 (IT/제조/금융, 커스텀 가능)
6. 고급 비즈니스 (협상, 계약, 접대)

회사 내부 용어도 등록하여 퀴즈/회화에 반영할 수 있습니다.

## 교재 기반 학습 (`/jp:textbook`)

수업 교재를 등록하면 진도에 맞춘 학습이 가능합니다:

```
/jp:textbook add "민나노니홍고 초급1"
```

- **preview**: 수업 전 예습 (새 어휘/문법 소개)
- **review**: 수업 후 복습 (핵심 포인트 + 빈칸 채우기)
- **quiz**: 확인 테스트 (단원별 / 누적)

"이번 주 수업이 12과까지야" → 자동으로 진도 동기화

## JLPT 커리큘럼

N5부터 N1까지 내장 커리큘럼:

| 레벨 | 어휘 | 한자 | 목표 |
|------|------|------|------|
| N5 | ~800어 | ~100자 | 기본 인사, 자기소개 |
| N4 | ~1,500어 | ~300자 | 일상 대화 |
| N3 | ~3,750어 | ~650자 | 뉴스/신문 이해 |
| N2 | ~6,000어 | ~1,000자 | 비즈니스 대응 |
| N1 | ~10,000어 | ~2,000자 | 전문적 토론 |

## 데이터

- 진도 데이터: `jp-data/progress.json` (프로젝트에 자동 생성)
- 학습 로그: `jp-data/STUDY-LOG.md`
- 대시보드: `jp-data/dashboard.html`
- 내보내기: `jp-data/exports/`
- `.gitignore`에 `jp-data/` 추가를 권장합니다

## 플러그인 구조

```
.claude-plugin/
└── marketplace.json
jp-tutor/
├── .claude-plugin/plugin.json
├── README.md
├── commands/
│   ├── jp.md                  # /jp 범용 진입점
│   ├── jp-start.md            # /jp:start 온보딩
│   ├── jp-daily.md            # /jp:daily 일일 학습
│   ├── jp-review.md           # /jp:review 스마트 복습
│   ├── jp-update.md           # /jp:update 진도 동기화
│   ├── jp-dashboard.md        # /jp:dashboard 대시보드
│   ├── jp-vocab.md            # /jp:vocab 어휘 퀴즈
│   ├── jp-grammar.md          # /jp:grammar 문법 연습
│   ├── jp-kanji.md            # /jp:kanji 한자 학습
│   ├── jp-translate.md        # /jp:translate 번역 연습
│   ├── jp-culture.md          # /jp:culture 문화/관용표현
│   ├── jp-biz.md              # /jp:biz 비즈니스 일본어
│   ├── jp-textbook.md         # /jp:textbook 교재 기반
│   ├── jp-export.md           # /jp:export 내보내기
│   └── jp-report.md           # /jp:report 리포트
├── agents/
│   ├── reading-tutor.md       # 독해 튜터 (sonnet)
│   ├── writing-tutor.md       # 작문 튜터 (opus)
│   └── conversation-partner.md # 회화 파트너 (opus)
└── skills/jp/
    ├── SKILL.md               # 핵심 라우팅/규칙
    └── references/
        ├── quiz-rules.md
        ├── exercise-types.md
        ├── grading-rubrics.md
        ├── curricula/
        │   ├── japanese-jlpt.md
        │   └── business-japanese.md
        └── templates/
            ├── dashboard.html
            └── flashcard.html
```

## 라이선스

MIT
