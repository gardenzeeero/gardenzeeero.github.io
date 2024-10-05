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
## ✅ 스프링 컨테이너 생성
---
보통 `ApplicationContext`를 스프링 컨테이너라고 한다. `ApplicationContext`는 인터페이스 이고 XML기반으로 만들 수도 있고, 어노테이션 기반으로 만들 수도 있다.

> 정확히는 스프링 컨테이너는 `BeanFactory`와 `ApplcationContext`로 구분된다. 하지만, `BeanFactory`를 직접적으로 사용하는 경우는 거의 없다. 따라서, 보통 `ApplicationContext`를 스프링 컨테이너라 부른다.

어노테이션 기반으로  스프링 컨테이너를 만드는 방법은 아래와 같다.

```java
ApplicationContext applicationContext = 
			new AnnotationConfigApplicationContext(AppConfig.class)
```

## 🧐 스프링 컨테이너의 생성 과정
---
스프링 컨테이너를 생성하기 위해선 설정 정보를 건네줘야 한다.

`new AnnotationConfigApplicationContext(AppConfig.class)`와 같이 AppConfig 클래스를 설정 정보를 전달해주면 된다.

![](images/2024-09-30-Spring-Inflearn-Basic-4-1.png)

위와 같이 스프링 컨테이가 만들어지고 전달 받은 **`AppConfig` 클래스를 활용해 빈 저장소를 채운다.**

빈 이름은 **기본적으로 메서드 이름**을 사용한다. 하지만 `@Bean(name = "memberService2")` 와 같이 **직접 이름을 부여**할 수도 있다. (**빈 이름은 항상 다른 이름을 부여**해야한다. 설정에 따라 무시되거나, 덮어버리거나 오류가 발생할 수도 있다.)

![](images/2024-09-30-Spring-Inflearn-Basic-4-2.png)

이후 의존관계를 설정할 준비를 한다. 

![](images/2024-09-30-Spring-Inflearn-Basic-4-3.png)

이후 AppConfig를 참고해 **의존관계를 주입**한다.(DI) 단순히 메서드를 호출하는 것이 아닌 **싱글톤 패턴**을 사용하기 때문에 일반적인 자바 코드 실행과는 조금 다르다.

![](images/2024-09-30-Spring-Inflearn-Basic-4-4.png)

## 😎 컨테이너에서 빈 조회하기
---
우선 모든 빈에 대해서 조회해보자. 설정 정보를 전달해주는 코드는 생략하도록 하겠다. 

```java
@Test
@DisplayName("모든 빈 출력하기")
void finedAllBean(){

	String[] beanDefinitionNames = ac.getBeanDefinitionNames();
	
	//getBeanDefinitionNames: ApplicationContext(또는 하위 클래스)에 등록된 모든 빈(Bean)의 이름을 문자열 배열로 반환
	for (String beanDefinitionName : beanDefinitionNames) {
		Object bean = ac.getBean(beanDefinitionName);
		System.out.println("name = " + beanDefinitionName + " object = " + bean);
	}
}
```

모든 빈의 이름(`beanDefinitionNames`)를 가져와 빈의 이름으로 조회하는 방식이다.

- `ac.getBeanDefinitionNames()` : 스프링에 등록된 **모든 빈 이름을 조회**한다. 
- `ac.getBean()` : 빈 이름으로 **빈(Bean) 객체(인스턴스)를 조회**한다.

스프링 빈에는 종류가 있다. `getRole()`을 통해 구분할 수 있다.
- ROLE_APPLICATION : 일반적으로 **사용자가 정의**한 빈 
- ROLE_INFRASTRUCTURE : **스프링이 내부**에서 사용하는 빈

## 🫛 빈 조회하기
---
기본적으로 두가지 방법으로 조회할 수 있다.
1. ac.getBean(빈이름, 타입) 
2. ac.getBean(타입)

만약 조회하는 빈이 없다면 아래와 같은 예외가 발생한다.

> `NoSuchBeanDefinitionException: No bean named 'xxxxx' available`

코드를 통해 예시를 살펴보자.

