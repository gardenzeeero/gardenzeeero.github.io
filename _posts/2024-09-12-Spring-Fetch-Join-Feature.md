---
title: "[Spring JPA] Fetch Join"
date: 2024-09-12 +09:00
categories:
  - Study
  - Spring
tags:
  - 스프링
  - 스프링부트
  - JPA
---
저번 글에서 N+1 문제를 해결하기 위해 Fetch Join을 사용한다고 했다. Fetch Join이 뭔지 알아보자.

## 🤔 FetchJoin이 뭔데?
---
일반적인 SQL의 Join의 종류는 아니고 **JPQL에서 성능 최적화를 위해 제공하는 기능**이다. Fetch Join을 하게 되면 **연관된 엔티티도 함께 조회**된다. 사용방법은 아래와 같다.

```
select m from Memebr m join fetch m.team
```

join 뒤에 fetch 를 붙여 사용한다. 보다시피 fetch join은 별칭을 사용할 수 없다. (하이버 네이트는 가능) 위와 같은 JPQL을 실행하면 아래와 같은 SQL문이 날라간다

```sql
SELECT
    M.*, T.*
FROM MEMBER M
INNER JOIN TEAM T ON M.TEAM_ID=T.ID
```

SQL의 결과는 아래의 그림과 같다.

![](images/2024-09-12-Spring-Fetch-Join-1.png)

## 컬렉션 Fetch Join
---
OneToMany 관계에서 Fetch Join을 하는 경우이다. 아래의 경우 하나의 팀에 여러명의 선수가 있다. 이때 team의 이름이 팀A인 team만 들고오는 상황이다.

```
select t
from Team t join fetch t.memebers
where t.name = '팀A'
```

team을 조회하면서 연관된 컬렉션도 함께 조회된다.

```sql
SELECT
    T.*, M.*
FROM TEAM T
INNER JOIN MEMEBER M ON T.ID=M.TEAM_ID
WHERE T.NAME = '팀A'
```

![](images/2024-09-12-Spring-Fetch-Join-1.png)

위와 같이, INNER JOIN이 일어나게 된다. OneToMany 관계이기 때문에 `SELECT t`를 했지만  2개의 검색 결과가 나온 것을 볼 수 있다. 따라서 **List 형태로 반환**받게 된다. 

물론 우리가 조회한 team은 하나이기 때문에 해당 **List 내부에 team은 모두 같은 인스턴스**를 가리키게 된다. 따라서 List에 존재하는 팀과 해당 팀의 멤버를 출력하면 아래와 같이 나온다.

![](images/2024-09-12-Spring-Fetch-Join-2.png)

같은 객체를 가리키고 있기 때문에 당연한 결과이다.

## DISTINCT
---
SQL에서 DISTINCT는 중복된 결과를 제거한다. JQPL에서는 DISTINCT가 **SQL에 DISDINCT를 추가**할 뿐만 아니라 **애플리케이션에서도 제거**해준다.

```
select distinct t
from Team t join fetch t.members
where t.name = '팀A'
```

따라서, 위의 예시처럼 중복된 Team이 2개 나오더라도 애플리케이션에서 중복된 내용을 제거해준다. 물론 중복의 제거는 애플리케이션에서 진행된다. SQL에서 진행되는 것이 아니다.

Hibernate 6 부터는 JPQL에 distinct 를 사용하지 않더라도 중복된 데이터를 제거해준다고 한다.

## 일반 Join과 차이점
---
일반적으로 JPQL에서 Join을 사용하면 회원 컬렉션도 함께 조회되지 않는다.

```
select t
from Team t join t.members m
where t.name = '팀A'
```

위 JPQL에서는 아래와 같은 쿼리를 날린다.

```sql
SELECT 
    T.*
FROM TEAM T
INNER JOIN MEMBER M ON T.ID=M.TEAM_ID
WHERE T.NAME = '팀A'
```

JPQL에서 select의 대상은 Team 뿐이다. 따라서 SQL에서 멤버에 대한 정보는 가져오지 않는다.

**JPQL은 결과를 반환할 때 연관관계까지 고려하지 않기 때문이다.**

(그래서 N+1 문제가 EAGER 설정을 해도 해결되지 않는 것이다.)

#### 출처 및 참고
---
책 - 김영한의 JPA 프로그래밍
