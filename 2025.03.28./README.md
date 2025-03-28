# 📌 조직도 트리 구조 조회 기능 구현

## 📝 개요
회사 조직도를 트리 형태로 구성하여 UI에서 계층 구조를 시각적으로 표현할 수 있도록 **조직도 정보를 트리 형태로 가공하여 반환하는 기능**을 구현했다.  
조직도 정보는 재귀 쿼리를 통해 전체 부서 계층을 조회한 후, Java에서 트리 구조로 구성하여 응답값으로 전달하도록 구성하였다.

---

## 🚀 구현 내용

### 🔹 트리 구조 생성을 위한 데이터 모델 설계
✅ 조직의 각 항목(회사/부서)을 표현하는 모델 클래스를 정의하고, **부모-자식 관계를 연결할 수 있도록 `childs` 필드 추가**
✅ `buildTree()` 메서드를 통해 **정렬된 리스트를 기반으로 계층 트리 구조를 생성**

📌 **조직도 트리 모델 구조 요약**
```java
public class OrgGroup implements Comparable<OrgGroup> {
    String code;
    String parentCode;
    String name;
    String companyCode;
    String groupType; // 회사/부서 구분
    String relationCode;
    Long relationDepth;
    int sortOrder;
    List<OrgGroup> children;

    public void appendChild(OrgGroup child) {...}
    public static List<OrgGroup> buildTree(List<OrgGroup> groups) {...}
    public int compareTo(OrgGroup other) {...}
}
```

---

### 🔹 조직도 트리 조회 API
✅ 특정 회사 코드를 기준으로 **회사 및 하위 부서를 재귀적으로 조회**  
✅ Java에서 받은 리스트를 `buildTree()`를 통해 트리 구조로 변환 후 응답 반환

📌 **트리 조회 API 예시**
```java
@GetMapping("organizations/tree/{company_code}")
public ResponseEntity<?> getOrganizationTree(@PathVariable String company_code) {
    List<OrgGroup> result = new ArrayList<>();
    List<OrgGroup> companies = baseMapper.getCompanies(company_code);

    for (OrgGroup company : companies) {
        if (!company_code.equals(company.getCode())) {
            result.add(company);
        }
        result.addAll(baseMapper.getDepartmentsByCompany(company.getCode(), null));
    }

    return ResponseEntity.ok().body(TreeResponse.success(OrgGroup.buildTree(result)));
}
```

---

### 🔹 MyBatis 재귀 쿼리를 통한 부서 계층 조회
✅ `WITH RECURSIVE` 문을 활용하여 특정 회사를 기준으로 **모든 하위 부서를 재귀적으로 조회**  
✅ `relation_code`, `relation_depth` 등을 활용해 트리 구성 및 정렬을 위한 메타 정보 포함

📌 **부서 트리 쿼리 예시**
```sql
WITH RECURSIVE CTE AS (
    SELECT  code,
            IFNULL(parent_code, #{company_code}) AS parent_code,
            name,
            company_code,
            'D' AS group_type,
            CONCAT(IFNULL(parent_code, #{company_code}), '.', code) AS relation_code,
            2 AS relation_depth,
            sort_order
    FROM department
    WHERE ( (#{dept_code} IS NULL AND parent_code IS NULL) OR code = #{dept_code} )
    AND company_code = #{company_code}

    UNION ALL

    SELECT  d.code,
            d.parent_code,
            d.name,
            d.company_code,
            'D' AS group_type,
            CONCAT(cte.relation_code, '.', d.code) AS relation_code,
            cte.relation_depth + 1,
            d.sort_order
    FROM department d
    INNER JOIN CTE ON d.parent_code = cte.code
    AND d.company_code = #{company_code}
)
SELECT * FROM CTE;
```

---

## 📌 작업하면서 배운 점
- **트리 구조 데이터를 효율적으로 구성하기 위해 재귀 쿼리와 Java Tree 빌더를 함께 사용하는 방식이 효과적이다.**
- **정렬 기준(`sort_order`)과 계층 정보(`relation_depth`)를 명확히 정의하면 UI에서 일관된 구조로 출력할 수 있다.**
- **재귀 구조를 처리할 때는 순환 참조 및 무한 루프 방지를 위해 설계 구조가 명확해야 한다.**

---

## 🎯 향후 개선 방향
- **조직도 캐시 저장 또는 별도 뷰 테이블을 활용하여 성능 최적화**
- **조직도 변경 시 이벤트 기반으로 캐시 갱신 적용**
- **다국어 조직명 표현을 위한 i18n 처리 검토**

