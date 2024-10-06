---
title: "[Spring] Component Scan과 Autowired"
date: 2024-10-06 +09:00
categories:
  - Study
  - Spring-Inflearn
tags:
  - 스프링
  - 스프링부트
---
## 🧐 Component Scan
---
기존에는 `AppConfig` 클래스를 만들어 의존관계를 직접 주입해줬다. 하지만, 빈이 수십, 수백개가 되면 설정 클래스도 커지게 되고 복잡해진다.

그래서 스프링은 설정 정보가 없어도 **자동으로 스프링 빈을 등록하는 컴포넌트 스캔**이라는 기능을 제공한다.

```java
@Configuration
@ComponentScan
public class AutoAppConfig {
}
```

위와 같이 `@ComponentScan`을 붙여주면 된다. 기존처럼 `@Bean`을 붙여 만든 메서드가 하나도 없다. 그럼 Bean 등록은 어떻게 할까?

```java
@Component 
public class MemoryMemberRepository implements MemberRepository {}
```

Bean으로 등록할 클래스에 `@Component`를 붙여주면된다. 그럼 **`@ComponentScan`의 대상이 되어 Bean으로 등록**된다.

그럼 의존관계가 있으면 어떻게 해야할까? **`@Autowired`를 이용해 해결**할 수 있다.

```java
@Component
public class MemberServiceImpl implements MemberService {
	
	private final MemberRepository memberRepository;
 
	@Autowired
	public MemberServiceImpl(MemberRepository memberRepository) {
		this.memberRepository = memberRepository;
	}
}
```

`@Autowired`를 붙이게 되면 스프링 컨테이너에서 **의존관계를 찾아 자동으로 주입**해준다. 여러 의존관계도 한번에 주입받을 수 있다.

```java
@Test
void basicSacan(){
	AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class);

	MemberService memberService = ac.getBean(MemberService.class);
	assertThat(memberService).isInstanceOf(MemberService.class);
}
```

테스트 해보면 잘 동작하는 것을 알 수 있다.

![](images/2024-10-06-Spring-Inflearn-Basic-6-3.png)

- 빨강 : 로그를 보면 컴포넌트 스캔이 잘 동작하는 것을 확인 가능
- 주황 : 싱글톤 빈으로 생성
- 초록 : 생성자를 통해서 @AutoWired로 주입 (DI)
#### 컴포넌트 스캔 동작 규칙
컴포넌트 스캔에서 빈 이름은 어떻게 정해지는 걸까? 기본적으로 **맨 앞글자를 소문자로 바꾼 클래스명**을 사용한다. 물론 직접 지정도 가능하다.
- 기본 전략 : MemberServiceImpl - > memberServiceImpl
- 직접 지정 : `@Component("memberService2")`와 같이 직접 지정 가능

## 🔎 탐색 위치와 기본 스캔 대상
---
모든 자바 클래스를 다 컴포넌트 스캔하면 시간이 오래 걸린다. 그래서 꼭 필요한 위치부터 탐색하도록 시작 위치를 지정할 수 있다.

```java
@Configuration
@ComponentScan(
        basePackages = "hello.core.member"
)
public class AutoAppConfig {
}
```

위와 같이 탐색 시작 위치를 정해줄 수 있다. 만약 **지정해주지 않으면 Configuration 클래스의 패키지가 시작위치**가 된다.

그래서 **권장하는 방법은 설정 정보 클래스를 프로젝트 최상단**에 두는 것이다.

스프링 부트의 경우에도 시작 정보인 `@SpringBootApplication`를 최상단에 둔다. (이 어노테이션 안에 `@CompoenentScan`이 포함되어있다.)

#### 컴포넌트 스캔 기본 대상
컴포넌트 스캔은 @Component 뿐만 아니라 다음과 내용도 추가로 대상에 포함한다.
- `@Component` : 컴포넌트 스캔에서 사용
- `@Controller` : 스프링 MVC 컨트롤러에서 사용
- `@Service` : 스프링 비즈니스 로직에서 사용
- `@Repository` : 스프링 데이터 접근 계층에서 사용
- `@Configuration` : 스프링 설정 정보에서 사용

