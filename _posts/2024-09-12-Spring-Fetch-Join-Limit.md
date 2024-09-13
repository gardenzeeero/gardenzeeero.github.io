---
title: "[Spring JPA] Fetch Join 사용시 페이징 처리"
date: 2024-09-12 +09:00
categories:
  - Study
  - Spring
tags:
  - 스프링
  - 스프링부트
  - JPA
---
저번 글에서 Fetch Join에 대해 알아봤다.

하지만 Fetch Join에는 해결하지 못하는 다양한 문제들이 있다. 이에 대해 알아보자.

## 페이징 처리
---
페이징 처리는 모든 데이터를 가져 오는 것이 아닌 내가 원하는 만큼만 가져오기 위해 사용한다. 조회 쿼리를 실제로 보면 limit을 사용한다.

하지만, fetch join과 함께 사용하면 아래와 같은 경고를 출력한다.

```
HHH000104: firstResult/maxResults specified with collection fetch; applying in memory!
```

해당 로그는 쿼리의 결과물을 **모두 메모리에 올린 후 페이징 처리를 수행**했다는 의미이다.

해당 오류가 뜨는 이유는 SQL을 확인해보면 알 수 있다. 일반적인 페이징 처리와는 다르게 **limit 키워드를 사용하지 않는다.** 즉, 모든 데이터를 들고와서 애플리케이션단에서 페이징 처리를 하는 것이다. 이는 페이징의 성능상 이점을 잃어버리는 것이다.

물론 개발자가 보는 결과는 **페이징 처리가 잘된 모습**이다.

대량의 데이터를 처리하는 스프링 배치 환경에서는 이러한 점들이 더욱 중요하다. 그래서 Hibernate 5.2.13 버전 부터 `fail_on_pagination_over_collection_fetch` 옵션을 지원하기 시작했다.

application.properties에서 아래의 코드를 추가해 활성화 시킬 수 있다.

```
spring.jpa.properties.hibernate.query.fail_on_pagination_over_collection_fetch=true
```

해당 옵션을 키면 **컬렉션 데이터를 메모리 상에서 페이징으로 가져오려고 할때 에러를 발생**한다.

#### 출처 및 참고
---
책 - 김영한의 JPA 프로그래밍