```java
@Test
@DisplayName("빈 이름으로 조회")
void findBeanByName(){
	MemberService memberService = ac.getBean("memberService", MemberService.class);
	assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
}

@Test
@DisplayName("이름 없이 타입으로만 조회")
void findBeanByType(){
	MemberService memberService = ac.getBean(MemberService.class); 
	assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
	//인터페이스로 조회를 하면 인터페이스 구현체가 대상이 됨
}

@Test
@DisplayName("구체 타입으로 조회")
void findBeanByType2(){
	MemberServiceImpl memberService = ac.getBean("memberService",MemberServiceImpl.class);
	assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
	// 인스턴스 타입을 보고 결정하기 때문에 인터페이스가 아닌 구체 타입으로 적어도 된다.
	// But, 이는 구현에 의존하는 것보다 역할에 의존하는게 좋기 때문에 좋은 코드는 아니다.
}

@Test
@DisplayName("빈 이름으로 조회 실패")
void findBeanByNameX(){
	assertThrows(NoSuchBeanDefinitionException.class,
			() -> ac.getBean("xxxxx", MemberService.class));
	
	// 우측 ac.getBean("xxxxx", MemberService.class)을 실행했을 때
	// 좌측 NoSuchBeanDefinitionException 예외가 발생하면 성공
}
```

### 만약 동일한 타입이 둘 이상이라면?
---
`ac.getBeansOfType()` 메서드를 사용해 타입을 이용한 조회를 했다고 생각해보자. 

만약 아래와 같이 설정 정보가 구성되어있다면 어떻게 될까?

```java
@Test
@DisplayName("타입으로 조회시 같은 타입이 둘 이상 있으면, 중복 오류가 발생한다.")
void findBeanByTypeDuplicate(){
	MemberRepository bean = ac.getBean(MemberRepository.class);
}

@Configuration
static class SameBeanConfig{
	@Bean
	public MemberRepository memberRepository1(){
		return new MemoryMemberRepository();
	}
	@Bean
	public MemberRepository memberRepository2(){
		return new MemoryMemberRepository();
	}
}
```

`memberRepository1`과 `memberRepository2`가 **모두 `MemberRepository`타입**이다. 따라서, 타입으로 조회했을 경우, **두개의 빈이 조회**된다. 이때는 `NoUniqueBeanDefinitionException`이 발생한다.

이때는 **빈 이름이 지정해서 조회**하면 된다. `memberRepository1`를 조회하고 싶다면 아래와 같이 하면 된다.

```java
@Test
@DisplayName("타입으로 조회시 같은 타입이 둘 이상 있으면, 빈 이름을 지정하면 된다")
void findBeanByName(){

	MemberRepository memberRepository = ac.getBean("memberRepository1", MemberRepository.class);
	assertThat(memberRepository).isInstanceOf(MemberRepository.class);
}
```

### 상속관계
---
만약 아래와 같은 상속관계에 있다면 어떻게 조회될까?

![](images/2024-09-30-Spring-Inflearn-Basic-4-6.png)

1번 타입으로 조회하게 되면 1~7번까지 모두 조회된다. 만약 Object 타입으로 조회한다면 모든 빈이 조회된다. 당연히 해당 타입들의 빈이 여러개라면  `NoUniqueBeanDefinitionException`이 발생한다. 

물론, 해당 타입의 빈을 모두 들고와 작업을 할 생각이라면 아래와 같이 들고 올 수도 있다.

```java
 @Test
@DisplayName("부모 타입으로 조회 시 자식이 둘 이상 있으면, 중복 오류가 발생한다")
void findBeanByParentTypeDuplicate() {
	Assertions.assertThrows(NoUniqueBeanDefinitionException.class,
			() -> ac.getBean(DiscountPolicy.class));
}

@Test
@DisplayName("부모 타입으로 조회 시 자식이 둘 이상 있으면, 빈 이름을 지정하면 된다.")
// 인터페이스로 조회
void findBeanByParentTypeBeanName() {
	DiscountPolicy rateDiscountPolicy = ac.getBean("rateDiscountPolicy", DiscountPolicy.class);
	assertThat(rateDiscountPolicy).isInstanceOf(RateDiscountPolicy.class);
}
@Test
@DisplayName("특정 하위 타입으로 조회.")
// 구체적인 타입으로 조회
void findBeanBySubType() {
	RateDiscountPolicy bean = ac.getBean(RateDiscountPolicy.class);
	assertThat(bean).isInstanceOf(RateDiscountPolicy.class);
}

@Test
@DisplayName("부모 타입으로 모두 조회.")
void findAllBeanByParentType() {
	Map<String, DiscountPolicy> beansOfType = ac.getBeansOfType(DiscountPolicy.class);
	assertThat(beansOfType.size()).isEqualTo(2);
	
	for (String key : beansOfType.keySet()) {
		System.out.println("key = " + key + " value = " + beansOfType.get(key));
	}
}

@Test
@DisplayName("부모 타입으로 모두 조회하기 - Object.")
void findAllBeanByObjectType() {
	Map<String, Object> beansOfType = ac.getBeansOfType(Object.class);
	for (String key : beansOfType.keySet()) {
		System.out.println("key = " + key + " value = " + beansOfType.get(key));
	}
}

@Configuration
static class TestConfig {
	@Bean
	public DiscountPolicy rateDiscountPolicy() {
		return new RateDiscountPolicy();
	}
	// public 다음 리턴 타입을 구현클래스가 아닌 인터페이스로 적는 이유
	// 개발 혹은 설계 시 역할과 구현을 쪼갰던 것과 동일 개념
	// 역할 : DiscountPolicy / 구현 : RateDiscountPolicy
	// 다른 곳에서 의존관계 주입할 때 더욱 유연하게 가능하기 때문이다.

	@Bean
	public DiscountPolicy fixDiscountPolicy() {
		return new FixDiscountPolicy();
	}
}
```

