---
model: sonnet
description: "비즈니스 일본어 학습 -- 경어/메일/회의/전화/업종별 용어"
argument-hint: "[모듈: email|meeting|keigo|phone|industry] [주제]"
---

# 비즈니스 일본어 (/jp:biz)

## 데이터 로드

1. `jp-data/progress.json`을 읽습니다.
2. 파일이 없으면 → "/jp:start 명령어로 먼저 학습 프로필을 설정해주세요." 안내 후 종료.
3. `business` 섹션과 프로필 레벨을 확인합니다.

## 비즈니스 프로필 확인

progress.json의 `business` 필드에서:
- `modules`: 모듈별 진행 상태 (email, meeting, keigo, phone, industry)
- `company_terms`: 사용자 맞춤 회사 용어 (있는 경우)

## 모듈 라우팅

인자를 파싱하여 모듈을 결정합니다:

### 모듈 미지정 시
progress.json의 business.modules를 분석하여 자동 추천:
1. 아직 시작하지 않은 모듈 우선
2. 정답률이 낮은(🟥/🟨) 모듈 우선
3. "추천 모듈: [모듈명] — [이유]" 형태로 안내

### email — 비즈니스 이메일
- `skills/jp/references/curricula/business-japanese.md` 모듈 2 참조
- 이메일 템플릿 학습: 제목 → 서두 인사 → 본문 → 맺음말 → 서명의 전체 구조
- 상황별 이메일 작성 연습 (의뢰, 사과, 보고, 문의, 일정 조율, 감사)
- 사내(社内) vs 사외(社外) 차이를 반드시 명시
- 연습: 주어진 상황에 맞는 이메일 전문 작성 → 첨삭 피드백

### meeting — 회의/보고
- `skills/jp/references/curricula/business-japanese.md` 모듈 3 참조
- 상황별 표현 학습: 보련상(報連相), 회의 진행, 프레젠테이션, 의견 표현
- 회화 시뮬레이션: 회의 상황 설정 → 롤플레이 (4~6턴)
- 사후 피드백: 경어 적절성, 표현 다양성 평가

### keigo — 경어 변환 드릴
- `skills/jp/references/curricula/business-japanese.md` 모듈 1 참조
- 경어 3종 변환 드릴: 존경어(尊敬語) ↔ 겸양어(謙譲語) ↔ 정중어(丁寧語)
- 4문제 1라운드, `skills/jp/references/quiz-rules.md` 적용
- 문맥 지정: 상사(上司) vs 동료(同僚) vs 부하(部下), 사내 vs 사외
- 이중경어 주의 사항 해설 포함

### phone — 전화 응대
- `skills/jp/references/curricula/business-japanese.md` 모듈 4 참조
- 전화 응대 롤플레이: 받기, 걸기, 부재 대응, 클레임 대응
- 상황 설정 후 텍스트 대화 (4~6턴)
- 대화 중 교정 없음, 종료 후 피드백 제공
- `skills/jp/references/grading-rubrics.md` 회화 채점 기준 적용

### industry — 업종별 용어
- `skills/jp/references/curricula/business-japanese.md` 모듈 5 참조
- 학습자의 업종 확인 (progress.json 또는 질문)
- 업종별 핵심 용어 플래시카드 제시 (5~10개)
- 용어 퀴즈: 4지선다 4문제, `skills/jp/references/quiz-rules.md` 적용
- company_terms가 있으면 커스텀 용어도 포함

## 비즈니스 특화 규칙

1. 모든 예문은 실제 업무 상황 기반으로 작성
2. 경어 레벨을 항상 명시: 社内/社外, 上司/同僚/部下
3. 이메일 연습은 제목부터 맺음말까지 전체 구조 포함
4. progress.json의 business.company_terms가 있으면 예문에 반영

## 배지 및 진행 추적

- 비즈니스 모듈별로 별도 배지 추적: `business.modules.{module}.concepts`
- 퀴즈 결과: correct/attempts 업데이트, nextReview 갱신
- 롤플레이 결과: business.modules.{module}.sessions에 등급 기록
- XP: 퀴즈 정답 +10, 롤플레이 세션 +25
- 배지 계산은 `skills/jp/references/grading-rubrics.md` 기준 적용

## 세션 종료

1. progress.json 업데이트
2. 학습 요약: 모듈명, 학습 항목 수, 정답률, 획득 XP
3. 다음 추천: 미완료 모듈 또는 복습 필요 항목 안내
