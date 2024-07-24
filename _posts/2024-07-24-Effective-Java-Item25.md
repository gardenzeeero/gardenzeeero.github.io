---
title: "[Effective Java] 톱레벨 클래스는 한 파일에 하나만 담으라"
date: 2024-07-24 +09:00
categories:
  - Study
  - Java
tags:
  - 자바
  - 이펙티브자바
---
해당 글은 Effective Java를 공부하며 정리한 글이다.     
[정리된 Github](https://github.com/gardenzeeero/effective-java-study)

>하나의 소스파일에는 하나의 톱레벨 클래스만 선언해라

## 톱레벨 클래스
---
톱레벨 클래스란 다른 클래스안에 정의된 것이 아닌,   
**독립적으로 정의된 클래스**를 말한다.

즉, **중첩 클래스(내부 클래스)가 아닌 클래스**이다.

제목에서 볼 수 있듯이 톱레벨 클래스는 한 파일에 하나만 담아야한다.

## 한 파일에 하나만 담아야하는 이유
---
여러 톱레벨 클래스를 하나의 파일에 담으면 생기는 문제가 뭘까?

아래와 같이 A.java 파일이 있다고 가정하자

```java
class A{
	static final String NAME = "A"
}

class B{
	static final String Name = "B"
}
```

내용과 소스코드 이름만 다른 경우를 생각해보자    
아래는 소스파일 이름을 B.java라고 하자

```java
class A{
	static final String NAME = "AA"
}

class B{
	static final String NAME = "BB"
}
```

A와 B를 모두 사용하는 경우를 가정하면 문제가 생긴다.테스트 코드는 아래와 같다.

```java
public class Test{
	public static void main(String[] args){
		System.out.println(A.Name + B.Name);
	}
}
```

위의 코드는 컴파일 하는 순서에 따라 다른 결과가 나온다.
1. **javac Test.java B.java**
	- A.NAME을 참조할 때 A가 없는 것을 발견하고 A.java를 불러온다.
	- B는 A.java에 선언되어있어 넘어간다.
	- B.java를 컴파일 할 때 이미 A.java에 선언되어 있다고 컴파일 오류가 발생한다.
	- **결론 : 컴파일 오류**
2. **javac Test.java A.java**
	- A.NAME을 참조할 때 A가 없는 것을 발견하고 A.java를 불러온다.
	- B는 A.java에 선언되어있어 넘어간다.
	- A.java는 이미 불러왔으므로 넘어간다.
	- **결론 : AB 출력**
3. **javac B.java Test.java**
	- B.java를 컴파일한다.
	- A.NAME과 B.NAME이 B.java에 모두 선언되어 있으므로 넘어간다.
	- **결론 : AABB 출력**

운좋게 1번과 같은 순서로 컴파일을 한다면 컴파일 오류로 잡아 낼 수 있다.

하지만, 2번과 3번과 같이 컴파일 오류가 나지 않는 경우엔 **순서에 따라 출력이 달라질 수 있다.**

**(사실 킹갓 IDE가 잡아준다.)**

## 결론
---
1. 하나의 소스파일에 하나의 톱레벨 클래스만 두자
2. private으로 선언한 내부 클래스를 쓰자

```java
public class A{
	private static class B{
		//이런식으로 선언해서 A내부에서만 사용
	}
}
```