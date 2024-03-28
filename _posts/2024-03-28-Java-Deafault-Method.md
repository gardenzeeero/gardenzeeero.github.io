---
title: JVM Architecture (JVM 메모리구조)
date: 2024-03-21 +09:00
categories:
  - Study
  - Java
tags:
  - 자바
---
# Default 메서드 등장배경
---
java 8 부터 Interface에 default method를 추가할 수 있게 되었다.

그럼 이 Default 메서드는 왜 추가된걸까?

그 이유는 아래의 같은 상황을 생각해보면 된다.

![](images/2024-03-28-Java-Deafault-Method.png)

만약 Interface A를 구현 세개의 Class가 있다고 가정해보자.   
만약 Interface의 수정사항이 생긴다면 어떻게 해야할까?   
당연히 Interface를 구현한 Class A, B, C를 모두 수정해주어야 한다. 

뭐 세개니까 이정도는 할만하다.

근데 만약 아래와 같은 상황이라면??

![](images/2024-03-28-Java-Deafault-Method-1.png)

보기만해도 답도 없다. 저 많은 클래스들을 모두 수정할 수 있을까?   

그래서 등장한 것이 Default Method이다. 

# Default Method
---
## Default Method 란?
---
Default Method는 **인터페이스 내에 구현된 메서드**이다. 

```java
public interface InterfaceName{
	void absMethod1();
	void absMethod2();

	default int defaultMethod(){
		...
	}
}
```

Default Method는 위와 같은 형태로 선언하게 된다.

보다시피 추상 메서드와 다른 점이 몇가지 존재한다.
1. `default` 예약어가 붙는다. 
2. `{}` 구현부가 존재한다.

## Default Method 의 장점
---
위에서 보았던 문제점을 해결할 수 있다.

**문제점** : 인터페이스를 수정하면 구현한 **모든 클래스를 수정**해야한다.  
**해결 방법** : 새로 추가하는 Method를 **Default Method로 선언**한다.

## Default Method 의 문제점
---
클래스의 다중상속을 막은 이유를 떠올려보면 답이 나온다.   
클래스의 다중상속을 막은 이유는 메서드의 충돌을 막기 위해서이다.

만약 클래스 A와 B가 모두 `play()`라는 선언부가 동일한 메서드를 가지고 있다고 가정하자   
다중상속이 가능해 A와 B를 모두 상속받고 `play()`메서드를 사용하면 어떻게 될까?   
자식의 입장에서는 **A의 `play()`와 B의 `play()`** 중 어떤 메서드를 써야할 지 **혼돈**이 온다. 

자바는 이러한 상황을 막기 위해 **다중상속을 막았다.**    
**Interface**의 경우 구현부가 없는 **추상메서드만 존재**하기 때문에 어떤 메서드를 쓰던 상관없다.

하지만 **Default Method가 나온 이후**로는 상황이 달라졌다.   
구현부가 존재할 수 있기 때문에 **Method의 충돌**이 일어날 수 있다.

아쉽지만 이러한 경우 해당 클래스로 가서 **Default Method를 재정의** 해줘야한다.

# 개방 폐쇄 원칙 : OCP(Open Close Principle)
---
객체지향 설계 5대원칙(SOLID) 중 **개방 폐쇄 원칙**이 있다.

개방 폐쇄 원칙이 뭘까?

**소프트웨어 개체(클래스, 모듈, 함수 등등)는 확장에 대해 열려 있어야 하고, 수정에 대해서는 닫혀 있어야 한다.**

## 개방 패쇄 원칙의 두 가지 속성
---
1. **Open : 확장에 열려 있다.**
	- 모듈의 **동작을 확장**할 수 있다는 것을 의미한다.
	- 애플리케이션의 요구 사항이 변경될 때, 이 변경에 맞게 새로운 동작을 추가해 모듈을 확장할 수 있다. 즉, **모듈이 하는 일을 변경**할 수 있다.
2. **Close : 수정에 닫혀 있다.**
	- 모듈의 **소스 코드를 수정하지 않아도** 모듈의 기능을 **확장하거나 변경**할 수 있다.


즉, 기존의 코드를 건드리지 않고 기능을 확장해야 이 원칙을 지킬 수 있다.

## Default Method의 OCP
---
Default Method는 **개방 폐쇄 원칙(OCP)를 준수 하기 위해 등장**한 것이다.

만약 **Interface를 수정**하고 구현한 클래스들을 수정했다면?

소스코드를 수정하지 않고 기능을 확장한다는 **폐쇄(Close)원칙을 위반한 것**이다.

따라서 자바는 해당 **원칙을 지키는 방향**으로 Default Method를 추가한 것이다.


# 출처 및 참고자료
---
[그림 및 참고자료](https://velog.io/@heoseungyeon/%EB%94%94%ED%8F%B4%ED%8A%B8-%EB%A9%94%EC%84%9C%EB%93%9CDefault-Method)
