# 📌 OTP 인증 기능 구현 및 처리 흐름 정리

## 📝 개요
회사별 인증 방식에 따라 OTP(One-Time Password) 인증 기능을 구현하고 개선하는 작업을 진행했다. 본 기능은 사내 하이브리드 앱 로그인 시 사용자 인증을 보완하기 위한 보안 수단으로 도입되었으며, 각 회사의 정책에 따라 SMS, FCM(Google Push) 방식으로 전송이 가능해야 했다. OTP 발급부터 인증 검증, 디바이스 등록까지의 전 과정을 설정 기반으로 유연하게 처리하도록 구성했다.

---

## ⚙️ 전체 구성 흐름

### ✅ 1. OTP 요청 흐름
1. 사용자가 로그인 요청 (이메일 + 전화번호 입력)
2. 사용자 정보 확인 → 해당 회사의 OTP 사용 여부 및 방식 확인
3. OTP 생성 → 설정된 방식으로 OTP 전송 (SMS / FCM)
4. 전송 성공 시 DB에 OTP 저장

### ✅ 2. OTP 인증 처리 흐름
1. 사용자 입력 OTP 수신
2. DB에 저장된 OTP와 비교
3. 일치 시 OTP 삭제, 인증 성공 처리
4. JWT 토큰 발급 및 디바이스 등록

### ✅ 3. OTP 요청 API 처리 (`requestOtp()`)
- 사용자의 전화번호로 OTP 생성 및 전송 요청
- 회사별 설정에 따라 방식 분기 처리 (SMS / GOOGLE_PUSH 등)

```java
int otp = OtpProvider.generateOTP(mobile_number, otpDuration, now.toString());
mdotp.setMobileNo(mobile_number);
mdotp.setOtp(String.valueOf(otp));
```

---

## 🔐 OTP 인증 처리

### ✅ OTP 생성 및 저장
- OTP는 전화번호 + 시간 기반으로 생성됨
- OTP는 `MdEntityOtp` 엔티티에 저장되며, 만료 시간 정보 포함

### ✅ OTP 인증 검증 (`verifyOtp()`)
- DB에서 해당 전화번호로 발급된 최신 OTP 조회
- 입력값과 비교 → 일치 시 삭제, 인증 성공
- 시간 차이(최대 10초) 허용 처리 로직 포함

```java
if (db_otp.getOtp().equals(p_otp.getOtp())) {
    otpRepository.delete(db_otp);
    return 인증 성공;
}
```

---

## ✉️ 전송 방식별 처리

### ✅ 1. SMS 방식 (KT / LG 등)
- 회사 설정에 따라 분기
- MyBatis 매퍼를 통해 DB 전송 테이블에 저장하여 전송

```java
ktmap.put("sms_msg", String.format(otp_message_format, otp));
mybatisMapper.saveKtSms(ktmap);
```

### ✅ 2. FCM Push 방식
- FirebaseApp 인스턴스를 통해 푸시 전송
- `extra` 필드에 FCM token 저장 후 전송 처리

```java
Message message = Message.builder()
  .setToken(token)
  .setNotification(...)
  .putAllData(data)
  .build();
```

---

## ⚙️ 전체 설정 구조 (application.yml 예시)
```yaml
g23:
  mdialer-service:
    auth:
      otp:
        sendtype: GOOGLE_PUSH
        duration: 60
        firebase:
          credential-path: /path/to/credential.json
        msg_format: "인증번호는 [%s] 입니다."
      sms:
        user_id: "testuser"
        cb_number: "029302930"
        mm_telecom: "KT"
      kakao:
        service_no: "kakao123"
        server_url: "https://kakao.example.com"
        auth_token: "xxxxx"
```

---

## 📂 디바이스 등록 처리
- 인증 성공 시 디바이스 정보를 등록하거나 업데이트
- 중복 로그인 제한, 다중 디바이스 허용 여부 확인

```java
MdEntityDevice device = deviceRepo.findFirstByUserIdAndCorpCodeAndDeviceId(...);
if (device == null) {
    device = new MdEntityDevice();
}
device.setLastLogin(new Date());
deviceRepo.save(device);
```

---

## 🧩 배운 점 정리
- OTP 발급, 전송, 검증, 디바이스 등록까지의 흐름을 설정 기반으로 분기 처리함으로써 **유연한 구조를 설계할 수 있었다**.
- OTP 검증 시 **시간 차를 고려한 유효성 검증 로직**이 사용자 경험 개선에 도움이 됨.
- 회사별 OTP 전송 방식(Firebase / SMS) 구현 시, **공통 인터페이스 설계와 설정 기반 분기처리**가 중요함.
- JWT 기반 로그인 토큰 발급과 OTP 인증을 통합하여 **보안성과 사용자 편의성을 동시에 확보**함.

---

## 🎯 향후 개선 방향
- OTP 요청 횟수 제한 및 도전 차단 기능 구현 (Rate Limit)
- OTP 인증 실패 로그 및 통계 수집 기능 추가
- 메시지 포맷 커스터마이징 기능 지원 (회사별 문구 설정)
- WebSocket 기반 인증 처리로 사용자 피드백 실시간 반영 고려

