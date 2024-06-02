---
title: "[Java] 제네릭 런타임 시 문제"
date: 2024-05-25 +09:00
categories:
  - Study
  - Java
tags:
  - 자바
---
## 제네릭
---
제네릭은 알다시피 타입 안정성과 형 변환의 번거로움을 줄여준다. 

근데 중요한 것은 **컴파일 시점**에만 타입체크를 해준다.     
즉, **런타임에는 보장되지 않는다.**

그 이유는 **타입소거(Type Erasure)**때문이다.

## 타입소거
---
타입소거는 자바 컴파일러가 **제네릭 코드를 일반 코드로 변환**할 때 사용하는 프로세스이다.

이 과정에서 제네릭 타입 정보가 컴파일 된 코드에서 제거된다.    
따라서, 컴파일된 코드에는 제네릭 타입 정보가 포함되지 않아 **런타임에 해당 정보를 사용할 수 없다.**

## 런타임시 문제점
---
```java
abstract class Animal {
    public void go() {
        System.out.println("Animal");
    }
}
class Dog extends Animal {
    public void go() {
        System.out.println("Dog");
    }
}
class Cat extends Animal {
    public void go() {
        System.out.println("Cat");
    }
}

public void go() {
    List<Dog> animals = new ArrayList<Dog>();
    animals.add(new Dog());
    animals.add(new Dog());
    animals.add(new Cat());     // Error
    doAction(animals);
}
```

위와 같이 dog 리스트에 cat을 넣으려고 하면 당연히 에러가 뜬다.

컴파일 시점에 타입 체크를 해줬기 때문에 당연한 것이다.

```java
public <T extends Animal> void doAction(List<T> animals) {

    animals.add((T) new Cat()); 

    for (Animal animal: animals) {
        animal.go();
    }
}
```

하지만 위의 메서드를 보면 타입소거 이후에도 문제가 생기지 않는다.

```java
public void doAction(List animals) {

    animals.add(new Cat()); 

    for (Animal animal: animals) {
        animal.go();
    }
}
```

하지만 Dog List를 넘겨주면 Cat이 들어갈 수 없음에도 들어가버리는 문제가 생긴다.

그래서 doAction을 호출하면 Cat이 출력되는 마법이 생긴다.