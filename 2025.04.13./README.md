# 📌 What I Learned – 채용공고 추천 자동화 프로젝트 `job-recommend-bot`

---

## 📝 프로젝트 개요

`job-recommend-bot`은 채용 사이트에서 채용공고를 수집하고,  
GPT API를 통해 사용자 이직 조건에 맞는 공고인지 판단한 후  
Slack Webhook으로 추천 공고를 전달하는 자동화 시스템이다.  

Node.js 기반으로 구현되었으며, 크롤링, GPT 연동, 슬랙 메시지 전송을  
하나의 파이프라인 흐름으로 구성했다.

---

## ✅ 전체 구성 흐름

```
1. 채용공고 크롤링
2. GPT 분석
3. 적합도 판단
4. Slack으로 알림 전송
```

---

## 🛠️ 적용 기술 및 구성 요소

| 구성 요소 | 기술 |
|-----------|------|
| 언어 | Node.js (ESM) |
| 크롤링 | Puppeteer |
| AI 분석 | OpenAI GPT API |
| 알림 | Slack Webhook |
| 실행 | CLI, crontab, PM2 |

---

## 🔍 기능별 정리 및 배운 점

### 1. 채용공고 크롤링

- Puppeteer로 headless browser 기반 동적 크롤링
- 사이트별 selector 구조 분리
- 중복 제거 및 비정상 페이지 필터링

### 2. GPT 분석

- 사용자 이력 + 공고 내용을 포함한 prompt 구성
- GPT로부터 적합도 점수(0~100점) 반환
- 일정 기준 초과 시 알림 전송 대상 선정

```js
{
  role: "user",
  content: "다음 채용공고가 이직 조건에 부합하는지 평가해줘..."
}
```

### 3. Slack 알림 전송

- Webhook URL 기반 메시지 전송
- 제목, 점수, 링크 포함

```json
{
  "text": "[88점] 백엔드 개발자 - 링크"
}
```

### 4. 실행 방식

```bash
node main.js
# 또는 crontab 등록
*/30 * * * * node /home/ubuntu/job-recommend-bot/main.js
```

---

## ⚙️ 구현 중 고려한 사항

- 크롤링 구조 유연성 확보 (모듈화)
- GPT 요청 최소화 (조건 분기 및 캐싱 예정)
- Slack 메시지 포맷 최적화
- 예외 및 로그 처리 구성

---

## 📌 향후 개선 방향

- GPT 응답 캐시 처리
- 사용자 조건 외부 설정화
- 이메일/노션 알림 추가
- GPT 토큰 사용량 최적화
- 분석 모델 다변화

---

## ✅ 정리

| 항목 | 내용 |
|------|------|
| 목표 | 조건 기반 채용공고 Slack 추천 자동화 |
| 핵심 기술 | Puppeteer, GPT API, Slack Webhook |
| 구성 방식 | 크롤링 → 분석 → 필터링 → 알림 |
| 주요 학습 | 실무형 자동화 파이프라인 설계와 AI 연동 흐름 구성 |

