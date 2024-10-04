---
title: "[Spring JPA] @Modifying 어노테이션"
date: 2024-09-12 +09:00
categories:
  - Study
  - Spring
tags:
  - 스프링
  - 스프링부트
  - JPA
---
## 😩 알게된 계기
---
게시글 수정 후 수정 여부가 적용이 안되다고하였다. 우리는 JPA의 Auditing 기능을 사용했기 때문에 엔티티를 수정하면 당연히 적용이 되어야한다.

하지만 문제는 우리가 JPA의 Dirty Chekcing을 이용한 것이 아니라 `@Modifying`과 `@Quary`를 이용한 벌크 연산을 했다는 것이다.

```java
@Modifying
@Query("update Comment c set c.content = :content where c.id = :id")
void updateById(@Param("id") Long id, @Param("content") String content);
```

사실 이건 `@Query`에 대한 이해도 부족이었다. 나는 값을 업데이트 하는데 조회쿼리 1번, 업데이트 쿼리 한번이 나가는 것이 마음에 들지 않았다. `@Query`는 직접 update 쿼리를 만들어서 보내는 형태이다. 아래와 같은 상황이 생긴 것이다.

1. `@Query`를 통해 직접 업데이트 메서드를 날림
2. 따라서, 영속성 컨텍스트의 1차 캐시에 엔티티가 저장되지 않음
3. 1차 캐시에 정보가 없기때문에 JPA Auditing의 대상이 되지 않음

직접 lastModifiedDate를 수정하는 방법도 있었지만 이는 유지보수 측면에서 자동화된 Auditing과 충돌을 일으킬 수 있다고 생각했다. 그래서 조회 후 업데이트 하는 방식으로 결정했다.

사실 `@Modifying` 어노테이션 때문인줄 알고 막 찾아보았다. 그 중 새롭게 알게된 사실이 많아 정리해보았다.

## 🧐 Modifying 어노테이션
---
JPA에서 기본적으로 `@Query`를 이용해 만들어낸 변경이 일어나는 쿼리(INSERT, DELETE, UPDATE)의 경우 `@Modifying`을 붙이도록 강제하고 있다. 만약 붙이지 않는다면 **QueryExecutionRequestException**가 발생하게 된다.

해당 어노테이션에는 두가지 옵션이 존재한다.

![](images/2024-10-04-Spring-Modifying-Auditing-2.png)
#### flushAutomatically
해당 옵션은 `@Query`를 이용한 쿼리 메서드를 사용할 때, **쿼리를 실행하기 전 영속성 컨텍스트의 변경 사항을 DB에 flush 할 것인지를 결정**한다. 즉, 쓰기지연(write-behind) 저장소에 있는 쿼리들을 모두 날리는 것이다.

기본값은 false이고 true로 설정시 동작한다.
#### clearAutomatically
해당 옵션은 쿼리 실행전에 영속성 컨텍스트를 비울 것인지 결정한다.

만약 true로 설정 한다면 업데이트 후 조회할 때 달라진다.

1. 엔티티 조회
2. 업데이트 쿼리
3. 엔티티 다시 조회

만약 false라면 1번에서 1차 캐시에 엔티티가 저장된다. 따라서 3번에서 조회할 때 1번의 엔티티를 가져오게 된다. 즉, **업데이트가 되지 않은 엔티티를 조회하게 되는 것**이다. 예시를 보자

```java
void update() {
	Comment comment = new Comment(); // Comment - Transient 상태
	comment.setContent("before modifying");
	commentRepository.save(comment); // article - Persistent 상태
	
	commentRepository.updateById(1L, "after"); // UPDATE SQL

	commentRepository.findById(1L);	//만약 옵션이 true라면 여기서 다시 조회 쿼리
}
```

그럼 무조건 true로 설정하는 것이 좋은건가? 그건 상황에 따라 다르다. 업데이트 이후에 조회할 일이 없다면 굳이 true로 해서 영속성 컨텍스트를 비울 필요까진 없다. 다른 엔티티까지 비워지기 때문에 불필요한 조회가 여러번 일어날 수도 있다.

만약 재조회할 일이 있다면 굳이 `@Query`를 사용하지 않고, 조회 후 Dirty Cheking을 통해 업데이트하는 것이 더 좋은 방법이다.

## 😮 Hibernate와의 조합
---
`flushAutomatically` 옵션을 false이면 업데이트 쿼리 이전에 어떤 쿼리가 나가선 안된다.

**하지만 실제론 그렇지 않다.**

그 이유는 Hibernate의 **FlushModeType** 때문이다. 해당 타입은 어떤 상황에서 영속성 컨텍스트를 Flush 할 지 정해준다. 해당 타입이 AUTO로 설정 되어있다면 위와 같이 동작한다. default가 AUTO이기 때문에 별다른 설정이 없다면 flushAutomatically가 false라도 true처럼 동작한다.

FlushModeType에 대해서는 따로 잘 정리해둔 [블로그](https://keencho.github.io/posts/jpa-flush/)를 참고해보면 될 것 같다.

FlushModeType가 AUTO를 간단히 설명하자면 아래와 같다.
- 트랜잭션이 커밋되기 이전
- 영속성 컨텍스트에 보류 중인 변경이 포함된 데이터베이스 테이블을 조회하는 쿼리를 실행하기 전