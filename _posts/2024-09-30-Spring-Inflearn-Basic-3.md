---
title: "[Spring] Configuration의 등장"
date: 2024-09-29 +09:00
categories:
  - Study
  - Spring-Inflearn
tags:
  - 스프링
  - 스프링부트
---
해당 글은 김영한님의 Spring 강의를 듣고 한차시를 요약적으로 정리한 것이다. 기술의 단점을 보완하고 발전시키는 방향으로 진행된다. 따라서, 아래의 글이 최적의 방법이 아닐 수 있다. 기술 등장 배경과 이점에 집중해보자.

## 🤔 Configuration를 쓰기전에는?
---
기존에는 아래와 같은 방식으로 설계되어있었다.
```java
public class OrderServiceImpl implements OrderService {
	
	private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
}
```

비즈니스 요구사항이 변경되ㄴ어 FixDiscountPolicy를 RateDiscountPoliicy로 수정한다고 생각해보자. 해당 코드는 아래와 같이 변경되어야한다.
```java
public class OrderServiceImpl implements OrderService {

	private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
}
```

잘 바꿨다고 생각할 수도 있지만 이는 **OCP와 DIP를 위반**한 것이다. 현재 OrderServiceImpl의 직접적인 수정이 이루어졌기때문이다. 

## 😮 어떻게 해결할까?
---
인터페이스에만 의존하도록 하면 된다.
```java
public class OrderServiceImpl implements OrderService {
	 //private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
	 private DiscountPolicy discountPolicy;
}
```

지금 상태로썬 구현체가 없기때문에 실행하려면 NPE가 발생한다. 따라서 DiscountPolicy의 구현객체를 생성하고 주입해줘야한다.

## AppConfig의 등장
---
구현 객체를 생성하고 연결하는 별로의 클래스를 두는 것이다. 우선 각 클래스에 생성자를 만들어주면 된다. 그리고 애플리케이션의 전체 동작 방식을 구성(config)하는 AppConfig를 아래와 같이 만들어주면 된다.
```java
public class AppConfig {

    public MemberService memberService(){
        return new MemberServiceImpl(new MemoryMemberRepository()); //생성자 주입
    }

    public OrderService orderService(){
        return new OrderServiceImpl(new MemoryMemberRepository(), new FixDiscountPolicy());
    }
}
```

> 이러면 Repository가 다른 인스턴스라 연결이 안되는거 아니야? 라는 생각을 할 수 있지만 내부적으로 static HashMap을 쓰기 때문에 문제없다. (Repository 클래스는 데이터에 접근하기 위한 인터페이스일 뿐이다.)

OrderServiceImpl은 인터페이스만 의존하면 된다. **어떤 객체가 들어올지는 외부에서 결정**된다. 즉, AppConfig가 결정하는 것이다. OrderServiceImpl은 실행에만 집중하면 된다. OrderServiceImpl의 입장에서 의존관계를 마치 외부에서 주입해주는 것 같다고 해서 **DI(Dependency Injection)**이라고 한다.

이로써 **DIP가 해결**된다.

### 테스트
---
테스트는 아래와 같이 AppConfig에게 memberService를 요청하여 수행하면 된다.

```java
public class MemberServiceTest {

    MemberService memberService;

    @BeforeEach
    public void beforeEach(){
        //@BeforeEach: 각 테스트 메서드가 실행되기 전에 공통으로 수행되는 설정 작업
        AppConfig appConfig = new AppConfig();
        memberService = appConfig.memberService();
    }
    @Test
    void join(){
        //given
        Member member = new Member(1L, "memberA", Grade.VIP);

        //when
        memberService.join(member);
        Member findMember = memberService.findMember(1L);

        //then
        assertThat(member).isEqualTo(findMember);
    }
}
```

## 🤪 AppConfig 리팩토링과 활용
---
현재 AppConfig에는 **중복된 부분이** 많다.

```java
public class AppConfig {

    public MemberService memberService(){
        return new MemberServiceImpl(new MemoryMemberRepository()); //생성자 주입
    }

    public OrderService orderService(){
        return new OrderServiceImpl(new MemoryMemberRepository(), new FixDiscountPolicy());
    }
}
```

