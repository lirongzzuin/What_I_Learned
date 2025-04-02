# 📌 CUBRID DB 사용 정리 (vs MySQL)

## 📝 개요
기존에 MySQL과 JPA 기반의 DB 사용에 익숙한 상태에서, 온나라 시스템 연동을 위해 CUBRID DB를 처음 접하게 되었다. 해당 고객사는 CUBRID DB를 기반으로 인사 데이터를 관리하고 있었으며, 우리는 해당 DB에서 조직도 및 사용자 정보를 추출해 시스템과 연동하는 작업을 수행했다.

이 문서는 CUBRID를 처음 사용하는 입장에서 이해해야 할 차이점들과, 연동 구현 과정에서 확인한 설정 및 특징을 정리한 내용이다.

---

## ⚙️ 기본 설정 방식 (MySQL vs CUBRID)

| 항목 | MySQL | CUBRID |
|------|--------|---------|
| JDBC URL | `jdbc:mysql://host:port/dbname` | `jdbc:cubrid:host:port:dbname` |
| 드라이버 클래스 | `com.mysql.cj.jdbc.Driver` | `cubrid.jdbc.driver.CUBRIDDriver` |
| 드라이버 의존성 | 보통 Maven/Gradle에서 자동 관리 | 별도로 jar 추가 필요 (수동 다운로드 후 라이브러리에 등록) |

```xml
<!-- CUBRID 설정 예시 -->
<username>onnara_view</username>
<password>onnara_view</password>
<conn-url><![CDATA[jdbc:cubrid:111.222.333.444:33000:onnara_db]]></conn-url>
<driver>cubrid.jdbc.driver.CUBRIDDriver</driver>
```


## 🔍 쿼리 문법상의 차이점 및 특징

### 1. ROWNUM()
- MySQL에서는 `LIMIT`을 사용하지만, CUBRID는 `ROWNUM()`이라는 함수를 통해 순번을 부여한다.
```sql
SELECT ROWNUM() AS row_no FROM some_table;
```

### 2. 문자열 함수
- MySQL에서는 `SUBSTRING()` 또는 `SUBSTR()`, `LOCATE()` 등을 사용한다.
- CUBRID에서도 `SUBSTRING()`, `INSTR()`, `LOCATE()` 등을 사용할 수 있지만, 일부 함수는 오라클 스타일에 더 가깝게 작동함.

### 3. CASE 문
- MySQL과 유사한 `CASE WHEN ... THEN ... ELSE ... END` 문법을 그대로 사용할 수 있다.

### 4. NULL 처리 방식
- `IFNULL()` 또는 `COALESCE()` 사용 가능.

### 5. 별칭 처리
- `AS` 키워드 없이도 별칭 지정이 가능하지만, 명시적으로 사용하는 것이 가독성 측면에서 더 나음.


## 📂 실무에서의 연동 흐름 요약

### ✅ XML 설정 기반 쿼리 정의
```xml
<select-dept><![CDATA[
  SELECT ORGID AS CODE, ... FROM COM_ORGANIZATIONINFO ...
]]></select-dept>
```

### ✅ 설정파일(XML) 파싱 → DB 연결
```java
Document document = DocumentBuilderFactory.newInstance().newDocumentBuilder().parse(configFile);
String url = xpath.evaluate("//hrissync-database/conn-url", document);
```

### ✅ JDBC 연결 (SimpleDriverDataSource 사용)
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

### ✅ 쿼리 실행 → 조직도 및 사용자 정보 추출
```java
PreparedStatement ps = connection.prepareStatement(selectDept);
ResultSet rs = ps.executeQuery();
while (rs.next()) {
    String code = rs.getString("CODE");
    ...
}
```

### ✅ 결과 저장 및 가공
- 추출한 결과는 내부 `Hist` 테이블에 저장한 후
- 필요 시 `Create`, `Update`, `Delete` 대상을 구분하여 처리


## ✅ CUBRID를 사용할 때 주의할 점

- 드라이버 라이브러리는 기본적으로 메이븐 중앙 저장소에 등록되어 있지 않음 → 수동 설치 필요
- `ROWNUM()`은 쿼리 순서에 따라 달라질 수 있으므로, 정렬 기준을 명확히 해야 함
- 다중 DB 연동 시 드라이버 충돌 가능성에 유의해야 함
- 트랜잭션 및 락 처리 방식이 DBMS에 따라 다를 수 있으므로, 연동 전에 쿼리 테스트 필수


---

## 🧾 정리
- CUBRID는 기본적인 SQL 문법이 MySQL과 크게 다르지 않으나, 세부 함수나 설정 방식에서 차이가 존재함
- JDBC URL, 드라이버 설정, 문자열 처리 방식 등을 중심으로 차이점을 파악하면 실제 연동에는 큰 어려움 없음
- XML 설정을 기반으로 한 유연한 쿼리 관리 구조를 활용하면 고객사마다 다른 DB 구조에도 효과적으로 대응 가능함

