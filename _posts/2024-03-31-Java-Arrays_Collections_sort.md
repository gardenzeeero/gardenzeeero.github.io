---
title: "[Java] Arrays.sort(), Collections.sort() 뜯어보기"
date: 2024-03-31 +09:00
categories:
  - Study
  - Java
tags:
  - 자바
---
# Arrays.sort()
---
우선 `Arrays`클래스는 **배열**을 쉽게 다루기 위해 만들었다.

배열에는 크게 **두가지 형태**의 자료가 들어갈 수 있다.
1. **기본(primitive)** 타입
2. **참조(reference)** 타입

## Arrays.sort() 알아보기
---
`Arrays.sort()`를 쓰게 되면 어떻게 정렬이 될까?

기본적으로 아무런 정렬 조건을 주지 않는다면 아래와 같이 정렬된다.
1. **기본(primitive)** 타입 : 오름차순
2. **참조(reference)** 타입 : 오름차순, 객체의 natural order에 따라

그렇다면 **정렬 조건**은 어떻게 줄 수 있을까?

API 공식문서(Java 8)를 통해 알 수 있다.

![](images/2024-03-31-Java-Arrays_Collections_sort.png)

보다시피 배열과 함께 **Comparator**를 전달한다.  
또한 매개변수로 **제네릭 타입 배열**을 받는다.  
즉, **기본형 변수**의 경우 **형변환을 해서 넘겨줘야 정렬조건을 바꿀 수 있다.**

그렇다면 Comparator은 어떻게 만들까?
(이건 다른글에서 알아보자)

## Arrays.sort() 뜯어보기
---
그렇다면 Arrays.sort()는 어떻게 정렬을 하는걸까?

1. **기본형(primitive)** 배열

```java
public static void sort(int[] a) {  
    DualPivotQuicksort.sort(a, 0, 0, a.length);  
}
```

보다시피 **DualPivotQuicksort**를 채택하고 있다.

`DualPivotQuicksorts`은 **pivot을 두개** 사용해 `Quicksort`보단 **조금 더 성능이 높다.**  
(하지만 시간복잡도 **빅-오는 동일**하다고 한다. 이는 다른 글을 통해 알아보자)

2. **참조형(reference)** 배열

```java
public static void sort(Object[] a) {  
    if (LegacyMergeSort.userRequested)  
        legacyMergeSort(a);  
    else  
        ComparableTimSort.sort(a, 0, a.length, null, 0, 0);  
}
```

보다시피 **ComparableTimSort**를 사용한다.  
LegacyMergeSort라고도 있는데 주석으로 나중에 사라질 것이라고 한다.(근데 계속 안사라지고 있다..)  

`ComparableTimsort`는 삽입(Insertion) 정렬과 병합(Merge) 정렬을 섞어 쓴 정렬방법이다.  
둘의 장점을 가져왔다고 보면 된다.


# Collections.sort()
---
우선 `Collections`클래스는 **Collection**을 쉽게 다루기 위해 만들었다.

## Collections.sort() 알아보기
---
`Collections.sort()`를 쓰게 되면 어떻게 정렬이 될까?

API 공식문서(Java 8)를 통해 알 수 있다.
![](images/2024-03-31-Java-Arrays_Collections_sort-1.png)

Arrays.sort()와 동일하게 정렬조건을 주지 않는다면 **요소의 natural order**로 정렬된다.   
**natural order**가 있어야하니 해당 요소는 당연히 **Comparable 인터페이스를 구현**하고 있어야한다.
보통 **오름차순**으로 정렬된다고 보면 된다. (사용자가 선언한 클래스가 아니라면)

## Collections.sort() 뜯어보기
---
그렇다면 Collections.sort()는 어떻게 정렬을 하는걸까?

코드를 뜯어봤다.

```java
public static <T extends Comparable<? super T>> void sort(List<T> list) {  
    list.sort(null);  
}

//List 인터페이스의 sort 디폴트 메서드
default void sort(Comparator<? super E> c) {  
    Object[] a = this.toArray();  
    Arrays.sort(a, (Comparator) c);  
    ListIterator<E> i = this.listIterator();  
    for (Object e : a) {  
        i.next();  
        i.set((E) e);  
    }  
}
```

와우! **List 인터페이스**의 디폴트 메서드인 **sort**를 사용하고 있었다.   
이 sort는 어떻게 작동할까?

보다시피 `toArray` 메서드를 사용해 **배열로 만든 후** `Arrays.sort`를 사용하였다.
당연히 참조형 배열이니 `Arrays.sort`에서 알아봤듯이 **Timsort**를 사용하게 된다.

즉, Collections.sort도 결국 **Arrays.sort를 사용하는 것**이다.

# 시간복잡도
---
결과적으로 `Arrays.sort`와 `Collections.sort`의 시간복잡도는 아래와 같게 된다.

![](images/2024-03-31-Java-Arrays_Collections_sort-2.png)


# 출처 및 참고자료
---
[Java8 API](https://docs.oracle.com/javase/8/docs/api/)
[사진출처](https://codingnojam.tistory.com/38)