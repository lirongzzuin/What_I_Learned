# 🧠 AI 과실 분석기 blame-detector

---

## 📌 프로젝트 개요

논쟁 중인 카카오톡 대화를 업로드하면 AI가 감정, 공격성, 발화 흐름 등을 분석하여 과실 비율을 산정해주는 웹 서비스.

> 이미지 업로드 → OCR → 감정/공격성 분석 → 과실 계산 → 결과 리포트 시각화 → 공유 기능 제공

모바일 UX에 최적화되었으며, 로그인 없이 누구나 즉시 사용할 수 있도록 구성함.  
카카오톡 공유 기능 중심의 UX와 공유 기반 확산 구조를 설계함.

---

## 📂 프로젝트 구조 및 기술 스택

```
blame-detector/
├── backend/     # Spring Boot 기반 API 서버
├── frontend/    # React + Vite + Tailwind CSS 기반 SPA
├── .gitignore
├── README.md
```

| 영역 | 기술 |
|------|------|
| 프론트엔드 | React, Vite, Tailwind CSS, Chart.js, html2canvas |
| 백엔드 | Spring Boot, Tesseract4J, WebClient |
| AI 분석 | Hugging Face API (감정 분석, 공격성 분류) |
| 공유 기능 | Kakao SDK, Web Share API, 링크 복사 |
| 배포 환경 | Vercel (FE), Render 또는 EC2 (BE 예정) |

---

## ✅ 주요 기능별 구현 및 학습 내용

### 1. 이미지 업로드 및 미리보기

```jsx
const [image, setImage] = useState(null);
const handleImageChange = (e) => {
  const file = e.target.files?.[0];
  if (file) {
    setImage(file);
    setPreview(URL.createObjectURL(file));
  }
};
```

- React의 `useState`로 업로드된 이미지 상태 관리
- `URL.createObjectURL()`로 사용자에게 즉시 미리보기 제공
- `FormData`로 백엔드에 이미지 전송

---

### 2. OCR 처리 (Spring Boot + Tesseract4J)

```java
Tesseract tesseract = new Tesseract();
tesseract.setDatapath("/usr/local/share/tessdata");
tesseract.setLanguage("kor+eng");
String text = tesseract.doOCR(imageFile);
```

- `tess4j` 라이브러리를 통해 이미지에서 텍스트 추출
- Mac 환경에서는 Tesseract CLI가 설치된 경로 설정 필요
- 줄 단위 텍스트 분리 후 메시지 분석용 리스트 생성

---

### 3. 감정/공격성 분석 (Hugging Face API)

```java
String apiUrl = "https://api-inference.huggingface.co/models/cardiffnlp/twitter-roberta-base-sentiment";
WebClient client = WebClient.create();
String response = client.post()
    .uri(apiUrl)
    .header("Authorization", "Bearer " + huggingfaceApiKey)
    .bodyValue(Map.of("inputs", text))
    .retrieve()
    .bodyToMono(String.class)
    .block();
```

- Hugging Face에서 제공하는 모델 2종 사용
  - 감정 분류: `positive`, `neutral`, `negative`
  - 공격성 여부: `toxic`, `non-toxic`
- WebClient로 비동기 API 호출

---

### 4. 과실 비율 계산

```java
int toxicA = ..., toxicB = ...;
int total = toxicA + toxicB;
int ratioA = (int) ((toxicA / (double) total) * 100);
int ratioB = 100 - ratioA;
```

- 분석된 메시지를 사용자 A/B로 구분
- toxic 비율이 높은 쪽에 더 많은 과실 배정
- 단순 비율 기반 초기 로직 구현 (향후 ML 기반 확장 가능성 고려)

---

### 5. 결과 리포트 시각화 (Chart.js + React)

```jsx
<Doughnut data={{
  labels: ["A", "B"],
  datasets: [{
    data: [ratioA, ratioB],
    backgroundColor: ["#60a5fa", "#f87171"],
  }]
}} />
```

- A/B 과실 비율을 도넛 차트로 시각화
- 감정, 공격성 분석 결과도 리스트로 시각적으로 구분
- TailwindCSS로 카드형 리스트 구성

---

### 6. 공유 기능

```js
// 링크 복사
await navigator.clipboard.writeText("https://blame-detector.vercel.app");

// 이미지 저장
const canvas = await html2canvas(resultRef.current);
link.href = canvas.toDataURL("image/png");
```

- ✅ **카카오톡 공유 (JS SDK)**: 메시지/버튼/링크 포함
- ✅ **링크 복사 기능**
- ✅ **html2canvas로 결과 화면 이미지 저장 + 워터마크 삽입**

---

### 7. 다시하기 기능

```jsx
const [result, setResult] = useState(null);

return result ? (
  <Result result={result} onReset={() => setResult(null)} />
) : (
  <ImageUpload onResult={setResult} />
);
```

- 분석 결과 → 다시 업로드 화면으로 돌아갈 수 있도록 상태 기반 전환

---

### 8. GitHub Push Protection 대응

- Hugging Face API 키가 `application.properties`와 `.env`에 커밋되어 GitHub 푸시 차단됨
- `BFG Repo-Cleaner`를 사용해 민감 정보 포함된 커밋 히스토리 완전 삭제

```bash
bfg --delete-files .env
git push origin main --force
```

- 이후 `.env`로 모든 민감 정보 분리 + `.gitignore` 설정

---

### 9. 배포 준비

- 프론트: Vercel에 `frontend/` 기준으로 배포 준비 완료
  - 환경 변수는 VITE_ 접두어로 설정
  - 예: `VITE_KAKAO_JS_KEY`, `VITE_HUGGINGFACE_API_KEY`
- Hugging Face API 키는 배포 주소 확보 후 등록 예정

---

## 📌 마무리 요약

- OCR → AI 분석 → 시각화 → 공유 → 재진입까지 실제 사용자 흐름을 따라 전체 구현
- 보안, 배포, 환경 분리 등 실무 수준의 구성 학습
- AI 기반 기능을 UX 중심으로 구현해보며 실용적인 웹 서비스 구조에 대한 감각을 익힘