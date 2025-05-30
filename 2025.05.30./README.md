# OTP 인증 및 로그인 기능 개발 기록

## 개발 목표

로그인 과정에 OTP(One-Time Password) 인증 단계를 추가하여 보안성을 강화하는 것을 목표로 함. 사용자에게 OTP를 발송하고, 이를 검증한 뒤 정상적으로 로그인 처리하는 플로우를 구현.

## 개발 흐름

1. **사용자 입력**: 사용자가 아이디를 입력하고 "OTP 요청" 버튼을 클릭.
2. **FCM 토큰 확인**: OTP 발송 시 FCM(Firebase Cloud Messaging) 토큰이 필요함. 앱 실행 시점에 토큰을 미리 확보해 `localStorage`에 저장.
3. **OTP 발송**: 입력된 아이디와 등록된 전화번호를 기반으로 OTP를 전송.
4. **OTP 입력**: 사용자가 수신한 OTP를 입력.
5. **OTP 검증 및 로그인 요청**: 입력한 OTP를 서버로 전송하여 검증하고, 검증이 성공하면 로그인 절차를 진행.

## 주요 고려 사항

* **FCM 토큰 사전 확보**: OTP 요청 이전에 FCM 토큰이 존재해야 하므로 앱 구동 직후 FCM 토큰을 가져와 `localStorage`에 저장.
* **FCM 토큰 중복 저장 방지**: 이미 저장된 토큰이 있을 경우 재요청하지 않고 그대로 사용.
* **에러 처리**: 필수 입력값 누락, OTP 인증 실패, 만료된 OTP 등 다양한 에러 케이스를 고려하여 사용자에게 명확한 피드백 제공.

## 사용된 주요 API 구조

### 1. OTP 요청 API (`/api/v1/otp/request`)

사용자가 아이디 입력 후 OTP 요청 시 호출.

```json
{
  "user_id": "user123",
  "mobile_no": "01012345678",
  "firebase_token": "firebase-token-example"
}
```

### 2. OTP 인증 및 로그인 API (`/api/v1/otp/verifylogin`)

사용자가 OTP를 입력하고 로그인 버튼을 눌렀을 때 호출.

```json
{
  "mobile_no": "01012345678",
  "otp": "123456",
  "device_id": "device-uuid-example",
  "company_name": "SampleCompany",
  "company_password": "company-pass",
  "cid_number": "CID000001",
  "user_id": "user123",
  "firebase_token": "firebase-token-example",
  "platform": "Android",
  "device_model": "Pixel 7",
  "os_version": "14",
  "app_version": "1.0.0"
}
```

## 구현한 예시 코드

### FCM 토큰 획득 및 저장

```javascript
FirebasePlugin.getToken(function(fcmToken) {
  if (!localStorage.getItem("fcm_token")) {
    localStorage.setItem("fcm_token", fcmToken);
  }
}, function(error) {
  console.error("FCM 토큰 획득 실패", error);
});
```

### OTP 요청

```javascript
function requestOtp() {
  const userId = document.querySelector(".input_id").value;
  const mobileNumber = "01012345678"; // 등록된 번호 예시
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
    "https://example.com/api/v1/otp/request",
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

### OTP 인증 및 로그인

```javascript
function verifyOtpAndLogin() {
  const userId = document.querySelector(".input_id").value;
  const password = document.querySelector(".input_password").value;
  const otp = document.querySelector(".input_otp").value;
  const mobileNumber = "01012345678";
  const firebaseToken = localStorage.getItem("fcm_token");

  const payload = {
    mobile_no: mobileNumber,
    otp: otp,
    device_id: device.uuid,
    company_name: "SampleCompany",
    company_password: "company-pass",
    cid_number: "CID000001",
    user_id: userId,
    firebase_token: firebaseToken,
    platform: device.platform,
    device_model: device.model,
    os_version: device.version,
    app_version: "1.0.0"
  };

  cordova.plugin.http.post(
    "https://example.com/api/v1/otp/verifylogin",
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

## 개발하면서 배운 점

* **FCM 토큰 관리**: OTP 요청 전에 토큰 확보가 필요하므로 앱 시작 시점에 토큰을 확보하고 저장하는 패턴을 확립.
* **에러 핸들링의 중요성**: 사용자 경험을 고려하여 다양한 에러 케이스에 대해 명확한 피드백을 제공.
* **보안 고려**: 인증 요청 시 필수값(아이디, 전화번호, OTP, FCM 토큰 등)을 반드시 검증.
* **코드 유지보수성**: 민감한 값(회사명, 전화번호, 서버 URL 등)을 하드코딩하지 않고 환경별로 구분하는 방법을 고민.

## 향후 개선할 점

* FCM 토큰 갱신 로직 추가 필요 (토큰 만료 또는 갱신 시 대응)
* OTP 유효시간이 초과될 경우 재요청 UX 개선
* 로그인 실패 시 재시도 제한 및 Lockout 정책 도입
