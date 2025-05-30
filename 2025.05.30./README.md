# OTP 인증 및 로그인 기능 개발 기록

## 개발 목표

Cordova 하이브리드 앱 기반 서비스에 OTP(One-Time Password) 인증 단계를 추가하여 보안성을 강화하고, 다양한 인증 수단(SMS, Google Push)을 선택적으로 사용할 수 있도록 구현하는 것을 목표로 함.

## 개발 흐름

1. **앱 실행**: FCM(Firebase Cloud Messaging) 토큰을 앱 실행 직후 획득하여 `localStorage`에 저장.
2. **로그인 화면 초기화**: 회사 설정값에 따라 OTP 사용 여부(`otp_used`) 및 OTP 타입(`otp_type`)을 확인하여 OTP 입력 필드를 동적으로 표시.
3. **OTP 요청**: 사용자가 아이디 입력 후 OTP 요청 버튼을 클릭하면, 등록된 휴대전화 번호로 OTP를 전송.
4. **OTP 입력**: 수신한 OTP를 앱 화면에 입력.
5. **OTP 검증 및 로그인 요청**: 입력한 OTP와 함께 로그인 요청을 전송하고, 성공 시 앱 메인 화면으로 이동.

## 주요 고려 사항

* **FCM 토큰 사전 확보**: 앱 구동 시 FCM 토큰을 미리 확보해야 OTP 요청이 가능.
* **OTP 사용 여부 동적 처리**: 회사별 설정에 따라 OTP 사용 여부와 타입을 확인하여 동적으로 화면 구성.
* **에러 핸들링**: 누락된 입력값, OTP 인증 실패, 만료된 OTP 등 다양한 예외 케이스에 대해 사용자에게 명확한 메시지를 제공.
* **하이브리드 앱 특성 고려**: Cordova 플러그인을 사용하여 디바이스 정보 및 FCM 토큰을 수집하고 네이티브 기능과 연동.

## 사용된 주요 API 구조

### 1. OTP 요청 API (`/otp/request`)

```json
{
  "user_id": "user123",
  "mobile_no": "01012345678",
  "firebase_token": "firebase-token-example"
}
```

### 2. OTP 인증 및 로그인 API (`/otp/verifylogin`)

```json
{
  "mobile_no": "01012345678",
  "otp": "123456",
  "device_id": "device-uuid",
  "company_name": "SampleCorp",
  "company_password": "secure-pass",
  "cid_number": "CID001",
  "user_id": "user123",
  "firebase_token": "firebase-token-example",
  "platform": "Android",
  "device_model": "Pixel 7",
  "os_version": "14",
  "app_version": "1.0.0"
}
```

## Cordova 하이브리드 앱 연동 방식

앱 초기화 시 FCM 토큰 확보, 디바이스 정보 수집, 설정값에 따라 OTP 사용 여부 확인 및 로그인 프로세스를 제어.

## 주요 구현 코드

### 1. FCM 토큰 획득 및 저장

```javascript
FirebasePlugin.getToken(function(fcmToken) {
  localStorage.setItem("fcm_token", fcmToken);
}, function(error) {
  console.error("FCM 토큰 획득 실패", error);
});
```

### 2. OTP 요청 함수

```javascript
function requestOtp() {
  const userId = document.querySelector(".input_id").value;
  const mobileNumber = settingData.mobile_no;
  const firebaseToken = localStorage.getItem("fcm_token");

  if (!firebaseToken) {
    alert("푸시 토큰이 없습니다. 앱을 재시작해주세요.");
    return;
  }

  const payload = {
    user_id: userId,
    mobile_no: mobileNumber,
    firebase_token: firebaseToken
  };

  cordova.plugin.http.post(
    `${server_host}/otp/request`,
    payload,
    {},
    function(response) {
      const res = JSON.parse(response.data);
      if (res.code === "0000") {
        alert("OTP가 전송되었습니다.");
      } else {
        alert("OTP 요청 실패: " + res.message);
      }
    },
    function(error) {
      alert("OTP 요청 중 오류 발생");
    }
  );
}
```

### 3. OTP 인증 및 로그인 함수

```javascript
function verifyOtpAndLogin() {
  const userId = document.querySelector(".input_id").value;
  const password = document.querySelector(".input_password").value;
  const otp = document.querySelector(".input_otp").value;
  const mobileNumber = settingData.mobile_no;
  const firebaseToken = localStorage.getItem("fcm_token");

  const payload = {
    mobile_no: mobileNumber,
    otp: otp,
    device_id: device.uuid,
    company_name: settingData.company_id,
    company_password: settingData.company_pw,
    cid_number: settingData.cid_no,
    user_id: userId,
    firebase_token: firebaseToken,
    platform: device.platform,
    device_model: device.model,
    os_version: device.version,
    app_version: app_ver_string
  };

  cordova.plugin.http.post(
    `${server_host}/otp/verifylogin`,
    payload,
    {},
    function(response) {
      const res = JSON.parse(response.data);
      if (res.code === "0000") {
        alert("로그인 성공");
        window.location.href = "main.html";
      } else {
        alert("로그인 실패: " + res.message);
      }
    },
    function(error) {
      alert("로그인 요청 중 오류 발생");
    }
  );
}
```

## 기능적 특징

* **OTP 사용 여부 동적 설정**: 서버로부터 `otp_used` 값을 받아와 로그인 화면에 OTP 입력란을 표시하거나 숨김.
* **OTP 타입 구분**: `otp_type` 값(`SMS`, `GOOGLE_PUSH`)에 따라 OTP 전송 방식을 동적으로 변경.
* **자동 로그인 지원**: 사용자가 자동 로그인을 선택하면 다음 앱 실행 시 아이디/비밀번호를 자동 입력하여 로그인 시도.
* **에러 케이스 대응**: OTP 인증 실패, 토큰 없음, 서버 오류 등 다양한 상황에 대비하여 상세한 사용자 피드백 제공.

## 개발하면서 배운 점

* **FCM 토큰 관리의 중요성**: 앱 구동 초기에 FCM 토큰을 확보하고 관리하는 패턴을 익힘.
* **OTP 타입 유연성 구현**: SMS 또는 Google Push 중 하나를 선택할 수 있도록 설계하여 다양한 인증 환경에 대응 가능.
* **코드 구조 개선**: 요청 실패 시 적절한 예외 처리와 사용자 메시지 출력 패턴을 체계화함.
* **Cordova 하이브리드 앱 특성 이해**: 디바이스 UUID, OS 버전 등 네이티브 정보를 활용해 서버에 정확한 인증 정보를 전달.

## 향후 개선할 점

* FCM 토큰 갱신 로직 추가 필요 (토큰 만료 또는 교체 시 대응)
* OTP 유효시간 초과 시 재요청 UX 개선
* 로그인 실패 횟수 초과 시 계정 잠금 기능 추가
* 민감 정보 외부 노출 방지를 위한 보안 강화 (디바이스 정보, 서버 응답 암호화 등)
