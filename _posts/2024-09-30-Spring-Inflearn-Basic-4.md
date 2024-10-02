---
title: "[Spring] 스프링 컨테이너 조작하기"
date: 2024-10-02 +09:00
categories:
  - Study
  - Spring-Inflearn
tags:
  - 스프링
  - 스프링부트
---
해당 글은 김영한님의 Spring 강의를 듣고 한차시를 요약적으로 정리한 것이다. 기술의 단점을 보완하고 발전시키는 방향으로 진행된다. 따라서, 아래의 글이 최적의 방법이 아닐 수 있다. 기술 등장 배경과 이점에 집중해서 보자.

## 스프링 컨테이너 생성
---
보통 `ApplicationContext`를 스프링 컨테이너라고 한다. `ApplicationContext`는 인터페이스 이고 XML기반으로 만들 수도 있고, 어노테이션 기반으로 만들 수도 있다.

어노테이션 기반으로  스플이 컨테이너를 만드는 방법은 아래와 같다.
```java
ApplicationContext applicationContext = 
					new AnnotationConfigApplicationContext(AppConfig.class)
```