---
title: "[Effective Java] 배열보다는 리스트를 사용하라"
date: 2024-07-24 +09:00
categories:
  - Study
tags:
  - 자바
  - 이펙티브자바
---
해당 글은 Effective Java를 공부하며 정리한 글이다.     
[정리된 Github](https://github.com/gardenzeeero/effective-java-study)

>배열은 공변이고 실체화 가능하다. 따라서 컴파일시 타입 안전을 보장하지 못한다.   
>따라서 배열대신 리스트를 사용하자.

## 배열 VS 제네릭 타입
---
배열과 제네릭 타입은 타입을 처리하는 방식이 다르다.

#### 배열은 공변이고 제네릭은 불공변이다.
- 공변 : 함께 변한다. 
	- Sub가 Super의 하위 타입이면 Sub[] 은 Super[]의 하위 타입이 된다.
- 불공변 : 함께 변하지 않는다.
	- 서로 다른 타입 Type1과 Type2가 있을 때, List\<Type1\>은 List\<Type2\>의 하위타입도 상위타입도 아니다.

말만 들으면 무슨 말인지 도통 알 수가 없다.    
아래의 예시를 보면 조금 더 이해하기 쉬울 수 있다.

```java
Object[] objectArr = new Long[1];
ojbectArr[0] = "String" //타입이 달라 ArrayStoreException을 던진다.
```

```java
List<Object> ol = new ArrayList<Long>();
ol.add("String");   //타입이 달라 넣을 수 없다.
```

두 코드 모두 Long을 담는 곳이기 때문에 String을 담을 수 없다.    
**하지만, 배열은 런타임에 실패하고 제네릭은 컴파일 조차 되지 않는다.**

#### 배열은 실체화 된다.
배열은 **런타임**에도 담기로 한 원소의 타입을 확인한다. (Long을 담기로 한 배열인지 체크)

반면에, 제네릭은 런타임에 **타입이 소거**된다. (List\<Long\> 이 그냥 List가 됨)    
대신 컴파일타임에 검사를 한다.

## 제네릭 배열이 불가능한 이유
---
위와 같이 배열과 제네릭은 성질이 다르다.   
따라서, 제네릭 배열을 만들 경우에 **타입이 안전하지 못하다.**

아래의 예시를 보자

```java
List<String>[] stringLists = new List<String>[1];  //1
List<Integer> intList = List.of(42);               //2
Object[] objects = stringLists;                    //3
objects[0] = intList;                              //4
String s = stringLists[0].get(0)                   //5 - 문제!!
```

1. 허용된다고 가정
2. 원소가 하나인 intList 생성
3. 배열은 공변이기 때문에 가능
4. 제네릭은 런타임에 소거되기에 배열에서 생기던 ArrayStoreException을 일으키지 않음
	- 제네릭은 실체화 되지 않기 때문
5. stringList에 List\<String\>만 담기로 했는데 List\<Integer\>가 담겨있음
	- 컴파일 할 때도 List\<String\>에서 get해서 String에 넣은거라 문제X
	- **String s에 넣으려고 하면 ClassCastException 발생**
	- 즉, 런타임에 펑~

이러한 이유 때문에 제네릭 배열을 불가능하게 만든 것이다.    
(비한정적 와일드카드 타입의 경우 실체화가 되는데 안쓴다.)

## 배열로 형변환 할 때 생기는 문제
---
E, List\<E\>, List\<String\> 과 같은 타입을 실체화 불가 타입이라고 한다.    
(타입이 소거되는 경우)

이러한 **실체화 불가타입을 배열로 형변환 할때 경고가 발생**한다.    
이 문제는 **@SafeVarargs 애너테이션**으로 해결 가능하다.(경고를 지워주는 역할)     
해당 애너테이션은 메서드 작성자가 해당 메서드가 타입 안전하다는 것을 보장하는 장치이다.(아이템 32)

이러한 문제를 해결하려면 **배열대신에 컬렉션을 사용하면 된다.**   
**조금 복잡해지고 성능이 나빠질 수 있지만 타입안정성이 보장되고 상호 운용성이 좋아진다.**


```java
/* 제네릭 쓰지 않은 버전 */
public class Chooser {
    private final Object[] choiceArray;

    public Chooser(final Object[] choiceArray) {
        this.choiceArray = choiceArray;
    }
    public Object choose(){
        Random random = ThreadLocalRandom.current();
        return choiceArray[random.nextInt(choiceArray.length)];
    }
}
```

위 클래스를 사용하려면 choose 메서드를 호출할 때마다 반환된 `Object`를 원하는 타입으로 형변환해야 한다.  
만약 **다른 타입 원소가 들어있으면 런타임시에 형변환 오류가 발생**한다.

```java
public class Chooser<T> {
    private final T[] choiceArray;

    public Chooser(Collection<T> choices) {
        this.choiceArray = (T[]) choices.toArray();
    }
	
    public Object choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceArray[rnd.nextInt(choiceArray.length)];
    }
}
```

위와 같이 제네릭을 적용하면 경고가 뜬다.    
그 이유는 **T가 무슨 타입인지 알 수 없으니** 강제 형변환이 런타임시 안전하다는 것을 보장하지 못하기 때문이다.   
(제네릭은 런타임시 타입이 소거되기 때문이다.)

동작은 하지만 컴파일러가 타입 안정성을 보장하지는 못한다.   
**결론은 배열대신 리스트를 사용하면 된다.**

```java
public class ListChooser {
    private final List<T> choiceList;

    public ListChooser(final Collection<T> choices) {
        this.choiceList = new ArrayList<>(choices);
    }
    
    public T choose(){
        Random random = ThreadLocalRandom.current();
        return choiceList[random.nextInt(choiceList.size())];
    }
}
```

이렇게 선언하면 런타입시 ClassCastException이 생길일은 없다.

## 결론
---
- **배열**
	- 공변이고 실체화 된다.
	- 런타임시 타입 안전하고 컴파일시 타입 안전하지 않다.
- **제네릭**
	- 불공변이고 실체되지 않는다.
	- 런타임시 타입 안전하지 않고 컴파일시 타입 안전하다.

따라서, **둘을 섞어 쓰다가 문제가 생기면 배열을 리스트로 대체하면 된다.**