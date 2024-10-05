---
title: "[Spring] 싱글톤 패턴과 스프링 컨테이너"
date: 2024-10-05 +09:00
categories:
  - Study
  - Spring-Inflearn
tags:
  - 스프링
  - 스프링부트
---
## 🤮 순수한 DI 컨테이너
---
순수하게 Configuration 클래스를 사용해서 DI를 해준다면 어떻게 될까?

```java
@Test
@DisplayName("스프링 없는 순수한 DI 컨테이너")
void pureContainer() {
	AppConfig appConfig = new AppConfig();
	//1. 조회 : 호출할 때 마다 객체를 생성
	MemberService memberService = appConfig.memberService();

	//2. 조회 : 호출할 때 마다 객체를 생성
	MemberService memberService2 = appConfig.memberService();

	// nenberService != memberService2
	assertThat(memberService).isNotSameAs(memberService2);
}
```

![](images/2024-10-05-Spring-Inflearn-Basic-5-2.png)

위와 같이 매번 **요청할 때 마다 다른 객체를 생성**한다. 초당 100개의 요청이 들어오면 100개의 객체를 생성하게 되는 것이다. 이는 **메모리 낭비**가 매우 심하다.

## 🙁 싱글톤 패턴
---
싱글톤 패턴이란 클래스의 **인스턴스가 딱 1개만 생성되도록 하는 디자인 패턴**이다.

이렇게 하기 위해선 클라이언트가 임의로 객체를 생성하도록 하면 안된다. 따라서 **private 생성자를 이용해 new 키워드를 사용하지 못하도록 막아야한다.** 간단하게는 아래와 같이 만들 수 있다.

```java
public class SingletonService {

    private static final SingletonService instatnce = new SingletonService();

    public static SingletonService getInstance() {
        return instatnce;
    }
    
    private SingletonService() {
    }

    public void logic(){
        System.out.println("싱글톤 객체 로직 호출");
    }
}
```

1. static 영역에 객체 **instance를 미리 하나 생성**해서 올려둔다.
2. 이 객체 인스턴스가 필요하면 오직 **getInstance() 메서드를 통해서만 조회**할 수 있다. 
	- 이 메서드를 호출하면 항상 같은 인스턴스를 반환한다.
3. **생성자를 private**으로 막아서 혹시라도 외부에서 **new 키워드로 인스턴스가 생성되는 것을 막는다.**

테스트를 해보자

```java
@Test
@DisplayName("싱글톤 패턴을 사용한 객체 사용")
void singleTonServiceTest(){

	SingletonService singletonService1 = SingletonService.getInstance();
	SingletonService singletonService2 = SingletonService.getInstance();

	assertThat(singletonService1).isSameAs(singletonService2);
}
```

> **isSameAs** : 객체의 내용(content)이 같은지를 비교 -> 메모리 상의 주소를 비교하므로, 객체의 실제 내용이 동일하지 않더라도 같은 객체로 간주    
> **isEqualTo** : 객체의 내용(content)이 같은지를 비교 -> 객체의 내용이 동일하다면, 객체가 서로 다른 메모리 위치에 있더라도 같은 것으로 간주

#### 싱글톤 패턴의 문제점
1. 싱글톤 패턴을 구현하는 **코드 자체**가 많이 들어간다.
2. 의존관계상 클라이언트가 구체 클래스에 의존한다. **DIP를 위반**한다. (SingletonService가 내부에 구체 클래스를 만들어둠)
3. 클라이언트가 구체 클래스에 의존해서 **OCP 원칙을 위반**할 가능성이 높다.
4. **테스트하기 어렵다.**
5. **내부 속성을 변경하거나 초기화** 하기 어렵다.
6. private 생성자로 **자식 클래스를 만들기 어렵다**. (private 생성자만 있으면 상속이 안된다. public이 있는 경우엔 가능하지만 싱글톤에 위배된다.)

결론적으로 유연성이 떨어진다. 따라서, 안티패턴으로 불리기도 한다.

