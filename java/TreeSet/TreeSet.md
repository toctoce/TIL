# TreeSet

- 자바의 `TreeSet`은 java.util 패키지에 포함된 정렬된 집합을 구현한 클래스이다. 
- `TreeSet`은 이진 탐색 트리의 일종인 레드-블랙 트리(Red-Black Tree) 구조로 구현되어 있으며, 요소를 저장할 때 자동으로 정렬해준다. 이 때문에 `TreeSet`은 중복을 허용하지 않으면서, 항상 정렬된 순서로 요소를 유지한다.
- 삽입, 삭제, 탐색 모두 시간복잡도가 `O(logN)`이다.
- 레드-블랙 트리 구조이기 때문에 `HashSet`과 다르게 `contains(x)`의 시간복잡도도 `O(logN)`이다.


```java
TreeSet<Integer> set = new TreeSet<>();

// 값 추가
set.add(x);

// 값 삭제
set.remove(x);

// 값 포함 여부 확인
set.contains(x);

// 크기 확인
set.size();

// 비어있는지 확인
set.isEmpty();

// 가장 작은 값 조회
set.first();

// 가장 큰 값 조회
set.last();

// 기준 값보다 큰 값 조회
set.higher(x);

// 기준 값보다 작은 값 조회
set.lower(x);

// 기준 값 이상인 값 조회
set.ceiling(x);

// 기준 값 이하인 값 조회
set.floor(x);

// 가장 작은 값 제거 후 반환
set.pollFirst();

// 가장 큰 값 제거 후 반환
set.pollLast();

// 전체 삭제
set.clear();

// 기준 값보다 작은 값 조회
set.headSet(30); // [10, 20]

// 기준 값 이상인 값 조회
set.tailSet(30); // [30, 40]

// 범위 값 조회
set.subSet(20, 40); // [20, 30]

// 내림차순 조회
set.descendingSet(); // [40, 30, 20, 10]
```
