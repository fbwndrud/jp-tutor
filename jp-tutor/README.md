# jp-tutor

Claude Cowork용 일본어 학습 플러그인.

## 학습 프로필

1. **JLPT 독학** - N5~N1 단계별 시험 준비
2. **비즈니스 일본어** - 경어, 이메일, 회의, 전화 등 실무 일본어
3. **교재 기반** - 사용자 교재 PDF/사진을 기반으로 복습 및 연습

## 명령어

| 명령어 | 설명 |
|--------|------|
| `/jp` | 메인 진입점. 자연어로 요청하면 자동 라우팅 |
| `/jp:start` | 초기 설정 (레벨 선택, 프로필 구성) |
| `/jp:daily` | 오늘의 학습 세션 시작 |
| `/jp:review` | 에빙하우스 기반 복습 |
| `/jp:vocab` | 어휘 퀴즈 |
| `/jp:grammar` | 문법 연습 |
| `/jp:kanji` | 한자 학습 |
| `/jp:translate` | 번역 연습 |
| `/jp:culture` | 문화/관용표현 학습 |
| `/jp:biz` | 비즈니스 일본어 모드 |
| `/jp:textbook` | 교재 기반 학습 모드 |
| `/jp:dashboard` | 학습 현황 대시보드 |
| `/jp:update` | 진행 데이터 업데이트/동기화 |

## 에이전트

- **reading-tutor** - 독해 지도 (Sonnet)
- **writing-tutor** - 작문 첨삭 (Opus)
- **conversation-partner** - 회화 연습 파트너 (Opus)