## 😃 싱글톤 컨테이너
---
우리가 아는 스프링 컨테이너도 싱글톤 컨테이너의 일종이다. 싱글톤 컨테이너는 **인스턴스를 싱글톤으로 관리한다.** 즉, 1개만 만들어서 사용한다. 이를 스프링에서는 빈(Bean)이라고 한다.

![](images/2024-10-05-Spring-Inflearn-Basic-5-1.png)

**스프링 컨테이너는 싱글턴 패턴을 적용하지 않아도, 객체 인스턴스를 싱글톤으로 관리한다.**

따라서, **싱글톤 패턴의 모든 단점을 해결**하고 객체를 싱글톤으로 유지할 수 있다.
- 추가적인 자바코드 삭제
- DIP, OCP 준수, 테스트 용이, private 생성자로부터의 자유

```java
@Test
@DisplayName("스프링 컨테이너와 싱글톤")
void springContainer(){

	ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
	
	MemberService memberService1 = ac.getBean("memberService", MemberService.class);
	MemberService memberService2 = ac.getBean("memberService", MemberService.class);

	assertThat(memberService1).isSameAs(memberService2);
}
```

싱글톤 패턴을 사용하지 않았지만 싱글톤 패턴을 이용한 것과 마찬가지로 테스트가 통과된다.

> 모든 빈이 싱글톤은 아니지만 95% 이상이 모두 싱글톤으로 사용된다.

## 🚨 싱글톤 방식의 주의점
---
싱글톤 방식은 알다시피 하나의 인스턴스를 여러 클라이언트가 공유해서 사용한다. 따라서, 싱글톤 객체는 stateful 하게 설계하면 안된다. stateful 하다는 것은 객체 내부에 상태(state)를 가지고 있다는 말이다. 

**즉, 객체가 어떤 값이나 정보를 필드에 저장해두고 해당 정보에 따라 동작이 바뀌는 것이다. 특정 클라이언트에 의존적이어도 안된다.** (하지만, 만약 필드 값이 변경될 일이 없는 불변이라면 stateful 이라고 보지 않는다.)

싱글톤은 왜 stateful하면 안되는 걸까? 그 이유는 쓰레드와 관련되어 있는데 아래의 예시를 보고 알 수 있다.

```java
public class StatefulService {
    private int price; //상태(state)를 유지하는 필드

    public void order(String name, int price) {
        this.price = price; //여기서 문제!
    }

    public int getPrice(){
        return price;
    }
}
```

위와 같은 서비스가 있을 때, 여러명의 사용자가 동시에 주문을 한다고 생각해보자.

```java
@Test
void statefulServiceSingeton(){
	ApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);
	StatefulService statefulService1 = ac.getBean(StatefulService.class);
	StatefulService statefulService2 = ac.getBean(StatefulService.class);

	//ThreadA: A사용자가 10000원 주문
	statefulService1.order("userA", 10000);
	//ThreadB: B사용자가 20000원 주문
	statefulService2.order("userB", 20000);

	//ThreadA: 사용자A가 주문 금액을 조회
	int price = statefulService1.getPrice();

	Assertions.assertThat(statefulService1.getPrice()).isEqualTo(20000);
}
```

Thread A에서 A 사용자가 10000원 주문 / Thread A에서 주문 금액을 조회

이 사이에 Thread B가 끼어들어 20000원을 주문한다면 Thread A에서 주문 금액을 조회하는 시점에는 price 필드의 값이 20000원이 되어있는 것이다. **A는 10000원을 주문했지만 20000원이라는 금액을 보는 것이다!!**

따라서 공유되지 않는 지역변수, 파라미터, ThreadLocal 등을 사용해야한다. 즉, stateless 하게 설계하자!

## 😎 @Configuration과 싱글톤
---
이 때까지 AppConfig를 보며 이상한 점을 느꼈어야 했다.

```java
@Configuration
public class AppConfig {

    @Bean
    public MemberService memberService(){
        return new MemberServiceImpl(memberRepository());
    }
    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
    @Bean
    public OrderService orderService(){
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }
    @Bean
    public DiscountPolicy discountPolicy(){
        //return new FixDiscountPolicy();
        return new RateDiscountPolicy();
    }
}
```

