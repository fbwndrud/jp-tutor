# CLAUDE.md — jp-tutor 프로젝트 규칙

## 반복 실수 방지 체크리스트

### 1. 버전은 기능 커밋과 함께 올린다
- 기능 추가/변경 커밋 시 **반드시** `plugin.json`과 `marketplace.json`의 `version`을 함께 업데이트
- 별도 "chore: bump version" 커밋이 아니라, feat 커밋에 포함시킨다
- 파일 위치: `jp-tutor/.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`

### 2. README는 기능 변경 시 같이 업데이트
- 새 기능 추가하면 `README.md`도 같은 커밋 또는 직후에 반영
- 유저가 따로 요청하기 전에 알아서 한다

### 3. 새 기능은 모든 관련 템플릿에 반영
- TTS 같은 공통 기능 추가 시, **모든 HTML 템플릿**에 적용되었는지 확인
- 현재 템플릿 목록 (4개):
  - `jp-tutor/skills/jp/references/templates/tts-player.html`
  - `jp-tutor/skills/jp/references/templates/flashcard.html`
  - `jp-tutor/skills/jp/references/templates/kana-chart.html`
  - `jp-tutor/skills/jp/references/templates/dashboard.html` (통계 전용, TTS 불필요)
- 하나만 고치고 나머지를 빠뜨리지 않는다

### 4. HTML 템플릿 DATA 주입 패턴
- 모든 템플릿은 `const DATA = null;` 패턴으로 데이터를 주입받음
- IIFE 앞에 반드시 **방어적 세미콜론** `;(function() {` 사용
- 이유: 데이터 치환 시 `const DATA = {...}` 뒤에 세미콜론이 빠지면 `{...}(function()` 로 해석되어 "is not a function" 에러 발생
- 새 템플릿 작성 시 이 패턴 필수:
  ```js
  const DATA = null;

  ;(function() {
    // ...
  })();
  ```

### 5. HTML 템플릿 간 디자인 토큰 통일
- CSS 변수 (--bg, --radius, --shadow 등)는 4개 템플릿 모두 동일하게 유지
- 디자인 테마: "Wabi-Sabi Digital" — 일본 잉크 & 종이 미학
- Display 폰트: `'Shippori Mincho', serif` (일본어 제목/카나 표시용)
- Body 폰트: `'Noto Sans JP', sans-serif`
- Google Fonts CDN: `Shippori+Mincho:wght@400;600;700;800&family=Noto+Sans+JP:wght@300;400;500;600;700`
- 핵심 색상: paper `#F5F0E8`, ink `#2C2C2C`, accent `#3D5A80`, vermillion `#C1440E`
- 배경 텍스처: SVG fractal noise grain overlay (--grain 변수)
- 새 템플릿 추가 시 기존 디자인 토큰 복사해서 시작

## 프로젝트 구조

```
jp-tutor/
  .claude-plugin/plugin.json    ← 버전 관리
  commands/*.md                 ← 15개 명령어
  skills/jp/SKILL.md            ← 메인 스킬 (500줄 이내 유지)
  skills/jp/references/         ← 참조 문서
  skills/jp/references/templates/ ← HTML 템플릿 (4개)
  agents/*.md                   ← 3개 에이전트
.claude-plugin/marketplace.json ← 버전 관리 (루트)
README.md
```

## 커밋 규칙

- feat 커밋에 버전 + README 포함
- 한국어 커밋 메시지 사용
- `Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>` 항상 포함