해당 어노테이션에 들어가 확인해보면 모두 `@Component`가 포함되어있는 것을 볼 수 있다.

![](images/2024-10-06-Spring-Inflearn-Basic-6-4.png)

> 사실 자바 어노테이션에는 상속관계라는 것이 없다. 하지만 스프링이 지원해주는 기능이다.

컴포넌트 스캔의 용도 뿐만 아니라 다음 애노테이션이 있으면 스프링은 부가 기능을 수행한다.

- `@Controller` : 스프링 MVC 컨트롤러로 인식
- `@Repository` : 스프링 데이터 접근 계층으로 인식하고, 데이터 계층의 예외를 스프링 예외로 변환
- `@Configuration` : 앞서 보았듯이 스프링 설정 정보로 인식하고, 스프링 빈이 싱글톤을 유지하도록 추가 처리
- `@Service` : 사실 특별한 처리를 하지 않음. 대신 개발자들이 핵심 비즈니스 로직이 여기에 있겠구나 라고 비즈니스 계층을 인식하는데 도움

## 🙅🏻 필터
---
필터를 이용해서 컴포넌트 스캔 대상을 분류할 수 있다.

- includeFilters : 컴포넌트 스캔 대상을 추가로 지정
- excludeFilters : 컴포넌트 스캔에서 제외할 대상을 지정

컴포넌트 스캔 대상을 어노테이션으로 관리해보자. 아래의 어노테이션을 붙인 클래스는 컴포넌트 스캔의 대상이 되도록 할 것이다.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface MyincludeComponent {
}
```

반대로 아래의 어노테이션을 붙인 클래스는 컴포넌트 스캔의 대상에서 제외시킬 것이다.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface MyExcludeComponent {
}
```

아래와 같이 설정 정보 클래스에 필터를 등록해주면 된다.

```java
@Configuration
@ComponentScan(
	includeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyincludeComponent.class),
	excludeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyExcludeComponent.class)
)
static class ComponentFilterAppConfig{

}
```

FilterType에 `FilterType.ANNOTATION`으로 되어있다. 필터 옵션에는 아래와 같이 5가지가 있다.

- ANNOTATION: **기본값**, 애노테이션을 인식해서 동작한다.
	- org.example.SomeAnnotation
- ASSIGNABLE_TYPE: 지정한 타입과 자식 타입을 인식해서 동작한다.
	- ex) org.example.SomeClass
- ASPECTJ: AspectJ 패턴 사용
	- org.example..*Service+
- REGEX: 정규 표현식
	- org\\.example\.Default.*
- CUSTOM: TypeFilter 이라는 인터페이스를 구현해서 처리
	- org.example.MyTypeFilter

사실 필터는 잘 사용할 일이 없다. 기본 설정에 최대한 맞춰 사용하는 것을 권장한다.

## 🚨 빈 충돌
---
만약 빈 이름이 충돌하면 어떻게 될까?

두가지 상황이 있다.
- **자동 빈 등록 vs 자동 빈 등록**
- **자동 빈 등록 vs 수동 빈 등록**

만약 자동 빈 등록으로 이름이 겹친다면 **`ConflictingBeanDefinitionException` 예외가 터진다**.

그럼 수동 빈 등록과의 충돌이 일어나면 어떻게 될까?

```java
@Component
public class MemoryMemberRepository implements MemberRepository {
}
```

위와 같은 클래스를 컴포넌트 스캔을 통해 빈이 등록되었다고 생각해보자.

```java
@Configuration
public class AutoAppConfig {

    @Bean(name = "memoryMemberRepository")
    MemberRepository memberRepository(){
        return new MemoryMemberRepository();
    }
}
```

만약 똑같은 이름으로 수동으로 이름을 지정하면 어떻게 될까?

결과는, **수동 빈 등록이 오버라이딩 해버린다.** 즉, 자동 빈 등록이 무시된다. 이 때문에 설정이 꼬이기도 하고 버그가 발생한다.

그래서 **최근 스프링 부트에서는 충돌이나면 오류를 터트리도록 기본 값을 바꾸었다!**