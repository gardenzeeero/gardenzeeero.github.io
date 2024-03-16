---
title: Selection, Insertion, Shell Sort
date: 2024-03-08 +09:00
categories:
  - School
  - Algorithm
tags:
  - 알고리즘
  - 정렬
---
# 선택 정렬 (Selection Sort) 
---
- 알고리즘
	1. 앞에서 부터 현재 위치를 옮겨감
	2. 리스트에서 최소값을 찾음
	3. 해당 값을 현재위치와 교환
	4. 현재위치를 한칸 뒤로 옮기고 2~3을 반복함
- 성능
	- 비교 연산 : O(n<sup>2</sup>/2)
		- n-1개와 비교 + n-2개와 비교 + ... + 1개와 비교
	- 교환 연산 : O(n)
		- n개의 자리에 값을 바꿔 넣어야함
- 특징
	- 실행 시간이 입력 데이터의 배치와 무관
		- 무작위, 오름차순, 내림차순 상관 없음
	- 데이터의 이동이 적음
	- 제자리 정렬(부가적인 저장 공간이 필요 없음)

![](images/2024-03-08-Algorithm-Sorting1.gif)

## 구현
---
```java
public class Selection extends AbstractSort{  
    public static void sort(Comparable[] a){  
        int N = a.length;  
        for(int i=0; i<N-1; i++){  
            int min = i;  
            for(int j=i+1; j<N; j++){  
                if(less(a[j], a[min])) min = j;  
            }  
            exch(a, i, min);  
        }  
        assert isSorted(a);  
    }  
  
    public static void main(String[] args) {  
        Integer[] a = {10, 4, 5, 2, 1, 8, 3, 6};  
        Selection.sort(a);  
        Selection.show(a);  
    }  
}
```

# 삽입 정렬 (Insertion Sort)
---
- 알고리즘
	1. 현재위치를 i 라고 함(0 < i < n)
	2. i번째 원소를 0부터 i-1까지 비교하여 정렬된 리스트에 추가
	3. i를 n-1까지 증가하며 반복
- 성능
	- 최악 (내림차순 정렬되어 있는 경우)
		- 비교연산 : O(n<sup>2</sup>/2)
		- 교환연산 : O(n<sup>2</sup>/2)
	- 최선 (오름차순 정렬되어 있는 경우)
		- 비교연산 : O(n)
		- 교환연산 : 0
- 특징
	- 입력데이터의 초기배치에 따라 달라짐


만약 데이터 하나의 크기가 큰 경우는? -> 데이터의 이동시 문제가 생김     
Java : 배열에 주소가 들어가 있으니 성능 영향 X     
C : 배열에 구조체 자체가 들어가 있으니 성능 영향 O

![](images/2024-03-08-School_Algorithm-Sorting1.gif)

## 구현
---
```java
public class Insertion extends AbstractSort {  
    public static void sort(Comparable[] a){  
        int N = a.length;  
        for(int i=1; i<N; i++){  
            for(int j=i; j > 0 && less(a[j], a[j-1]); j--){  
                exch(a, j, j-1);  
            }  
        }  
          
        assert isSorted(a);  
    }  
}
```

# 쉘 정렬 (Shell Sort)
---
- 삽입 정렬의 문제점을 보안함
	- 바로 앞의 원소랑 비교하기 때문에 가장 작은 원소가 들어올 경우 n-1번 교환해야 함

 - 알고리즘
	1. r개 떨어진 원소들간 삽입 정렬
		- 한번에 r개 만큼 이동이 가능함 (정렬 시간을 줄이는 원리)
	2. r을 줄이면서 1이 될 때까지 삽입 정렬
		- r == 1인 경우 일반적인 삽입 정렬
- 특징
	- h의 값이 작을수록 리스트의 정렬이 많이 되어 있음
	- h1은 어떤 값을 할당하더라도 정렬 되니까 어차피 마지막엔 정렬됨
	- 수열이 길면 단계 수가 증가해야하고 수열이 짧으면 성능향상이 적음

그럼 h-수열을 어떻게 할당하면 가장 효율적일까?
- 사실 최적은 아직 발견되지 않음
- 하지만, 보통 h<sub>i</sub> = 3 * h<sub>i-1</sub> + 1 로 사용함
	- 시간복잡도 : O(n<sup>1.5</sup>), 루트n 정도

![](images/2024-03-08-School_Algorithm-Sorting1-1.gif)

## 구현
---
```java
public class Shell extends AbstractSort {  
    public static void sort(Comparable[] a){  
        int N = a.length;  
        int h = 1;  
        while(h < N/3) h = 3*h + 1;  
          
        while(h >= 1){  
            for(int i=h; i<N; i++){  
                for(int j=i; j >= h && less(a[j], a[j-h]); j -= h){  
                    exch(a, j, j-h);  
                }  
            }  
            h /= 3;  
        }  
    }  
}
```


