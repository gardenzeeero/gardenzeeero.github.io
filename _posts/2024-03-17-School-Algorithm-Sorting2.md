---
title: "[Java] Counting, Bucket, Radix Sort"
date: 2024-03-08 +09:00
categories:
  - School
  - Algorithm
tags:
  - 알고리즘
  - 정렬
---
## 선형정렬 알고리즘
---
- 키에 대한 추가적인 <b>제약조건</b>이 존재
	- 예: 키 값의 범위, 키의 자리수
- Counting, Bucket, Radix 등이 있음

## Counting Sort
---
- 제약 조건
	- **키 값이 0~K-1 사이의 정수**
	- 입력 배열(A), 임시 배열(C), 결과 배열(B) 총 세개의 배열이 필요
- 기본 개념
	- 입력 배열에서 모든 값들의 **빈도수**를 체크
	- 해당 빈도수를 가지고 뒤에서 부터 index값을 찾아 결과 배열에 저장
- 알고리즘
	1. A의 모든 값 들의 빈도수를 B에 저장
	2. B를 순회하며 누적 값 저장
		- **인덱스**로서 바로 접근하기 위함
	3. 뒤에서 부터 읽으며 해당 값을 B에서 찾아 해당 C 인덱스에 저장
		- -1한 인덱스에 저장(B에 저장된 값은 인덱스가 아니라 누계이기 때문)
	4. A를 다 순회할 때까지 반복
- 성능
	- 시간 복잡도: **O(N + K)**
		- K가 N<sup>3</sup>이면 O(N<sup>3</sup>)이 됨
	- 공간 복잡도: O(N + K)
- 특징
	- **K의 값이 크면 무의미함** (비교기반 정렬이 더 유리할 수도 있음)
	- K의 크기가 N과 비슷하다면 O(N)만에 정렬 가능
	- **Stable**

### 구현
---
```java
public class Counting{  
    public static int[] sort(int[] a, int k){  
        int N = a.length;  
        int[] c = new int[k], b = new int[k];  
  
        for(int i=0; i<N; i++) c[a[i]]++;  
        for(int i=1; i<k; i++) c[i] += c[i-1];  
        for(int i=N-1; i>=0; i--) b[--c[a[i]]] = a[i];  
          
        return b;  
    }  
}
```

## Bucket Sort
---
- 제약 조건
	- **키 값이 0~K-1 사이의 정수**
- 기본 개념
	- 버킷을 나누어 Counting Sort를 여러번
		- 버킷이 하나면 그냥 Counting Sort임
- 알고리즘
	1. 비어있는 버킷을 적절히 준비
	2. 분할: 각 원소들을 해당하는 버킷에 저장
	3. 정렬: 각 버킷을 정렬
	4. 수집: 각 버킷을 차례로 방문해 원래 배열에 저장
- 성능
	- 시간 복잡도: 각 버킷을 정렬하는데 걸리는 시간의 합
		- Counting Sort로 진행하므로 버킷의 범위와 담기는 값의 개수에 따라 달라짐
- 특징
	- 버킷 수가 2인 정렬 알고리즘 -> Quick Sort
	- 배열 원소가 퍼져있어야 유리
	- Stable
	- 제자리정렬 X

## Radix sort
---
- 제약조건
	- **키가 d개의 요소로 이루어져 있어야함**
		- 자리수, 무늬와 숫자 등
- 종류
	- **높은 자리 우선** 정렬 (MSD Sort)
		- Most Significant Digit Sort
		- Key 1을 기준으로 버킷정렬
		- 각 Key 1으로 나누어진 요소를 각각 Key2로 정렬
		- 예시: 트럼프 카드
			- 무늬로 먼저 정렬
			- 무늬 별로 숫자 정렬
			- 무늬 순서로 합치기
	- **낮은 자리 우선** 정렬 (LSD Sort)
		- Least Significant Digit Sort
		- Key 2를 기준으로 버킷정렬
		- 다시 Key 1을 기준으로 버킷정렬
		- MSD와 다르게 Key 1 별로 각각 정렬하는 단계가 없어 많이 사용
- 성능
	- Key의 개수 d
	- O(d * 정렬 알고리즘 시간복잡도)
		- O(d * (n + k)): Counting Sort 사용시
- 특징
	- 보통 정수형을 정렬할 때 유용
		- 그래서 버킷 정렬을 사용하는 것임

### 구현
---
```java
public class Radix {  
    public static void sort(int[] a){  
        int m = a[0], exp = 1, N = a.length;  
        int[] b = new int[N]; //정렬 할 때 임시 저장  
  
        //제일 큰 수를 찾아서 while문을 돌릴 때 자리수를 파악  
        for(int i=0; i<N ;i++) if(a[i] > m) m = a[i];  
  
        while(m / exp > 0){  
            int[] c = new int[10]; //Counting Sort 임시 배열  
            for(int i=0; i<N; i++) c[(a[i] / exp) % 10]++;  
            for(int i=1; i<10; i++) c[i] += c[i-1];  
            for(int i=N-1; i>=0; i--) b[--c[(a[i] / exp) % 10]] = a[i];  
            for(int i=0; i<N; i++) a[i] = b[i];  
  
            exp *= 10;  
        }  
    }  
}
```