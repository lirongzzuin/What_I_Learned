# 📌 What I Learned – 스택(Stack) & 큐(Queue) 문제 풀이 방법

---

## 📝 스택(Stack)과 큐(Queue)이란?

| 자료구조 | 특징 |
|----------|------|
| 스택(Stack) | Last In, First Out (LIFO). 가장 나중에 넣은 데이터가 가장 먼저 나온다. |
| 큐(Queue) | First In, First Out (FIFO). 가장 먼저 넣은 데이터가 가장 먼저 나온다. |

- 스택: 한 방향으로만 데이터를 넣고 뺀다 (push, pop)
- 큐: 한쪽에서는 데이터를 넣고(Enqueue), 다른 쪽에서는 데이터를 뺀다(Dequeue)

---

## ✅ 스택/큐 문제를 푸는 흐름 순서

```
1. 문제를 읽고 "자료가 순서대로 관리되어야 하는가?" 판단
2. LIFO(후입선출) → 스택 사용
3. FIFO(선입선출) → 큐 사용
4. 스택 또는 큐를 초기화
5. 삽입과 삭제 규칙 구현
6. 반복문과 조건문으로 문제 요구사항 해결
7. 결과 반환
```

---

## ⚙️ 스택 & 큐 기본 사용법

### 스택(Stack)

```python
stack = []

# 삽입
stack.append(item)

# 삭제
item = stack.pop()

# 비어있는지 확인
if not stack:
    ...
```

### 큐(Queue)

```python
from collections import deque

queue = deque()

# 삽입
queue.append(item)

# 삭제
item = queue.popleft()

# 비어있는지 확인
if not queue:
    ...
```

---

## 🛠️ 스택 & 큐 문제 예시 흐름

### 1. 스택 문제 예시: 괄호 유효성 검사

```python
def is_valid_parentheses(s):
    stack = []
    for char in s:
        if char == '(':
            stack.append(char)
        elif char == ')':
            if not stack:
                return False
            stack.pop()
    return not stack
```

---

### 2. 큐 문제 예시: 프린터 문제

```python
from collections import deque

def printer_queue(jobs, target_idx):
    queue = deque([(i, p) for i, p in enumerate(jobs)])
    count = 0
    while queue:
        idx, priority = queue.popleft()
        if any(p > priority for _, p in queue):
            queue.append((idx, priority))
        else:
            count += 1
            if idx == target_idx:
                return count
```

---

## 📂 스택 & 큐 문제 풀이 시 주의할 점

| 항목 | 설명 |
|------|------|
| 데이터 삽입/삭제 방향 | 스택은 끝에서 삽입/삭제, 큐는 양쪽 다르게 삽입/삭제 |
| 비어있는지 체크 | pop, popleft 전에 반드시 스택/큐가 비었는지 확인 필요 |
| 시간복잡도 고려 | 큐는 deque를 사용해 popleft를 O(1)로 처리 |
| 무한 루프 주의 | 조건을 잘못 걸면 무한 반복 발생 가능 |

---

## ✅ 정리

- 스택은 LIFO, 큐는 FIFO 특성을 기억한다
- 삽입/삭제 시점을 문제에 맞게 조정한다
- 기본 동작(push, pop, enqueue, dequeue)을 정확히 이해하고 사용한다
- 자료구조 특성을 문제 해결에 적극 활용한다
