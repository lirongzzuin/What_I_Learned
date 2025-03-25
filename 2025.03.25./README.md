# 📌 운영 중인 DB 구조 수정 및 데이터 복제 작업

## 📝 개요
운영 중인 데이터베이스를 활용하여 기능을 추가하는 과정에서 **DB 구조 변경이 필요**했다.  
기존 테이블을 직접 수정할 경우 운영 환경에 영향을 줄 수 있기 때문에 **복제 테이블을 생성하여 변경 사항을 적용하는 방식**으로 진행했다.

---

## 🚀 구현 내용

### 🔹 1. 기존 테이블 복제 및 구조 변경
✅ 기존 테이블(`version_data`, `content_data`)을 직접 수정하는 대신 **복제 테이블(`version_data_copy`, `content_data_copy`)을 생성하여 변경 사항을 적용**
✅ `content_data_copy` 테이블에 **새로운 컬럼 추가 (`config_data`, `script_content`)**

📌 **테이블 컬럼 추가**
```sql
ALTER TABLE content_data_copy
ADD COLUMN config_data LONGTEXT,
ADD COLUMN script_content LONGTEXT;
```

✅ 기존 데이터에서 필요한 정보를 활용하여 **새로운 컬럼에 데이터 매핑**

📌 **config_data 컬럼 값 업데이트**
```sql
UPDATE content_data_copy
SET config_data = CONCAT(
    '{',
    '"logo": "",',
    '"server_address": "', 
        COALESCE(
            NULLIF(SUBSTRING_INDEX(SUBSTRING_INDEX(metadata, '"server_address":"', -1), '"', 1), ''),
            NULLIF(remote_host, ''),
            ''
        ), 
    '",',
    '"main_page": "",',
    '"privacy_policy": "",',
    '"server_port": "', 
        COALESCE(
            NULLIF(SUBSTRING_INDEX(SUBSTRING_INDEX(metadata, '"server_port":"', -1), '"', 1), ''),
            NULLIF(remote_port, ''),
            ''
        ), 
    '",',
    '"title": "",',
    '"main_page_id": "",',
    '"server_description": "', 
        COALESCE(
            CASE 
                WHEN NULLIF(SUBSTRING_INDEX(SUBSTRING_INDEX(metadata, '"server_address":"', -1), '"', 1), '') LIKE 'http://%' 
                THEN SUBSTRING_INDEX(SUBSTRING_INDEX(SUBSTRING_INDEX(metadata, '"server_address":"', -1), '"', 1), 'http://', -1)
                WHEN NULLIF(SUBSTRING_INDEX(SUBSTRING_INDEX(metadata, '"server_address":"', -1), '"', 1), '') LIKE 'https://%' 
                THEN SUBSTRING_INDEX(SUBSTRING_INDEX(SUBSTRING_INDEX(metadata, '"server_address":"', -1), '"', 1), 'https://', -1)
                ELSE NULLIF(SUBSTRING_INDEX(SUBSTRING_INDEX(metadata, '"server_address":"', -1), '"', 1), '')
            END,
            NULLIF(remote_host, ''),
            ''
        ), 
    '",',
    '"agreement_title": "",',
    '"content_version": ""',
    '}'
);
```

---

### 🔹 2. 테이블 간 관계 수정
✅ `version_data_copy` 테이블에 **`content_id` 컬럼을 추가**하여 데이터 관계를 명확히 설정
✅ `content_data_copy`의 `name`과 `version_data_copy`의 `name`을 비교하여 일치하는 경우 `content_id` 값을 설정하도록 수정

📌 **content_id 컬럼 추가 및 데이터 업데이트**
```sql
ALTER TABLE version_data_copy ADD COLUMN content_id INT;

UPDATE version_data_copy v
LEFT JOIN content_data_copy c ON v.name = c.name
SET v.content_id = IFNULL(c.id, 0000);
```

✅ 기존에는 `version_data`와 `content_data` 간의 관계가 명확하지 않아 **데이터 조회 및 연관 관리가 어려웠던 문제 해결**
✅ `content_data_copy`와 `version_data_copy`를 연결하여 **데이터 정합성을 유지하면서 신규 기능을 추가할 수 있는 기반 마련**

---

## 📌 작업하면서 배운 점
- **운영 중인 데이터베이스를 직접 수정하는 것은 리스크가 크므로, 복제 테이블을 활용하는 방식이 안전하다.**
  - 기존 데이터를 유지하면서 새로운 구조를 테스트하고 적용할 수 있어 배포 안정성이 향상됨.
  
- **테이블 간 관계를 명확히 정의하면 데이터 정합성을 보장할 수 있다.**
  - `version_data_copy`와 `content_data_copy` 간의 관계를 명확히 설정하여, 데이터 조회 및 관리가 용이해짐.
  
- **컬럼 데이터를 변환할 때 기존 데이터를 활용하는 방식이 효과적이다.**
  - `metadata` 컬럼에서 `server_address` 및 `server_port` 정보를 추출하여 새로운 `config_data` 컬럼을 채우는 방식으로 데이터 마이그레이션 진행.

---

## 🎯 향후 개선 방향
- **기존 테이블과 신규 테이블 간의 데이터 동기화 로직 추가 검토**
- **운영 환경에서 실제 사용되는 데이터에 대한 테스트 및 검증 강화**
- **DB 변경 사항을 배포할 때 영향도를 최소화할 수 있는 전략 수립**

