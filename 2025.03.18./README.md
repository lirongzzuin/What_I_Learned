# 📌 Cordova InAppBrowser를 활용한 이미지 확대 보기 기능 구현

## 📝 개요
하이브리드 앱 개발 과정에서 **조직도에서 특정 사용자의 프로필 사진을 클릭하면 해당 사진을 확대하여 볼 수 있도록** InAppBrowser 플러그인을 활용하여 기능을 구현했다. 기존에는 클릭 시 아무런 반응이 없었지만, 이제 클릭 시 새 창에서 이미지를 확대하여 볼 수 있도록 개선했다.

---

## 🚀 구현 내용

### 🔹 1. 사용자 상세 정보 조회 기능
사용자 조직도에서 특정 사용자의 정보를 조회할 수 있도록 하는 기능을 추가했다. 이 과정에서 사용자의 **프로필 사진 URL을 동적으로 생성**하여 저장하도록 했다.

📌 **주요 코드**
```javascript
$("#organization").on("click", ".view_detail", function(e) {
    e.stopPropagation();
    let user = user_map[$(this).attr("user_id")];
    let img_element = $(this).closest('li').find('img');
    let img_src = img_element.attr('src');
    
    let user_info = {
        "name": user.name,
        "photo_url": (user.photo_path_url !== undefined && user.photo_path_url !== '') 
            ? `${settingData.server_host}${settingData.server_port}${user.photo_path_url}`
            : img_src
    };

    console.log("생성된 이미지 URL:", user_info.photo_url);

    let infos = [
        {"key": "소속", "value": formatDisplayGroup(user.group_names, '@@')},
        {"key": "직책", "value": emptyToSpace(user.position)},
        {"key": "내선번호", "value": format_phone_number(default_setting.region_number + user.cid_no), buttons: (user.cid_no === "" ? [] : ["call"])},
        {"key": "휴대폰", "value": (user.cell_phone_number === "" ? '-' : format_phone_number(user.mobile_no)), buttons: (user.mobile_no === "" ? [] : ["sms", "call"])},
        {"key": "담당업무", "value": emptyToSpace(user.description)}
    ];

    openDetail(user_info, infos, null, 'organization');
});
```

---

### 🔹 2. 프로필 사진 클릭 시 InAppBrowser를 통한 확대 보기
프로필 사진 클릭 시 **InAppBrowser를 통해 확대하여 볼 수 있도록 개선**했다.

#### ✅ 주요 개선 사항
- **이미지 URL 확인 후 유효한 경우에만 InAppBrowser 실행**
- **기본 이미지(`default_image.jpg`) 클릭 시 확대 보기 기능 무시**
- **InAppBrowser에 닫기 버튼 및 뒤로 가기 버튼 추가**

📌 **주요 코드**
```javascript
$(document).on("click", ".portrait img", function(event) {
    event.stopPropagation();
    let imageUrl = $(this).attr("src");

    // 기본 이미지 클릭 방지
    if (!imageUrl || imageUrl.includes("default_image.jpg") || imageUrl === setThumbImage()) {
        console.warn("기본 이미지가 클릭되었습니다. InAppBrowser를 실행하지 않습니다.");
        return;
    }

    // URL 유효성 검사
    if (!imageUrl.startsWith("http")) {
        console.error("잘못된 이미지 URL:", imageUrl);
        alert("잘못된 이미지 URL입니다.");
        return;
    }

    // InAppBrowser 실행
    let inAppRef = cordova.InAppBrowser.open(imageUrl, '_blank', 'location=no,zoom=yes,toolbar=yes,enableViewportScale=yes');

    // 닫기 버튼 추가
    inAppRef.addEventListener('loadstop', function() {
        inAppRef.executeScript({
            code: `
                let closeButton = document.createElement('button');
                closeButton.innerText = '닫기';
                closeButton.style.position = 'fixed';
                closeButton.style.top = '10px';
                closeButton.style.right = '10px';
                closeButton.style.padding = '10px';
                closeButton.style.background = 'rgba(0, 0, 0, 0.5)';
                closeButton.style.color = 'white';
                closeButton.style.border = 'none';
                closeButton.style.borderRadius = '5px';
                closeButton.style.zIndex = '1000';
                closeButton.onclick = function() { window.close(); };
                document.body.appendChild(closeButton);
            `
        });
    });

    // 뒤로 가기 버튼 이벤트 추가 (Android 대응)
    inAppRef.addEventListener("backbutton", function() {
        inAppRef.close();
    });
});
```

---

## 📌 작업하면서 배운 점
- **웹뷰에서 동적인 콘텐츠를 표시할 때, 사용자의 경험을 고려하는 것이 중요하다.**
- **이벤트 리스너를 활용하면 UI 및 UX를 개선할 수 있다.**
  - 닫기 버튼 추가
  - 뒤로 가기 버튼 대응 (모바일 환경 고려)
  - 더블 클릭 이벤트 처리 가능
- **데이터의 유효성을 검증하는 과정이 필요하다.**
  - 기본 이미지(`default_image.jpg`)에 대한 예외 처리 필요
  - `http` 또는 `https`로 시작하는지 확인하여 보안 문제 방지

---

## 🎯 향후 개선 방향
- **더 나은 사용자 경험을 위해 인터랙티브한 이미지 뷰어 도입 검토**
- **모바일 및 웹 환경에서 일관된 UI/UX 제공을 위한 개선**
- **데이터 처리 방식 최적화 및 성능 개선**

