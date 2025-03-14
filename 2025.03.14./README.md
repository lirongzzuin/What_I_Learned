# 📌 컨텐츠 등록 및 수정 기능 개발

## 📝 개요
컨텐츠를 등록하고 수정하는 기능을 구현하면서, JS 및 SH 파일을 자동으로 생성 및 업데이트하는 로직을 추가했다. 기존에는 컨텐츠 테이블에서 `server_host`와 `server_port` 정보를 변경해도 해당 변경사항이 반영되지 않는 문제가 있었으나, 이를 해결하기 위해 해당 정보를 기반으로 SH 및 JS 파일이 함께 수정되도록 구현했다.

---

## 🚀 구현 내용

### 🔹 1. 컨텐츠 등록 (POST /content)
컨텐츠를 신규 등록할 때, 중복 검사를 수행하고 SH 및 JS 파일을 함께 생성하도록 했다.

#### ✅ 주요 처리 과정
1. **중복 검사**: 같은 이름의 컨텐츠가 존재하는지 확인
2. **JSON 데이터 생성**: `server_host`, `server_port` 등 필수 정보를 JSON 객체로 저장
3. **컨텐츠 저장**: DB에 저장 후 UID를 부여
4. **버전 데이터 생성**: Android 및 iOS용 버전 데이터 추가
5. **SH 및 JS 파일 생성**: 
   - `createOrUpdateJsFile()` : JS 파일 생성
   - `createShFile()` : SH 파일 생성

#### ✅ 컨텐츠 등록 API 코드
```java
@PostMapping("content")
public ResponseEntity<GResponse> createContents(HttpServletResponse response, @RequestBody ContentParam param) {
    response.setHeader("Job-Log", "컨텐츠 정보 등록");

    // 중복 검사
    ContentEntity content = contentRepo.findByName(param.getName());
    if (content != null) {
        return ResponseEntity.ok(new GResponse("400", "이미 존재하는 컨텐츠명"));
    }

    // JSON 데이터 생성
    JSONObject info = new JSONObject();
    info.put("server_host", param.getServerHost());
    info.put("server_port", param.getServerPort().isEmpty() ? ":443" : param.getServerPort());
    
    JSONObject define = new JSONObject();
    define.put("server_host", param.getServerHost());
    define.put("server_port", param.getServerPort().isEmpty() ? ":443" : param.getServerPort());
    
    // 컨텐츠 저장
    content = new ContentEntity();
    content.setCreator(userHelper.getCurrentUserID());
    content.setInfo(info.toString());
    content.setDefine(define.toString());
    content = contentRepo.save(content);

    // SH 및 JS 파일 생성
    fileService.createOrUpdateJsFile(param.getName(), param.getProductName(), define);
    fileService.createShFile(param.getName(), param.getProductName(), param.getServerHost(), param.getServerPort(), content);

    return ResponseEntity.ok(new GResponse("0000", "등록 완료"));
}
```

---

### 🔹 2. 컨텐츠 수정 (PATCH /content)
컨텐츠를 수정할 때, 기존 `server_host`와 `server_port` 정보를 업데이트하며, 해당 변경사항이 JS 및 SH 파일에도 반영되도록 했다.

#### ✅ 주요 처리 과정
1. **컨텐츠 조회**: 기존 컨텐츠가 존재하는지 확인
2. **버전 정보 업데이트**: 이름이 변경되었을 경우 관련 버전 데이터도 수정
3. **JSON 데이터 업데이트**: `server_host`, `server_port` 값 변경 반영
4. **SH 및 JS 파일 업데이트**

#### ✅ 컨텐츠 수정 API 코드
```java
@PatchMapping("content")
public ResponseEntity<GResponse> updateContent(HttpServletResponse response, @RequestBody ContentParam param) {
    response.setHeader("Job-Log", "컨텐츠 정보 수정");
    
    // 컨텐츠 조회
    ContentEntity content = contentRepo.findByName(param.getName());
    if (content == null) {
        return ResponseEntity.ok(new GResponse("404", "존재하지 않는 컨텐츠"));
    }

    // 기존 버전 정보 업데이트
    List<VersionEntity> versions = versionRepo.findByName(param.getOrgName());
    if (versions != null) {
        versions.forEach(version -> {
            version.setName(param.getName());
            versionRepo.save(version);
        });
    }

    // JSON 데이터 업데이트
    JSONObject info = new JSONObject(content.getInfo());
    info.put("server_host", param.getServerHost());
    info.put("server_port", param.getServerPort().isEmpty() ? ":443" : param.getServerPort());
    content.setInfo(info.toString());

    JSONObject define = new JSONObject(content.getDefine());
    define.put("server_host", param.getServerHost());
    define.put("server_port", param.getServerPort().isEmpty() ? ":443" : param.getServerPort());
    content.setDefine(define.toString());

    // SH 및 JS 파일 업데이트
    fileService.createOrUpdateJsFile(param.getName(), param.getProductName(), define);
    fileService.createShFile(param.getName(), param.getProductName(), param.getServerHost(), param.getServerPort(), content);
    
    return ResponseEntity.ok(new GResponse("0000", "수정 완료"));
}
```

---

## 📌 작업하면서 배운 점
- **데이터 일관성을 유지하는 것이 중요하다.**
  - 컨텐츠의 핵심 정보가 변경될 경우, 관련된 모든 데이터(JS, SH 파일 포함)도 함께 업데이트되어야 한다.
  
- **파일 생성 및 업데이트 로직은 트랜잭션과 함께 처리해야 한다.**
  - 컨텐츠 저장과 파일 생성이 하나의 트랜잭션 내에서 처리되지 않으면 데이터 불일치 문제가 발생할 수 있다.
  
- **SH 및 JS 파일 생성 시, 기본값을 명확히 지정하는 것이 중요하다.**
  - `server_port` 값이 없을 때 기본값(`:443`)을 적용하는 방식으로 데이터 일관성을 유지할 수 있다.
  
- **에러 핸들링을 철저히 해야 한다.**
  - 파일 생성 시 `IOException` 등의 예외 처리를 정확히 하지 않으면 배포 실패와 같은 문제가 발생할 수 있다.

---

## 🎯 향후 개선 방향
- **SH 및 JS 파일 변경 이력을 관리하여 버전별 롤백 기능 추가**
- **파일 생성 시 비효율적인 I/O 작업을 최소화하도록 리팩토링**

