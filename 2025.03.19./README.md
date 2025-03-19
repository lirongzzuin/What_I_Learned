# 📌 Cordova 하이브리드 앱 공지사항 기능 리팩터링

## 📝 개요
공지사항 기능을 구현하는 과정에서 **변수 선언 위치와 전역/지역 변수의 사용 방식으로 인해 문제**가 발생했다. 
이 문제를 해결하기 위해 코드 리팩터링을 진행하였으며, 불필요한 전역 변수 사용을 최소화하고, **보다 유지보수하기 쉬운 구조로 개선**하였다.

---

## 🚀 구현 내용

### 🔹 1. 문제점
- 공지사항을 저장하는 **noticeUids, readNoticeList, selfDeletedNoticeList** 변수가 **전역으로 선언**되어 있어 예상치 못한 동작이 발생.
- **변수의 값이 동적으로 변경**될 때, 여러 함수에서 접근하면서 **데이터 불일치 문제**가 발생.
- 특정 함수 실행 후 **데이터가 제대로 갱신되지 않거나, UI가 반영되지 않는 문제**가 나타남.

### 🔹 2. 해결 방법
- **필요한 경우에만 전역 변수 사용**하고, 함수 내부에서만 필요한 변수는 지역 변수로 한정하여 사용.
- 데이터를 불러온 후 **동기적으로 UI 업데이트를 보장하는 구조로 변경**.
- `getNotices()` 함수 내부에서 불필요하게 중복 호출되던 로직을 제거하고, **콜백 방식으로 데이터 흐름을 명확히 개선**.

📌 **리팩터링 후 코드 예시 (공지사항 데이터 불러오기)**
```javascript
function getNotices(categoryFilter, callback) {
    let alarmValue = $("#alarmSelect").val();
    let alarmFilter = alarmValue === "Y" ? "Y" : alarmValue === "N" ? "N" : "";
    let apiUrl = server_host + server_port + "/api/notices";

    categoryFilter = categoryFilter || "all";

    cordova.plugin.http.get(apiUrl, {}, {
        Authorization: 'Bearer ' + userData.token
    }, function(response) {
        let jsonData = JSON.parse(response.data);
        let notices = jsonData.data;
        
        notices.sort((a, b) => b.uid - a.uid);
        let filteredNotices = filterNoticesByAlarm(notices, alarmFilter);

        updateNoticeUI(filteredNotices, categoryFilter);
        markReadNotices(filteredNotices);
        localStorage.setItem('readNoticeList', JSON.stringify(readNoticeList));

        if (typeof callback === "function") {
            callback();
        }
    }, function(error) {
        console.error("공지사항 불러오기 실패:", error);
    });
}
```

📌 **리팩터링 후 코드 예시 (UI 업데이트 함수 분리)**
```javascript
function updateNoticeUI(notices, categoryFilter) {
    let noticeList = $("#noticeList");
    noticeList.empty();
    
    notices.forEach(function(notice) {
        if (selfDeletedNoticeList.includes(notice.uid)) return;
        if (categoryFilter !== "all" && notice.category !== categoryFilter) return;
        
        let formattedDate = notice.create_time ? formatDate(new Date(notice.create_time)) : "-";
        let isNewClass = readNoticeList.includes(notice.uid) ? '' : '<span class="new">N</span>';

        let noticeItem = `
            <li class="notice-item" data-uid="${notice.uid}">
                <div>
                    <h4 class="btn_notice_view" id="${notice.uid}">${isNewClass} ${notice.title}</h4>
                    <p>${formattedDate} / ${notice.creator_name || "알 수 없음"}</p>
                </div>
            </li>`;
        
        noticeList.append(noticeItem);
    });
}
```

---

## 📌 작업하면서 배운 점
- **전역 변수 사용을 최소화하면 유지보수성이 향상된다.**
  - 데이터가 여러 함수에서 변경될 경우, 예상치 못한 문제를 방지할 수 있음.
  
- **UI 업데이트 로직을 별도의 함수로 분리하면 코드 가독성이 좋아진다.**
  - 데이터를 가져오는 함수와 UI를 업데이트하는 함수가 분리되어, 변경 사항을 쉽게 반영할 수 있음.
  
- **비동기 호출을 효과적으로 관리하는 것이 중요하다.**
  - 데이터 요청 후 UI 업데이트를 보장하는 구조를 유지해야 사용자 경험이 향상됨.

---

## 🎯 향후 개선 방향
- **데이터 흐름을 더욱 명확히 하기 위해 상태 관리 적용 검토**
- **로컬 스토리지의 데이터 동기화 로직 개선하여 성능 최적화**
- **UI 반응 속도 향상을 위한 비동기 처리 개선**

