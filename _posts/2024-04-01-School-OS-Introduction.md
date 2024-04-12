---
title: "[OS] 운영체제의 역할과 구조 개요"
date: 2024-04-11 +09:00
categories:
  - School
  - OS
tags:
  - 운영체제
---
3우선 운영체제가 어떻게 동작하는지 알기 위해 전체적인 동작 과정을 알아야한다.   
그런 의미에서 대략적인 내용을 정리해본다.

## 1. 운영체제를 알기위해
---
운영체제(Operating System)란 무엇일까??

**"Everything a vendor ships when you order an operating system"** is good approximation

운영체제를 주문했을 때 배송오는 모든것"이라고 할 만큼 그 개념은 애매하다.  
어떤 책이든 운영체제라고 소개하면 운영체제라고 할 수 있다. 하지만 그 틀은 대개 비슷하다.

1. 운영체제는 **Resource Allocator(Manager)**이다.
	- 모든 물리적이고 추상적인 **자원을 관리**한다.
	- 물리적 자원 : CPU, Memory, HDD, Terminal, Network 등
	- 추상적 자원 : Task(Process), Segment/page, File System, Terminal Driver 등
		- 추상적 자원들은 대개 **물리적 자원을 효율적으로 관리**하기 위해 존재한다.
	- 동시다발적인 요청들에 **효율적이고 공정**하게 자원을 **사용하도록 결정**한다.
	- Memory의 할당 뿐만 아니라 분배 및 해제도 하기 때문에 Manager라고 보는게 더 맞다.
2. 운영체제는 **Control Program**이다.
	- **프로그램의 실행을 제어**해 컴퓨터의 **부적절한 사용 및 오류를 방지**한다.

### 1.1 운영체제의 역할
---
운영체제의 목표는 뭘까?

1. 일반 **사용자**의 관점 : **프로그램을 실행**시켜주고 문제를 해결해준다.
2. **시스템**의 관점 : **자원을 관리**해주고 **프로그램을 제어**한다.

그런 의미에서 운영체제의 역할은 크게 두가지라고 볼 수 있다.

**편의성**과 **효율성**을 향상시켜준다.

**편의성:** 컴퓨터 시스템을 더 편리하게 사용할 수 있도록 해준다.
**효율성**: 컴퓨터 하드웨어를 더 효과적으로 관리한다.

이 둘은 **trade-off**의 관계이다.   
편의성이 향상되면 효율성이 감소되고 편의성이 감소되면 효율성이 향상된다.   
그래서 우리는 이 둘이 모두 증진될 수 있도록 OS를 설계하도록 노력해야한다.   

### 1.2 컴퓨터 시스템 구조
---
컴퓨터 시스템은 크게 4가지로 나뉜다.

1. **Hardware**
	- 기본적인 **computing resource**를 제공함
	- CPU, Memory, I/O Device
2. **Operating System**
	- Hardware를 **제어하고 조정**한다.
	- Windows, Linux, Mac OS
3. **Application Programs**
	- 사용자의 문제를 해결하는데 드는 system resouce의 사용방식을 정의함
	- Compilers(system program), Web browsers, Database system
4. **Users**
	- 사용자

그림으로 나타내면 아래와 같다. 

![](images/2024-04-01-School-OS-Introduction.png)


### 1.3 운영체제 설계 및 구현
---
운영체제의 설계 및 구현에 **정답이 있는 것은 아니다.**  
하지만 **성공적인 설계**는 존재한다.   

따라서 **운영체제마다 내부 구조가 다르다**.   
또한, 어떤 하드웨어를 선택하는지, 어떤 종류의 시스템을 사용하는지에 따라서도 달라진다.

우선 설계를 하기 위해서는 목표를 설정해야한다.
- **User** goal : 사용하기 편해야 함, 배우기 쉽고 신뢰할 수 있어야함, 빨라야 함
- **System** goal : 설계, 구현, 유지보수가 쉬워야 함, 효율적이고 유동적이며 신뢰할 수 있어야 함

이러기 위러한 목표를 달성하기 위해서는 **Policy와 Mechanism을 분리**해야 한다.    
- Policy : **무엇을 수행**할 것인가?
	- Policy는 **언제든 바뀔 수 있다.**
	- 예시 : CPU timer를 몇초 동안 설정할 것인가?
- Mechanism : **어떻게** 해당 일을 수행할 것인가?
	- Policy의 **변경에 민감하지 않아야 한다**. 즉, 정책이 바껴도 Mechanism의 변경은 없어야한다.
	- 예시 : 매개변수로 넘어온 인자만큼 sleep한다.

이러한 Policy와 Mechanism의 분리는 **OS의 유연성**을 더해준다.

## 2. 컴퓨터의 부팅
---
컴퓨터 딱 켜지면 아래의 순서대로 실행이 된다.

1. **BIOS**의 ROM 또는 EEPROM에 저장된 **Firmware가 동작**한다.
	- 이 Firmware는 **POST routine**을 가진다.(Power On Self Test)
	- POST routine은 컴퓨터가 정상적인 상태인지 점검하는 테스트이다.
	- 이 POST routine이 정상적이라면 **Bootstrap loader**가 동작한다.
2. Bootstrap loader는 **disk의 boot sector**에 존재하는 **더 복잡한 boot program**을 실행시킨다.
	- BIOS의 Bootstrap loader의 역할은 이것 뿐이다.
3. boot program이 **kernel을 실행**시키고 OS가 동작한다.


## 참고 자료
---
책 - operating system concepts (일명 공룡OS)
그림 - 영남대학교 곽종우 교수님 OS 수업
