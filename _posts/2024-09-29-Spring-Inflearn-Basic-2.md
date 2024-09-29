---
title: "[김영한 Spring] 비즈니스 로직 설계"
date: 2024-09-29 +09:00
categories:
  - Study
  - Spring-Inflearn
tags:
  - 스프링
  - 스프링부트
---
## 비즈니스 요구사항 설계
---
- 회원
	- 회원을 가입하고 조회할 수 있다.
	- 회원은 일반과 VIP 두 가지 등급이 있다.
	- 회원 데이터는 자체 DB를 구축할 수 있고, 외부 시스템과 연동할 수 있다. (미확정)
- 주문과 할인 정책
	- 회원은 상품을 주문할 수 있다.
- 회원 등급에 따라 할인 정책을 적용할 수 있다.
- 할인 정책은 모든 VIP는 1000원을 할인해주는 고정 금액 할인을 적용해달라. (나중에 변경 될 수 있다.)
- 할인 정책은 변경 가능성이 높다. 회사의 기본 할인 정책을 아직 정하지 못했고, 오픈 직전까지 고민을 미루
고 싶다. 최악의 경우 할인을 적용하지 않을 수 도 있다. (미확정)


## 회원 도메인 설계
---
- 회원 도메인 요구사항
	- 회원을 **가입하고 조회**할 수 있다.
	- 회원은 **일반과 VIP** 두 가지 등급이 있다.
	- 회원 데이터는 자체 DB를 구축할 수 있고, 외부 시스템과 연동할 수 있다. **(미확정)**

![](images/2024-09-29-Spring-Inflearn-Basic-2-1.png)
![](images/2024-09-29-Spring-Inflearn-Basic-2-2.png)

### 구현
---

**회원 등급**
```java
public enum Grade {
	BASIC,
	VIP
}
```

**회원 엔티티** (getter, setter는 생략)
```java
public class Member {

	private Long id;
	private String name;
	private Grade grade;
	
	public Member(Long id, String name, Grade grade) {
		this.id = id;
		this.name = name;
		this.grade = grade;
	}
}
```

**회원 저장소 인터페이스**
```java
public interface MemberRepository {

	void save(Member member);

	Member findById(Long memberId);
}
```

**메모리 회원 저장소 구현체**
```java
public class MemoryMemberRepository implements MemberRepository {
	
	private static Map<Long, Member> store = new HashMap<>();
	
	@Override
	public void save(Member member) {
		store.put(member.getId(), member);
	}
	@Override
	public Member findById(Long memberId) {
		return store.get(memberId);
	}
}
```

DB가 확정이 안됐기 때문에 메모리 회원 저장소로 임시 대체함. HashMap은 동시성 문제가 있기 때문에 **ConcurrentHashMap**을 쓰는게 좋음

**회원 서비스 인터페이스**
```java
public interface MemberService {

	void join(Member member);
	
	Member findMember(Long memberId);
}
```

**회원 서비스 구현체**
```java
public class MemberServiceImpl implements MemberService {
	
	private final MemberRepository memberRepository = new MemoryMemberRepository();
	
	public void join(Member member) {
		memberRepository.save(member);
	}
	
	public Member findMember(Long memberId) {
		return memberRepository.findById(memberId);
	}
}
```

### 테스트
---
JUnit을 이용한 테스트 방법이다. Assertions는 `org.assertj.core.api` 패키지 사용

```java
class MemberServiceTest {
	
	MemberService memberService = new MemberServiceImpl();
	
	@Test
	void join() {
		//given
		Member member = new Member(1L, "memberA", Grade.VIP);
		
		//when
		memberService.join(member);
		Member findMember = memberService.findMember(1L);
		
		//then
		Assertions.assertThat(member).isEqualTo(findMember);
	}
}
```

## 주문 도메인 설계
---
- 주문과 할인 정책
	- 회원은 상품을 주문할 수 있다.
	- 회원 등급에 따라 할인 정책을 적용할 수 있다.
	- 할인 정책은 모든 VIP는 1000원을 할인해주는 고정 금액 할인을 적용해달라. (나중에 변경 될 수 있다.)
	- 할인 정책은 변경 가능성이 높다. 회사의 기본 할인 정책을 아직 정하지 못했고, 오픈 직전까지 고민을 미루고 싶다. 최악의 경우 할인을 적용하지 않을 수 도 있다. (미확정)

