# 📌 온나라 시스템 인사 DB 연동 작업 기록

## 📝 개요
고객사 중 한 곳이 내부적으로 사용하는 **온나라 시스템**의 인사 데이터를 우리 시스템과 연동해달라는 요청을 받았다. 이 고객사는 **CUBRID 데이터베이스**를 사용하고 있었고, 우리는 해당 DB에 직접 접근하여 부서 및 사용자 정보를 가져오는 기능을 구현해야 했다. 기존 고객사들은 대부분 MySQL, MSSQL 등의 DB를 사용했기 때문에, CUBRID 기반 연동은 처음이었고 새로운 경험이었다.

이번 작업은 **사내에서 사용하는 하이브리드 앱의 조직도와 사용자 정보 관리 기능**과도 연결되어 있었기 때문에, 데이터 구조 파악과 쿼리 작성, 연동 흐름을 모두 고려해야 했다.

---

## ⚙️ 연동 구조 및 흐름 설명

### 1. XML 기반 설정 구성
고객사의 DB 접속 정보 및 쿼리문은 회사별 XML 설정 파일에 정의되어 있고, 여기에 CUBRID에 맞춘 정보들을 작성했다. 

```xml
<hrissync-database>
  <username>onnara_view</username>
  <password>onnara_view</password>
  <conn-url><![CDATA[jdbc:cubrid:xxx.xxx.xxx.xxx:33000:onnara_db]]></conn-url>
  <driver>cubrid.jdbc.driver.CUBRIDDriver</driver>
  <default-password>1234</default-password>
```

부서(`select-dept`)와 사용자(`select-user`) 정보를 추출하기 위한 쿼리도 아래와 같이 함께 정의되어 있다.

```xml
<select-dept><![CDATA[
  SELECT ORGID AS CODE, ... FROM COM_ORGANIZATIONINFO ...
]]></select-dept>

<select-user><![CDATA[
  SELECT USERID AS USER_ID, ... FROM COM_USERINFO_DETAIL ...
]]></select-user>
```

또한, 숫자만 남기도록 전화번호 정제 규칙도 `<convert>` 태그로 정의했다.

---

### 2. 서버 측 연동 흐름 설명

연동은 다음과 같은 흐름으로 진행된다:

#### 1) `/schedule/run` API를 호출해 수동 실행 요청을 보냄
```java
@PostMapping("schedule/run")
public ResponseEntity<GResponse> scheduleUpdate(HttpServletResponse response) {
    response.setHeader("X-Job-Log", "인사정보 스케줄 직접 실행");
    scheduleService.runSchedule();
    return ResponseEntity.ok(new GResponse("0000", ""));
}
```

#### 2) `runSchedule()` 메서드가 실행되면서 설정 파일 로드 및 고객사별 반복 처리 시작

#### 3) 설정파일(XML)을 파싱하고 DB 접속 정보와 쿼리를 로드함
```java
Document document = DocumentBuilderFactory.newInstance().newDocumentBuilder().parse(configFile);
String selectDept = xpath.evaluate("//hrissync-database/select-dept", document);
String selectUser = xpath.evaluate("//hrissync-database/select-user", document);
```

#### 4) DB 접속 및 데이터 추출
```java
DataSource dataSource = DataSourceBuilder.create()
    .type(SimpleDriverDataSource.class)
    .username(username)
    .password(password)
    .url(url)
    .driverClassName(driverClassName)
    .build();
Connection connection = dataSource.getConnection();
```

#### 5) 추출한 데이터를 Hist 테이블에 저장
- 부서는 `deptHistRepo`
- 사용자는 `userHistRepo`

#### 6) 이후 부서, 사용자 데이터를 비교 후 Create/Update/Delete 대상으로 분리하고, 최종적으로 반영

```java
switch (userHist.getWork()) {
    case "C": userService.create(...); break;
    case "U": userService.update(...); break;
    case "D": userService.delete(...); break;
}
```

---

## 🧩 구현 시 고려한 점 및 개선 사항

- **CUBRID 드라이버 및 URL 작성 방식이 기존과 달라 설정 파일 파싱 및 연결 테스트에 시간 소요**
- **ONNARA 시스템 특성상 부서명이 중복되는 경우가 많아, 쿼리 내에서 ORGNAME에 대한 정제 로직을 포함시킴**
- **전화번호 필드의 정합성이 낮아 XML 설정 내 `<convert>`를 통해 불필요한 문자 제거 로직 구현**
- **기존 사용자/부서 데이터와의 비교 및 매핑 과정에서 코드 상 처리 로직을 명확히 나눠 안정성 확보**

---

## 📌 작업하면서 배운 점

- **XML 설정 기반으로 다수 고객사의 DB 구조를 유연하게 대응할 수 있다는 구조적 장점**을 다시 한 번 확인할 수 있었다.
- CUBRID DB 사용 경험이 처음이었지만, JDBC 기반 연결 구조만 정확히 이해하면 **SQL은 유사하게 접근 가능하다는 점**을 체감했다.
- 인사 시스템과 같은 민감한 정보는 연동 방식이 더 까다롭고 정합성이 중요하므로, **데이터 검증 및 로깅 체계 강화가 꼭 필요함**을 실감했다.
- **연동 흐름을 추적할 수 있도록 각 단계마다 로그를 남기고, 중간 저장 및 백업 구조를 명확히 구축**해야 유지보수 시 큰 도움이 된다.

---

## 🎯 향후 개선 방향

- 고객사별로 정제 로직(XML 내 convert 태그 등)을 **공통화하고 재사용 가능한 도구로 분리**
- **변경 이력 추적을 위한 로그 데이터 관리 및 UI에서의 모니터링 기능 보강**
- 다양한 DB 연동을 위한 테스트 환경 구축: **CUBRID, Tibero, Altibase 등 추가 확보 필요**
- 복잡한 쿼리나 정규표현식 정제 로직은 향후 DSL 구조로 개선 고려

