---
title: "[OS] 운영체제의 역할과 구조 개요"
date: 2024-04-11 +09:00
categories:
  - School
  - OS
tags:
  - 운영체제
---
우선 운영체제가 어떻게 동작하는지 알기 위해 컴퓨터의 전체적인 동작 과정을 알아야한다.   
그런 의미에서 대략적인 내용을 정리해본다.

## 1. 운영체제를 알기위해
---
운영체제(Operating System)란 무엇일까?

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

### 1.2 컴퓨터 시스템 구성
---
컴퓨터 시스템은 크게 4가지로 구성된다.

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

![](images/2024-04-11-School-OS-Introduction.png)

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
	- 보통 boot sector는 **저장장치 내 첫번째 구역**에 존재한다.
3. boot program이 **kernel을 실행**시키고 OS가 동작한다.


## 3. 컴퓨터 시스템 구조
---
CPU와 Device Controller들은 **bus를 통해 메모리에 접근하고 통신**한다.   
따라서 **동시에** 여러장치들이 메모리에 **접근한다면 경쟁상태**가 된다.

Bus의 폭은 CPU의 **register 크기**에 따라 32bit와 64bit 로 나뉜다.   
이는 **CPU가 한번에 처리**할 수 있는 크기를 나타낸다.

Bus는 **Address Bus, Data Bus, Controll Bus**가 있다.(자세한 내용은 Bus를 다룰 때 자세히 알아보자.)   
아무튼 **32bit 시스템**을 사용한다고하면 **Address는 32bit**가 된다.   
즉, 약 40억 번지 까지의 주소를 사용할 수 있다.

### 3.1 General Operation
---
일반적으로 CPU와 I/O 장치들은 **동시에 실행 될 수 있다.**

- CPU : 메인 메모리에서 local buffer로 데이터를 가져오고 내보낸다.
- I/O : 장치에서 Controller의 local buffer로 데이터를 가져온다.

**Device Controller**란 뭘까?
- Device가 동작하게 만들어주는 **H/W**
- 각 device controller는 각 장치를 담당한다.
- 일반적으로 local buffer를 가지고 있다.
- 일이 끝나면 **interrupt를 통해 CPU에게 알린다.**


## 4. Interrupt
---
Interrupt는 뭘까?

직역하면 중단, 방해 라는 의미가 있다.   
이러한 뜻은 실행흐름을 중단시킨다는 의미가 있다.

### 4.1 Interrupt의 기능
---
Interrupt는 Interrupt Vector를 통해 **Interrupt Service Routine(ISR)**에게 제어권을 전달한다.

Interrupt Vector에는 **ISR에 대한 주소**가 저장되어 있다.   
**Interrupt가 발생**했을 때 **Interrupt Vector**에 방문해 해당 Interrupt를 처리할 수 있는 ISR을 찾아간다.

Interrupt를 처리하는 동안에는 손실을 막기 위해 Interrupt는 막히기도 한다.
(Maskable Interrupt, Non-Maskable Interrupt)

### 4.2 External Interrupt vs Internal Interrupt
---
Interrupt는 **CPU를 기준**으로 **어디에서 발생하는 가**에 따라서 크게 두가지로 나눌 수 있다.

#### External Interrupt
- **CPU 외부**에서 일어나는 Interrupt이다.
- **비동기적**이다.
- **Maskable** Interrupt이다.
- **H/W 장치**들에 의해서 일어난다.
	- 예시 : 키보드 
	- **CPU Clock과 상관없이** 동작(비동기적)하고 **무시가능**(Maskable)하며 **H/W 장치**이다.

#### Internal Interrupt
Internal Interrupt는 세가지로 나뉜다.
1. Fault
	- **복구가능한 문제가 발생**해 해결 후 **다시 실행**시키는 것이다.
	- 현재위치 저장 -> ISR -> 저장된 위치로 돌아가 **해당 명령 다시 수행**
	- 예시 : page fault, memory space fault
2. Trap
	- 사용자가 **의도적으로 발생**시키는 Interrupt이다.(따라서 Exception이라기엔 무리가 있다.)
	- 따라서 정상적인 프로그램 흐름이라고 볼 수 있다.
	- 현재위치 저장 -> ISR -> 저장된 위치로 돌아가 **다음 명령** 실행
	- 예시 : System Call, Exception Handling, fork 메서드
3. Abort
	- 복구할 수 없는 치명적인 오류이다.
	- ISR만 실행
	- 예시 : divide by zero

### 4.3 Polling
---
대부분의 OS에는 Interrupt 방식을 채택한다.   
하지만 **Polling 방식**도 있다.

Polling 방식은 **Busy waiting** 이라고도 한다.   
**Device의 상태를 계속해서 체크**하고 있는다는 것이다. 이동안 **CPU는 다른일을 하지 못한다.**

이러한 방식에는 장단점이 존재한다.
- 장점 : **Response**가 빠르다. **context switching에 따른 overhead**가 없다.
- 단점 : CPU cycle의 낭비가 생긴다.

이러한 Polling 방식을 사용하기 위해서는 요구조건이 붙는다.
- 빠른 I/O Device, 짧은 I/O service routine, 프로세스가 드물게 실행되야 함


## 5. Storage Structure
---
Storage는 아래의 기준으로 **계층적 구조**를 가진다.
- 속도
- 비용 (Byte당 얼마)
- Size
- 휘발성

그림을 통해 파악하면 좀 더 편하다.

![](images/2024-04-11-School-OS-Introduction-1.png)

1. **register**
	- **CPU 내부**에 있는 가장 빠른 장치
	- CPU 및 Complier가 관리하고 할당한다.
