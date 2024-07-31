---
title: "[Effective Java] 타입 안전 이종 컨테이너를 고려하라"
date: 2024-07-31 +09:00
categories:
  - Study
tags:
  - 자바
  - 이펙티브자바
---
해당 글은 Effective Java를 공부하며 정리한 글이다.     
[정리된 Github](https://github.com/gardenzeeero/effective-java-study)

>일반적으로 한 컨테이너가 다룰 수 있는 타입이 매개변수의 수로 정해져 있다.    
>하지만, 키를 타입 매개변수로 바꾸면 이런 제약이 없는 타입 안전 이종 컨테이를 만들 수 있다.    
>타입 안전 이종 컨테이너는 Class를 키로 사용하며 이런 Class 객체를 타입 토큰이라고 한다.

## 리터럴과 클래스 리터럴
---
리터럴은 **소스 코드의 고정된 값 그 자체**를 의미한다.   

정수형 리터럴 23, 따옴표로 감싸진 문자열 리터럴 "hello" 등이 있다.

클래스 리터럴은 자바에서 **클래스 타입을 나타내는 특별한 표현식**이다.    

String.class, Integer.class 등이 있다.

클래스 리터럴은 Class 객체를 나타내며, Class 객체는 해당 클래스에 대한 메타데이터를 포함하고 있다.

즉, 클래스에 대한 메타 데이터를 사용할 때 이용한다.

## 타입 토큰
---
**런타임시 제네릭은 소거**된다.     
따라서, 제네릭 타입 정보를 런타임에 유지하기 위해 클래스 리터럴을 이용한다.

이 **클래스 리터럴을 타입 토큰**이라고 한다.

타입 토큰은 제네릭 타입을 사용하는 메서드나 클래스에서, 타입 정보를 전달하는 용도로 사용된다.    
런타임에도 타입 안정성을 위해 사용한다. RestTemplate 등에 사용된다.

## 일반적인 제네릭
---
일반적으로 제네릭의 사용방식은 컬렉션이나 컨테이너에 쓰였다.   
(Set\<E\>, Map\<K, V\> 또는 ThreadLocal\<T\> 등)

이런 경우에는 Set과 Map에 담길 수 있는 종류가 **정해져 있다**.

하지만, 좀 더 유연한 컨테이너의 사용을 위해 **타입 안전 이종 컨테이너**가 나왔다.

## 타입 안전 이종 컨테이너
---
컨테이너 대신 **키를 매개변수화**하여 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공하는 방식이다.

제네릭 타입 시스템이 **값의 타입이 키와 같음을 보장**해줄 수 있다.

위의 말을 보면 이해가 어려울 수 있으니 아래의 예시를 통해 이해해보자.

#### 예시
타입별로 즐겨 찾는 인스턴스를 저장하고 검색할 수 있는 Favorites 클래스가 있다고 해보자.

```java
public class Favorites {
	public <T> void putFavorite(Class<T> type, T instance);
	public <T> T getFavorite(Class<T> type);
}
```

putFavorite() 메서드는 **Class 객체**와 **해당 객체와 동일한 타입을 가지는 instance**를 넘겨준다.    
getFavorite() 메서드는 **Class 객체**를 넘겨주면 **동일한 타입을 가지는 instance를 반환**한다. 

- Class 클래스의 리터럴 타입은 Class가 아닌 Class\<T\>이다.    
	- String.class의 타입은 Class\<String\>이고, Integer.class의 타입은 Class\<Integer\>이다.
	- 이때, String.class 같은 클래스 리터럴을 타입 토큰이라 하는 것이다.
- 키가 매개변수화 된 것 이외에는 map과 유사하다.

**Favorites 클래스의 사용 예를 살펴보자**

```java
private static class Favorites {
    private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void putFavorites(Class<T> type, T instance){
        favorites.put(Objects.requireNonNull(type), instance);
    }

    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type)); //타입소거 때문에 Object로 저장 되기 때문
    }
}

public static void main(String[] args){
		Favorites favorites = new Favorites();

    favorites.putFavorite(String.class, "Java");
    favorites.putFavorite(Integer.class, 1234);
    favorites.putFavorite(Class.class, Favorites.class);

    String favoriteStr = favorites.getFavorite(String.class);
    Integer favoriteInt = favorites.getFavorite(Integer.class);
    Class favoriteClass = favorites.getFavorite(Class.class);

    System.out.printf("%s %x %s %n", favoriteStr, favoriteInt, favoriteClass.getName());

}
```

main에서 putFavorites 메서드를 살펴보자

**String.class를 키**로 하여 "Java"를 저장한다.   
"Java"를 꺼내 쓸 때는 getFavorite에 **키값으로 String.class를 전달**한다.

String.class를 키값으로 전달한 경우 반환값은 항상 String이다. (Integer를 반환하는 일은 없다.)   
즉, **타입 안전이 보장**되는 것이다.

## 알아야 할 점
---
**1. Class\<?\>를 이용해 와일드 카드와 같은 효과를 냈다.**    
아래와 같이 favorites의 타입은 Map\<Class\<?\>, Object\> 이다. 

```java
private Map<Class<?>, Object> favorites = new HashMap<>();
```

일반적으로 `Map<?, Object>` 와 같이 선언했다면, 타입 안정성 때문에 **null 외에는 키값으로 사용할 수 없다.**

하지만 Class\<?\>는 어떤 타입의 클래스 객체를 나타내기 위한 타입이다.    
즉, 해당 와일드 카드에는 어떠한 타입도 들어갈 수 있다.    

이렇게 모든 타입을 커버할 수 있는 Class\<?\>를 사용함으로써 어떤 타입이든 키로 쓸 수 있게 된 것이다.   
(그냥 쓰면 null 빼고는 못들어가는데, 뭐든 들어갈 수 있는 Class\<T\>를 쓰면서 뭐든 키로 쓸 수 있게 된 것)

**2. value는 단순 Object 타입이다.**   
따라서, 어떤 타입이 들어오든 받아들인다.    
어떻게 들으면 타입 안전을 지키지 못한다고 생각할 수 있다.

하지만 **key와 value의 관계**에서 타입 안전을 지켜준다.    
(String.class가 key로 들어오면 String을 반환하도록 설계됨)

## 문제점
---
**1. 키를 Class 로타입으로 넘기면 타입 안정성이 깨진다.**   
(물론 컴파일 시 비검사 경고는 뜬다.)

아래의 경우를 살펴 보자

```java
f.putFavorite((Class)Integer.class, "Integer의 인스턴스가 아닙니다.");
int fInt = f.getFavorite(Integer.class);  // ClassCastException 발생
```

putFavorite를 호출 시 아무 문제 없다가 getFavorite를 호출할 때 ClassCastException이 발생한다.

해결하기 위해선 putFavorite 메서드에서 타입 캐스팅을 통해 미리 타입체크를 하면 된다.

```java
public <T> void putFavorite(Class<T> type, T instance) {
    favorites.put(Objects.requireNonNull(type), type.cast(instance));
}
```


**2. 실체화 불가 타입에는 사용할 수 없다.**    
String, String[]은 저장할 수 있지만 List\<String\>은 저장할 수없다.

List\<String\>용 Class 객체는 없기 때문이다.

이 점은 슈퍼 타입 토큰을 통해 해결을 시도할 수 있지만 다루지 않겠다.

## 결론
---
일반적으로 한 컨테이너가 다룰 수 있는 타입이 매개변수의 수로 정해져 있다.

하지만, 키를 타입 매개변수로 바꾸면 이런 제약이 없는 타입 안전 이종 컨테이를 만들 수 있다.

타입 안전 이종 컨테이너는 Class를 키로 사용하며 이런 Class 객체를 타입 토큰이라고 한다.