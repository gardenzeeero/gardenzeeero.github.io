---
title: "[Algorithm] Sorting 개요"
date: 2024-03-08 +09:00
categories:
  - Study
  - Algorithm
tags:
  - 알고리즘
  - 정렬
---
## 정렬의 장점
---
리스트의 항목들을 특정 순서로 배치 -> 검색 및 비교가 빨라짐

검색 : O(n) -> O(log<sub>2</sub> n) - 이진탐색을 통해 더 빨리 검색 가능   
비교 : O(m * n) -> O(m + n) - m행 리스트와 n행 리스트 비교시

## 정렬 알고리즘 분류
---
1. 데이터 수와 메모리 용량간의 관계
	- 내부 정렬
		- 데이터가 한번에 메모리에 올라갈 수 있어서 메모리에서 정렬 진행
		- 속도는 빠르지만 데이터의 양이 많으면 안됨
		- Selection, Insertion, Shell, Quick, Heap, Bubble, Merge, Radix 등
	- 외부 정렬 
		- 대용량 데이터를 여러개의 파일로 나누어 내부정렬 후 합병
		- 속도는 느리지만 데이터가 많을 때 사용

2. 데이터 값에 대한 제한 여부
	- 비교기반 정렬
		- 값을 비교하여 정렬
		- Quick, Merge, Heap 등
		- Ω(n log<sub>2</sub> n) 이상 좋아질 수 없음
	- 선형 정렬
		- 비교를 하지않고 시간복잡도가 선형적으로 증가하는 정렬
		- Counting, Radix 등이 있음
		- O(n)이 나옴


## Java를 이용한 정렬 알고리즘
---
알고리즘 구성
- 각 정렬 알고리즘에 대한 클래스를 만들어 사용
- Static method로 sort(Comparable other)를 정의
	- Comparable은 무슨 Type이든 받을 수 있도록 만든 Interface
- less(), each(), show(), isSorted() 등의 static method 정의

Comparable Interface
- 구현하는 클래스의 객체들 간 natrual order를 정의함
	- Collections.sort() 등에서 사용
	- ArrayList\<Integer\> A 일때 Collection.sort(A)와 같이 사용
	-  Integer 의 compareTo 메서드의 기준에 따라 정렬 (바꾸고 싶으면 comparator 추가)
- int compareTo()를 구현해야함
	- 정렬 기준을 정하는 것임
	- 반환값에 따라 정렬 : 음수(a<b), 0(a\==b), 양수(a>b)

## 예시
---
AbstractSort 클래스
- 각 정렬 마다 별도의 클래스를 사용
- 추상 클래스를 통해 공통적인 메서드 사용
```Java
public abstract class AbstractSort {  
    public static void sort(Comparable[] a){};  
      
    protected static boolean less(Comparable v, Comparable w){  
        return v.compareTo(w) < 0;  
    }  
      
    protected static void exch(Comparable[] a, int i, int j){  
        Comparable t = a[i]; a[i] = a[j]; a[j] = t;  
    }  
      
    protected static void show(Comparable[] a){  
        for(int i=0; i < a.length; i++){  
            System.out.print(a[i] + " ");  
        }  
    }  
      
    protected static boolean isSorted(Comparable[] a){  
        for(int i=1; i<a.length; i++){  
            if(less(a[i], a[i-1])) return false;  
        }  
        return true;  
    }  
}
```

Comparable Interface 예시
```Java
public class Date implements Comparable<Date>{  
    private final int month;  
    private final int day;  
    private final int year;  
  
    public Date(int month, int day, int year){  
        if(!isValid(month, day, year)) throw new IllegalArgumentException("Invalid date");  
        this.month = month;  
        this.day = day;  
        this.year = year;  
    }  
  
    public int compareTo(Date that){  
        if(this.year < that.year) return -1;  
        if(this.year > that.year) return +1;  
        if(this.month < that.month) return -1;  
        if(this.month > that.month) return +1;  
        if(this.day < that.day) return -1;  
        if(this.day > that.day) return +1;  
  
        return 0;  
    }  
}
```