2. **cache**
	- **CPU가 전적으로 관리**한다.
	- **CPU 내부/외부**에 따라 나뉜다.
		- **on-chip** cache : **CPU의 연산장치와 붙어**있으며 보통 **L1** cache가 해당된다.
			- 고성능 CPU의 경우 L2 cache도 있을 수 있다.
		- **off-chip** cache : CPU밖에 존재한다. 보통 L2, L3 cache가 해당된다.
	- **어떤 값을 저장**하냐에 따라 나뉜다.
		- **i-cache** : **명령어**를 저장, 주로 L1 cache가 해당됨
		- **d-cache** : **데이터**를 저장, 주로 L2, L3 cache가 해당됨
		- 이렇게 구분하는 구조를 **Havard Architecture**라고 함 <-> **폰노이만 Architecture**
4. **main memory**
	- 이 밑으로는 **OS가 관리**한다.
	- 따라서 OS의 입장에서는 register와 cache의 존재를 모른다.
5. **electronic disk**
	- 흔히 말하는 SSD이다.
	- 여기서 부터는 비휘발성 저장장치이다.
6. **magnetic disk**
	- 흔히 말하는 HDD이다.
7. optical disk
	- 흔히 말하는 CD ROM이 해당된다.
8. magnetic tapes
	- 순차 저장 장치이다.


## 6. 운영체제의 관리 체계
---
### 6.1 Process Management
----
**Program과 Process란?**

- **Program**
	- **정적인** entity
	- **하나만 존재**한다.
- **Process**
	- **동적인** entity, **실행중인 program**
	- **여러개의 copy**가 존재할 수 있다.
	- 동작하기 위해서 **여러 자원들이 필요**하다.
	- process의 **종료**는 **재사용가능한 모든 자원의 반납**을 의미한다.

#### Multiprogramming
Multiprogramming은 **wait(I/O 등)하는 시간**으로 인한 **CPU cycle의 손실**을 막기 위해 만들어졌다.   
즉, **효율성을 향상**시키기 위해 만들어졌다.

- 하나의 process가 연산 작업 후 **waiting 하는 동안 다른 process의 연산**을 하는 방식이다.
- process는 **job scheduling**에 의해 선택된다.
	- 해당 process들은 메모리에 올라와 **ready-to-execute**여야함.
- 장점 : **CPU가 항상 무언가를 실행**시키도록 한다.(busy 상태를 유지함)

#### Time Sharing
Time Sharing은 여러 **process들이 동시에 실행되는 것 처럼보이도록** 만들어졌다.  
즉, **편의성을 향상**시키기 위해 만들어졌다.

- **일정한 시간**마다 process를 바꿔 실행한다.
	- 일정한 시간 : **time slice** 라고 한다.
	- **context switching** : process를 바꾸는 일련의 행위 **(overhead가 생김)**
- **Interactive System**이다.
	- **Response time이 빠름** -> 사용자가 자신의 작업과 상호작용할 수 있도록 한다.
- **CPU Scheduling**을 통해 이루어진다.
- 만약 process를 **memory에 올리지 못하면** 다른 process를 swap out 시키고 **swap in** 한다.
- 장점 : 사용자의 입장에서 여러 **process가 동시에 실행**되는 것 처럼 **보인다.**

#### Management 활동
운영체제가 process를 관리하기 위해 아래와 같은 일들도 수행해야 한다.
- user process 및 system process의 **생성 및 종료**한다.
- process의 **suspending 및 resuming**

위의 일들도 중요하지만 더 신경써야하는 문제들이 있다.
- **process synchronization(동기화)** mechanism 제공한다.
- **process communication(통신)** mechanism 제공한다.
- **dealock handling**에 대한 mechanism 제공한다.
	- deadlock이란 아무도 접근하지 못하게 된 상황이다.(자세한건 추후에 다룬다.)

### 6.2 Memory Management
---
process를 실행시키기 전후로 **memory에 올라와 있는** 모든 **data와 명령어**를 관리한다.   
(data cache, instruction cache 등)

#### Management 활동
Memory를 관리하는 것이기 때문에 **공간의 자원을 관리**하는 것이라고 볼 수 있다.
- memory에서 **어떤 자원이 누구에 의해** 사용되는지 tracking 한다.
- process와 data를 **memory에 올리고 내리는 것**을 결정한다.
- 필요한 memory 공간을 **할당 및 해제**한다.

### 6.3 File Management
---
OS는 storage에 대한 **일관적이고 논리적인 관점**을 제공한다.

이는 **File**이라는 **추상적 개념**을 사용한다. 이는 **물리적인 속성을 논리적으로 나타낸 것**이다.    
각 medium은 device driver에 의해 관리된다.

#### File System & Storage Management
**File System management**
- File은 **directory 구조**로 이루어져 있다. (대개 tree로 이루어져 있지만 OS마다 다르다.)
- **접근권한**을 통해 사용자를 결정한다.
- OS는 아래의 일도 수행한다.
	- file 및 directory를 **생성하고 삭제**한다.
	- file 및 directory를 **조작**한다.
	- **storage와 mapping**한다. (어떤식으로 저장할 것인지 결정, 예시 : 조각모음)
	- 비휘발성 storage에 **backup**한다.

**Mass-stroage Management **
- **할당공간 관리 및 자유공간 관리**
- Storage **할당**
- **Disk scheduling**
	- 여러 process가 같이 접근 하려고 할 때 **scheduling을 한다.**
	- 예시 : HDD에서 읽을 때 찾는 도중 다른 process가 요청한 정보를 만나면 같이 읽어간다.

## 참고 자료
---
책 - operating system concepts (일명 공룡OS)
그림 - 영남대학교 곽종우 교수님 OS 수업
