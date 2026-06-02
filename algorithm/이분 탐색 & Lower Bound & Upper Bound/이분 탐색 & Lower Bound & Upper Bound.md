# 이분 탐색 & Lower Bound & Upper Bound

## 이분 탐색

이진탐색은 데이터가 정렬되어 있는 상태에서 원하는 값을 찾아내는 알고리즘

```java
private static int binarySearch(int[] arr, int target) {
    // start, mid, end는 인덱스
    int start = 0;
    int end = arr.length-1;

    while (start <= end) {
        int mid = (start + end) / 2;

        if (arr[mid] < target) {
            start = mid + 1;
        }
        else if (arr[mid] > target) {
            end = mid - 1;
        }
        else return mid;
    }
    return -1;
}
```

## Lower Bound
원하는 값 이상의 값을 찾아내는 알고리즘
```java
public static int lowerBound(int[] arr, int target) {
    int start = 0;
    int end = arr.length;

    while (start < end) {
        int mid = (start + end) / 2;

        // target보다 작은 경우는 정답 후보가 될 수 없음
        // -> mid를 포함하지 않는 오른쪽 범위를 탐색
        if (arr[mid] < target) {
            start = mid + 1;
        }
        // target보다 크거나 같은 경우는 정답 후보가 될 수 있음
        // -> mid를 포함한 왼쪽 범위를 탐색
        else {
            end = mid;
        }
    }

    return start;
}
```
## Upper Bound
원하는 값 초과의 값을 찾아내는 알고리즘
```java
public static int upperBound(int[] arr, int target) {
    int start = 0;
    int end = arr.length;

    while (start < end) {
        int mid = (start + end) / 2;

        // target보다 작거나 같은 경우는 정답 후보가 될 수 없음
        // -> mid를 포함하지 않는 오른쪽 범위를 탐색
        if (arr[mid] <= target) {
            start = mid + 1;
        } 
        // target보다 큰 경우는 정답 후보가 될 수 있음
        // -> mid를 포함한 왼쪽 범위를 탐색
        else {
            end = mid;
        }
    }

    return start;
}
```
