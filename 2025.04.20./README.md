# 📌 What I Learned – 해시테이블(Hash Table) 문제 풀이 방법

---

## 📝 해시테이블(Hash Table)이란?

- 키(Key)를 이용해 데이터를 저장하고 검색할 수 있는 자료구조
- 배열(Array) + 해시함수(Hash Function) 조합으로 구현
- **시간 복잡도**
  - 삽입: 평균 O(1)
  - 검색: 평균 O(1)
  - 삭제: 평균 O(1)

---

## ✅ 해시테이블이 필요한 경우

- 빠르게 데이터의 존재 여부를 확인할 때
- 키-값 매핑이 필요할 때
- 빈도수를 세야 할 때
- 빠른 조회가 필요할 때

---

## 🔍 코딩 테스트 해시테이블 문제 접근 흐름

```
1. 문제를 읽고 "값을 빠르게 찾아야 하는가?" 판단
2. 해시맵(딕셔너리) 사용 여부 결정
3. 키(Key)와 값(Value) 정의
4. 해시맵에 데이터 저장
5. 해시맵 조회 및 가공
6. 결과 반환
```

---

## ⚙️ 해시테이블 기본 패턴

```python
hash_map = {}

# 삽입
hash_map[key] = value

# 검색
if key in hash_map:
    value = hash_map[key]

# 업데이트
hash_map[key] += 1

# 삭제
del hash_map[key]
```

---

## 🛠️ 해시테이블 활용 예시 흐름

### 1. 문자열에서 가장 많이 등장한 문자 찾기

```python
def most_common_char(s):
    freq = {}
    for c in s:
        freq[c] = freq.get(c, 0) + 1

    max_char = ''
    max_count = 0
    for char, count in freq.items():
        if count > max_count:
            max_char = char
            max_count = count
    return max_char
```

---

### 2. 두 배열의 공통 원소 찾기

```python
def intersection(arr1, arr2):
    seen = set(arr1)
    result = []
    for num in arr2:
        if num in seen:
            result.append(num)
    return result
```

---

## 📂 해시테이블 문제 풀이 시 주의할 점

| 항목 | 설명 |
|------|------|
| 키 선택 | 중복이 없어야 하고 비교가 빠른 타입 사용 (문자열, 숫자) |
| 해시 충돌 | 일반 문제에서는 고려하지 않아도 되지만, 키 중복에 주의 |
| get() 메서드 사용 | dict.get(key, default)로 KeyError 방지 |
| 시간 복잡도 고려 | 해시맵의 O(1) 특성을 적극 활용 |

---

## ✅ 정리

- 해시테이블은 **빠른 검색, 삽입, 삭제**가 필요한 문제에 적합
- **해시맵 생성 → 저장 → 조회 및 가공**의 기본 흐름으로 해결
- 키와 값을 문제 상황에 맞게 설정하는 것이 핵심
- 해시테이블을 활용하면 다양한 유형의 코딩 테스트 문제를 효율적으로 해결할 수 있다
