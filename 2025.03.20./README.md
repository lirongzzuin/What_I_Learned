# 📌 MyBatis를 활용한 조직도 기반 사용자 조회 정렬 문제 해결

## 📝 개요
웹 애플리케이션에서 특정 회사의 사용자 목록을 조회할 때, **부서 조직도를 기반으로 정렬**이 되어야 하지만 조회된 데이터의 정렬 순서가 일관되지 않는 문제가 발생했다. 기존 쿼리는 `sort_order`를 활용하여 정렬하려 했지만, 일부 사용자들이 올바르게 정렬되지 않는 문제가 있었다.

이 문제를 해결하기 위해 **상위 코드와 조회 순서(`sort_order`)를 활용하여 정렬 기준을 명확히 지정**하는 방식으로 쿼리를 개선했다.

---

## 🚀 구현 내용

### 🔹 1. 기존 문제점
- 조직도를 기반으로 사용자 목록을 조회할 때, **정확한 정렬이 이루어지지 않음**.
- 일부 사용자들이 `sort_order` 값이 없거나 `0`으로 설정되어 있어, 정렬 우선순위에서 예외적인 동작이 발생.
- 상위 부서와의 계층 구조를 고려한 정렬이 필요.

### 🔹 2. 해결 방법
- **조직 계층 구조를 반영하기 위해 `WITH RECURSIVE`를 사용하여 회사별 조직도를 재귀적으로 조회**.
- **사용자의 부서 정렬 순서를 가져오기 위해 `dept_sort_order` 값을 추가**하여 조직도 기반 정렬을 보장.
- **정렬 기준을 명확히 설정**하여 데이터 조회 시 일관된 순서를 유지.

📌 **수정 후 MyBatis 쿼리**
```sql
WITH RECURSIVE CTE AS (
    SELECT code, parent_code
    FROM company_structure
    WHERE code = #{company_code} AND enabled = true
    UNION ALL
    SELECT c.code, c.parent_code
    FROM company_structure c 
    INNER JOIN CTE ON c.parent_code = CTE.code AND c.enabled = true
)
SELECT  
    u.*, 
    d.name AS department_name, 
    d.full_hierarchy AS department_hierarchy,
    (SELECT sort_order 
     FROM department_structure 
     WHERE code = u.department_code AND company_code = u.company_code) AS department_sort_order
FROM users u
JOIN department_structure d 
    ON u.company_code = d.company_code 
    AND u.department_code = d.code
JOIN (
    SELECT * FROM CTE
) c ON c.code = u.company_code
WHERE u.sort_order IS NOT NULL AND u.sort_order > 0 -- 정렬 우선순위가 없는 사용자 제외
ORDER BY 
    u.company_code, 
    department_sort_order, 
    u.sort_order;
```

---

## 📌 작업하면서 배운 점
- **조직도를 기반으로 한 데이터 조회 시 계층 구조를 반영하는 것이 중요하다.**
  - `WITH RECURSIVE`를 활용하여 조직도를 효과적으로 조회하고, 하위 조직까지 포함할 수 있도록 구현.
  
- **정렬 기준을 명확히 정의해야 한다.**
  - 사용자별 정렬 순서를 지정하는 `sort_order` 값이 없거나 `0`인 경우를 고려해야 데이터 일관성이 유지됨.
  
- **데이터 조회 성능을 고려해야 한다.**
  - 조인(`JOIN`)이 포함된 쿼리는 최적화가 필요하며, `INDEX`를 활용하여 조회 성능을 개선할 수 있음.

---

## 🎯 향후 개선 방향
- **조직도 데이터를 캐싱하여 반복 조회 시 성능 최적화**
- **사용자별 정렬 순서를 동적으로 설정할 수 있도록 관리 기능 추가**
- **데이터 정렬 방식에 대한 테스트 케이스 추가 및 검증 강화**

