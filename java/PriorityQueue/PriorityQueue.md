# PriorityQueue

- 자바의 `PriorityQueue`는 java.util 패키지에 포함된 우선순위 큐를 구현한 클래스이다.
- `PriorityQueue`는 내부적으로 힙(Heap) 구조를 사용하며, 기본적으로 가장 작은 값이 먼저 나온다.
- 삽입은 `O(logN)`, 최우선 값 조회는 `O(1)`, 최우선 값 삭제는 `O(logN)`이다.
- 특정 값 탐색과 삭제는 `O(N)`이다.

```java
PriorityQueue<Integer> queue = new PriorityQueue<>();

// 값 추가
queue.add(x);
queue.offer(x);

// 가장 우선순위가 높은 값 조회
queue.peek();

// 가장 우선순위가 높은 값 제거 후 반환
queue.poll();

// 특정 값 삭제
queue.remove(x);

// 값 포함 여부 확인
queue.contains(x);

// 크기 확인
queue.size();

// 비어있는지 확인
queue.isEmpty();

// 전체 삭제
queue.clear();

// 내림차순 우선순위 큐 생성
PriorityQueue<Integer> reverseQueue = new PriorityQueue<>(Collections.reverseOrder());

// 객체 정렬
PriorityQueue<Member> members = new PriorityQueue<>(
    Comparator.comparingInt(member -> member.age)
);
```
