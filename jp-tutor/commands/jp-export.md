---
description: "학습 데이터 내보내기. 어휘/문법/한자 목록을 CSV, Anki 덱 형식으로 내보냅니다."
allowed-tools: ["Read", "Write", "Edit", "Bash"]
model: haiku
argument-hint: "[형식: csv|anki] [카테고리: kana|vocab|grammar|kanji|business|textbook] [범위]"
---

# 학습 데이터 내보내기

## 사전 준비

1. `jp-data/progress.json`을 읽으세요.
   - 파일이 없으면: "아직 학습 데이터가 없습니다. `/jp:start`로 시작해주세요!" 안내 후 종료.

## 내보내기 형식 결정

사용자 입력에서 형식과 카테고리를 파악합니다:
- $ARGUMENTS에 "anki"가 있으면 Anki 형식, 아니면 CSV 기본
- 카테고리 미지정 시 전체 내보내기

## CSV 내보내기

`jp-data/exports/` 디렉토리가 없으면 생성합니다.

### 가나 (kana)
파일: `jp-data/exports/kana_{date}.csv`
헤더: 문자,유형(히라가나/카타카나),발음,배지,정답수,시도수,정답률,마지막복습
kana.concepts에서 `_hira`/`_kata` 접미사로 유형을 구분합니다.

### 어휘 (vocab)
파일: `jp-data/exports/vocab_{level}_{date}.csv`
헤더: 단어,읽기,뜻,배지,정답수,시도수,정답률,마지막복습
각 concepts의 데이터를 행으로 변환합니다.

### 문법 (grammar)
파일: `jp-data/exports/grammar_{level}_{date}.csv`
헤더: 문법포인트,배지,정답수,시도수,정답률,마지막복습,오류노트

### 한자 (kanji)
파일: `jp-data/exports/kanji_{level}_{date}.csv`
헤더: 한자,배지,정답수,시도수,정답률,마지막복습

### 비즈니스 (business)
파일: `jp-data/exports/business_{date}.csv`
헤더: 모듈,용어,배지,정답수,시도수

### 교재 (textbook)
교재별 파일: `jp-data/exports/textbook_{name}_{date}.csv`
헤더: 단원,개념,배지,정답수,시도수

## Anki 내보내기

탭 구분 텍스트 파일을 생성합니다.
파일: `jp-data/exports/{category}_anki_{date}.txt`

### 어휘
front: 일본어 단어
back: 읽기; 뜻

### 한자
front: 한자
back: 음독; 훈독; 뜻; 관련 단어

### 문법
front: 문법 패턴
back: 의미; 예문

## 완료 메시지

```
내보내기 완료!
- 형식: {CSV/Anki}
- 카테고리: {카테고리}
- 항목 수: {count}개
- 저장 위치: {파일 경로}

Anki의 경우: Anki → 파일 → 가져오기 → 텍스트 파일 선택
```