![](images/2024-09-29-Spring-Inflearn-Basic-2-4.png)

**역할과 구현을 분리**해서 자유롭게 구현 객체를 조립할 수 있게 설계했다. 덕분에 회원 저장소는 물론이고, 할인 정책도 유연하게 변경할 수 있다

![](images/2024-09-29-Spring-Inflearn-Basic-2-5.png)

- 회원을 메모리에서 조회하고, 정액 할인 정책(고정 금액)을 지원해도 주문 서비스를 변경하지 않아도 된다.
- 역할들의 협력 관계를 그대로 재사용 할 수 있다.

![](images/2024-09-29-Spring-Inflearn-Basic-2-6.png)

### 구현
---
**할인 정책 인터페이스**
```java
public interface DiscountPolicy {

    int discount(Member member,int price);
}
```

**정액 할인 정책 구현체** (VIP면 1000원 할인)
```java
public class FixDiscountPolicy implements DiscountPolicy{

    private int discountFixAmount = 1000; //1000원 할인
    
    @Override
    public int discount(Member member, int price) {
        if (member.getGrade() == Grade.VIP){
            //enum은 equals()가 아닌 == 타입을 사용
			//equals() 메서드는 두 객체의 내용을 비교
			//enum 상수 비교는 내용 비교보다는 메모리 주소 비교가 더 의미있으므로 == 사용
            return discountFixAmount;
        } else {
            return 0;
        }
    }
}
```

**주문 엔티티** (getter, toString 생략)
```java
public class Order {
    private Long memberId;
    private String itemName;
    private int itemPrice;
    private int discountPrice;

    public Order(Long memberId, String itemName, int itemPrice, int discountPrice) {
        this.memberId = memberId;
        this.itemName = itemName;
        this.itemPrice = itemPrice;
        this.discountPrice = discountPrice;
    }

    //계산로직
    public int caculatePrice(){
        return itemPrice - discountPrice;
    }
}
```

**주문 서비스 인터페이스**
```java
public interface OrderService {
    Order createOrder(Long memberId, String itemName, int itemPrice);
}
```

**주문 서비스 구현체**
```java
public class OrderServiceImpl implements OrderService{

    private final MemberRepository memberRepository = new MemoryMemberRepository();
    private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
    
    @Override
    public Order createOrder(Long memberId, String itemName, int itemPrice) {
        // 주문 생성 메서드
        Member member = memberRepository.findById(memberId);
        // 회원 정보 조회
        int discountPrice = discountPolicy.discount(member, itemPrice);
        // 등급에 따른 할인 적용

        return new Order(memberId, itemName, itemPrice, discountPrice);
    }
}
```

주문 생성 요청이 오면, 회원 정보를 조회하고, 할인 정책을 적용한 다음 주문 객체를 생성해서 반환한다.
**메모리 회원 리포지토리와, 고정 금액 할인 정책을 구현체로 생성한다.**

### 테스트
---
```java
public class OrderServiceTest {

    MemberService memberService = new MemberServiceImpl();
    OrderService orderService = new OrderServiceImpl();

    @Test
    void createOrder(){
        Long memberId = 1L;
        Member member = new Member(memberId, "memberA", Grade.VIP);
        memberService.join(member);

        Order order = orderService.createOrder(memberId, "itemA", 10000);
        Assertions.assertThat(order.getDiscountPrice()).isEqualTo(1000);
        // VIP일 때는 1000원 할인 하기로 함
        // order에 담긴 member가 VIP라면 테스트는 passed
		// 따라서 해당 테스트 케이스는 passed
    }
}
```

## 문제점
---
- 이 코드의 설계상 문제점은 무엇일까요?
- 다른 저장소로 변경할 때 OCP 원칙을 잘 준수할까요?
- DIP를 잘 지키고 있을까요?
- **의존관계가 인터페이스 뿐만 아니라 구현까지 모두 의존하는 문제점이 있음**
	- MemoryMemberRepository를 의존함