만약 MemberRepository의 구현체가 달라진다거나 DiscountPolicy의 구현체가 달라진다면 **모든 매개변수를 수정**해줘야한다. 따라서 아래와 같이 리팩토링 하면된다.

```java
public class AppConfig {

    public MemberService memberService(){
        return new MemberServiceImpl(memberRepository());
    }

    private MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    public OrderService orderService(){
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }

    public DiscountPolicy discountPolicy(){
        return new FixDiscountPolicy();
    }
}
```

MemoryMemberRepository를 다른 구현체로 변경하려면 **memberRepository 메서드만 변경**해주면 된다. 또한, 역할과 구현 클래스가 한눈에 들어온다.

FixDiscountPolicy를 RateDiscountPolicy로 변경한다면 `discountPolicy()`를 아래와 같이 변경해주기만 하면 된다.

```java
public DiscountPolicy discountPolicy(){
	return new RateDiscountPolicy();
}
```

**딱 한줄 바꾸어주었는데 할인 정책이 바뀌었다! 다른 클라이언트 코드는 하나도 변경하지 않았다!**

즉, 소프트웨어 요소를 새롭게 확장해도 사용 영역의 변경은 닫혀있다. 이로써 OCP를 지킬 수 있게 되었다.

## 🧐 제어의 역전 IOC(Inversion of Control)
---
이처럼 프로그램의 흐름에 대한 권한은 모두 AppConfig가 가지고 있다. 어떤 할인 정책을 설정할 것인지, 어떤 OrderService 구현체를 사용할 것인지 모두 AppConfig가 결정한다. OrderServiceImpl은 그저 자신의 로직을 수행하기만 할 뿐이다.

이렇게 프로그램의 흐름을 직접 제어하는 것이 아니라 외부에서 관리하는 것을 IoC(제어의 역전)이라고 한다.

#### 프레임워크 vs 라이브러리
- 프레임워크가 내가 작성한 코드를 제어하고, 대신 실행하면 그건 프레임워크다. (JUnit5)
- 내가 작성한 코드가 직접 제어의 흐름을 담당하면 그건 라이브러리다.

#### IoC 컨테이너, DI 컨테이너
AppConfig 처럼 객체를 생성하고 관리하면서 의존관계를 연결해주는 것을 **IoC 컨테이너 또는 DI 컨테이너** 라고 한다.

보통은 의존관계 주입에 초점을 맞추어 DI 컨테이너라고 한다. (어셈블러, 오브젝트 팩토리 라고 부르기도한다.)

## ![](images/2024-09-30-Spring-Inflearn-Basic-3-1.png) 스프링 컨테이너
---
스프링에서는 AppConfig에서 생성된 객체들을 관리해준다. 이러한 객체들을 **Bean**이라고 하고 이러한 Bean들을 관리해주는 공간을 **스프링 컨테이너**라고 한다. `ApplicationContext`라는 클래스를 통해 접근할 수 있다. 

스프링 컨테이너는 `@Configuration`이 붙은 AppConfig를 구성정보로 사용한다. 그리고 `@Bean`이 붙은 메서드를 모두 호출한다. 그 후 반환된 객체들을 모두 스프링 컨테이너에 등록한다. (`@Bean`이 붙은 메서드 명을 스프링 빈의 이름으로 사용한다.)

앞선 테스트에서 볼 수 있듯이 OrderServiceImpl을 얻기 위해서 개발자가 `AppConfig.orderService()`를 실행했다. 하지만 이제는 `application.getBean()` 메서드를 사용해 찾을 수 있다.

```java
public static void main(String[] args) {
//        AppConfig appConfig = new AppConfig();
//        MemberService memberService = appConfig.memberService();
//        OrderService orderService = appConfig.orderService();

        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);

        MemberService memberService = applicationContext.getBean("memberService", MemberService.class);
        
        OrderService orderService = applicationContext.getBean("orderService", OrderService.class);

        Long memberId = 1L;
        Member member = new Member(memberId, "memberA", Grade.VIP);
        memberService.join(member);

        Order order = orderService.createOrder(memberId, "itemA", 20000);
    }
```