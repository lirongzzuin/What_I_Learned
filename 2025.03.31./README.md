# 📌 Cordova 하이브리드 앱 - iOS 엔터프라이즈 배포 갱신 작업

## 📝 개요
회사에서 개발한 Cordova 기반 하이브리드 앱을 iOS용으로 엔터프라이즈 배포한 지 약 1년이 지난 시점에서, 특정 고객사로부터 **인증서 만료 예정 안내**를 받았다. 이에 따라 **새로운 엔터프라이즈 인증서로 앱을 다시 서명하고 재배포**하는 작업을 수행하였다.

이번 작업은 다음과 같은 절차로 진행되었다:
1. 인증서 갱신 및 iOS 배포용 `.ipa` 파일 생성
2. Cordova CLI를 통한 앱 빌드 및 Xcode 프로젝트 연동
3. 다운로드 링크 구성 및 고객사 서버 배포
4. 설치 테스트 및 피드백 대응

---

## 🔧 작업 상세

### 1. 인증서 갱신 및 iOS용 `.ipa` 파일 재배포
- Apple Developer 계정에서 **새로운 iOS 엔터프라이즈 인증서(Distribution)**를 발급
- 기존 앱의 `Provisioning Profile` 및 인증서 정보 업데이트
- **Xcode > Product > Archive**를 통해 앱 아카이빙 후, `.ipa` 파일 추출

```bash
# Cordova 프로젝트에서 iOS 플랫폼 초기화
$ cordova platform rm ios
$ cordova platform add ios

# 빌드 후 Xcode로 열기
$ cordova prepare ios
$ open platforms/ios/YourApp.xcworkspace
```

### 2. Xcode에서 `manifest.plist` 포함하여 아카이브
- Xcode에서 아카이브한 후, **Export > Enterprise > Include manifest for over-the-air installation** 옵션을 선택
- 이 과정에서 `.ipa`와 함께 배포에 필요한 `manifest.plist` 파일도 자동 생성됨
- Export 결과 디렉토리 구조 예시:
  ```
  └── ExportedApp/
      ├── App.ipa
      ├── manifest.plist
      └── index.html
  ```

### 3. 다운로드 링크 및 HTML 페이지 구성
- 배포용 서버에 위의 세 파일을 업로드
- 사용자는 iOS Safari에서 아래와 같은 링크를 통해 앱을 설치 가능

```text
itms-services://?action=download-manifest&url=https://[배포서버]/ios/builds/manifest.plist
```

- 또는 `index.html`을 통해 다운로드 안내 및 설치 버튼 제공

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>앱 다운로드</title>
</head>
<body>
  <h3>앱 다운로드</h3>
  <p>iOS에서 Safari로 접속 후 아래 버튼을 클릭하세요.</p>
  <a href="itms-services://?action=download-manifest&url=https://[배포서버]/ios/builds/manifest.plist">
    <button>앱 설치</button>
  </a>
</body>
</html>
```

- 해당 HTML은 고객사 전용 배포 페이지에서 접근 가능하도록 구성

### 4. 테스트 및 고객사 대응
- 실제 배포 서버에 업로드 후, iOS 디바이스에서 다운로드 및 설치 테스트 진행
- 인증서가 새롭게 반영되었는지 확인
- 고객사 측에 갱신 배포 사실 및 설치 경로 안내

---

## 📌 작업하면서 배운 점
- iOS의 **엔터프라이즈 배포 인증서는 1년 유효기간**을 가지므로, 사전 점검 및 갱신이 중요함
- `.plist` 파일의 구성 요소를 이해하고, 올바른 서명과 앱 URL이 배포 성공에 핵심적
- Xcode에서 아카이브 시 **Include manifest** 옵션을 통해 배포에 필요한 모든 파일을 자동으로 구성할 수 있음
- Cordova 기반 앱이라도 플랫폼 간 빌드와 배포 관리에 유연하게 대응할 수 있도록 프로젝트 구조를 잘 관리하는 것이 중요함

---

## 🎯 향후 개선 방향
- 인증서 만료 시점을 관리하는 **자동 모니터링 시스템 구축**
- 고객사별로 인증서 만료 알림 및 배포 이력 관리 시스템 도입
- MDM 솔루션을 활용한 배포 자동화 검토 및 대규모 고객 대응력 강화

---