## BeanFactory와 ApplicationContext
---
우리가 사용했던 스프링 컨테이너는 아래와 같은 구조를 가진다.

![](images/2024-09-30-Spring-Inflearn-Basic-4-7.png)

이 때까지 사용했던 대부분의 기능은 사실 `BeanFactory`가 제공하는 기능이다.

`ApplicationContext`는 빈을 조회하는 기능 뿐만 아니라 다른 부가기능도 조회한다. 따라서, 아래와 같이 다양한 인터페이스를 구현한다.

![](images/2024-09-30-Spring-Inflearn-Basic-4-8.png)

1. 메시지소스를 활용한 국제화 기능 
	- 예를 들어서 한국에서 들어오면 한국어로, 영어권에서 들어오면 영어로 출력 
2. 환경변수
	- 로컬, 개발, 운영등을 구분해서 처리 
3. 애플리케이션 이벤트 
	- 이벤트를 발행하고 구독하는 모델을 편리하게 지원 
4. 편리한 리소스 조회 
	- 파일, 클래스패스, 외부 등에서 리소스를 편리하게 조회

스프링 컨테이너는 어노테이션 기반이 아닌 xml 등 다른 방식으로 설정할 수도 있는데 이는 생략한다. 결국 형식만 다르지 비슷하다.

하지만, 원리정도는 알자

## BeanDefinition
---
스프링이 이렇게 다양한 설정 방식을 지원하는 방법은 `BeanDefinition`과 관련되어있다.

`BeanDefinition`은 빈 설정 메타정보이다. 스프링 컨테이너는 이 메타 정보를 기반으로 스프링 빈을 생성한다.

![](images/2024-09-30-Spring-Inflearn-Basic-4-9.png)

- `AnnotationConfigApplicationContext` 는 `AnnotatedBeanDefinitionReader`를 사용해서 `AppConfig.class` 를 읽고 `BeanDefinition`을 생성한다. 
- `GenericXmlApplicationContext` 는 `XmlBeanDefinitionReader`를 사용해서 `appConfig.xml` 설정 정보를 읽고 `BeanDefinition` 을 생성한다. 
- 새로운 형식의 설정 정보가 추가되면, `XxxBeanDefinitionReader`를 만들어서 `BeanDefinition`을 생성하면 된다.

`BeanDefinition` 예시는 아래와 같다.
- BeanClassName: 생성할 빈의 클래스 명(자바 설정 처럼 팩토리 역할의 빈을 사용하면 없음)
- factoryBeanName: 팩토리 역할의 빈을 사용할 경우 이름, 예) appConfig
- factoryMethodName: 빈을 생성할 팩토리 메서드 지정, 예) memberService
- **Scope: 싱글톤(기본값)**, 잘 기억해두자
- lazyInit: 스프링 컨테이너를 생성할 때 빈을 생성하는 것이 아니라, 실제 빈을 사용할 때 까지 최대한 생성을 지연 처리 하는지 여부
- InitMethodName: 빈을 생성하고, 의존관계를 적용한 뒤에 호출되는 초기화 메서드 명
- DestroyMethodName: 빈의 생명주기가 끝나서 제거하기 직전에 호출되는 메서드 명
- Constructor arguments, Properties: 의존관계 주입에서 사용한다. (자바 설정 처럼 팩토리 역할의 빈을 사용하면 없음)

사실 실무에서 `BeanDefinition`을 다룰 일은 없고 오픈 소스에서 보일때가 있다.

## Reference
---
해당 글은 김영한님의 Spring 강의를 듣고 한차시를 요약적으로 정리한 것이다. 기술의 단점을 보완하고 발전시키는 방향으로 진행된다. 따라서, 아래의 글이 최적의 방법이 아닐 수 있다. 기술 등장 배경과 이점에 집중해서 보자.