위의 코드에서 `MemberService`와 `OrderService`가 **모두 `memberRepository()`를 호출**한다. 그럼 다른 인스턴스를 반환 받는거 아닌가? 라는 생각이 들어야한다.

진짜로 그런지 테스트해보자

```java
@Test
void configurationTest(){
	AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

	MemberServiceImpl memberService = ac.getBean("memberService", MemberServiceImpl.class);
	OrderServiceImpl orderService = ac.getBean("orderService", OrderServiceImpl.class);
	MemoryMemberRepository memberRepository = ac.getBean("memberRepository", MemoryMemberRepository.class);

	MemberRepository memberRepository1 = memberService.getMemberRepository();
	MemberRepository memberRepository2 = orderService.getMemberRepository();

	assertThat(memberService.getMemberRepository()).isEqualTo(memberRepository);
	assertThat(orderService.getMemberRepository()).isEqualTo(memberRepository);
}
```

테스트가 통과한다. **즉, 모두 같은 인스턴스를 사용한다.** 어찌보면 스프링 컨테이너가 싱글톤을 보장해준다 했으니 당연히 이런 결과가 나와야한다. 그럼 `memberRespository()`가 한번만 호출되는 걸까?

Configuration에 log를 남겨 확인해보면 **한번만 호출되는 것이 맞다!**

어떻게 그렇게 되는걸까?

## 🪄 @Configuration과 바이트코드 조작
---
상식으로는 3번 호출되는 것이 맞다.
1. `@Bean`이 붙은 메서드를 통해 1번
2. 각각의 서비스가 호출해서 총 2번

스프링은 해당 자바 코드를 변경할 수는 없다. 하지만, 싱글톤은 보장해줘야한다.

그래서 **스프링은 바이트 코드 조작 라이브러리를 사용한다!**

```java
 @Test
void configurationDeep(){
	//파라미터로 넘겨준 AppConfig도 Bean으로 등록된다.
	AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
	
	AppConfig bean = ac.getBean(AppConfig.class);

	//getClass :  객체가 속한 클래스의 정보를 반환
	System.out.println("bean = " + bean.getClass());
}
```

해당 출력값을 보면 요상한 점을 발견할 수 있다.

`bean = class hello.core.AppConfig$$EnhancerBySpringCGLIB$$bd479d70`

오잉? 일반적인 클래스라면 `class hello.core.AppConfig` 라고 출력되어야한다.

![](images/2024-10-05-Spring-Inflearn-Basic-5-3.png)

이건 스프링이 CGLIB 라는 바이트코드 조작 라이브러리를 사용해 **AppConfig를 상속받은 임의의 클래스를 만들고 빈으로 등록한 것이다.**

내부적으로는 '이미 빈으로 등록된 빈은 생성하지 않고 반환한다'는 로직으로 동작한다.

#### Configuration없이 Bean만 붙인다면?
만약 `@Configuration`없이 `@Bean`만 붙인다면 어떻게 될까?

`AppConfig bean = ac.getBean(AppConfig.class);`를 통해 bean 호출해보면 알 수 있다. CGLIB로 조작한 클래스가 아닌 `class hello.core.AppConfig`라는 **순수한 클래스가 나온다.**

또한, `memberRepository()`도 3번 호출된다. 각각의 Service에도 다른 memberRepository가 들어간다. 

**즉, 싱글톤이 보장되지 않는다.**

이래서 스프링의 설정 정보에는 꼭 `@Configuration`을 붙여주자. 그래야 스프링 컨테이너가 싱글톤을 보장할 수 있다.

## Reference
---
해당 글은 김영한님의 Spring 강의를 듣고 한차시를 요약적으로 정리한 것이다. 기술의 단점을 보완하고 발전시키는 방향으로 진행된다. 따라서, 아래의 글이 최적의 방법이 아닐 수 있다. 기술 등장 배경과 이점에 집중해서 보자.