---
title: "[Java] JVM Architecture (JVM 메모리구조)"
date: 2024-03-21 +09:00
categories:
  - Study
  - Java
tags:
  - 자바
---
# JVM Architecture
---
당연하게도 JVM은 운영체제 종속적임    
그래서 운영체제마다 JVM의 구조도 조금씩 다름   
하지만 일반적인 구조는 아래와 같음

![](images/2024-03-17-Java-JVM-Architecture.png)

## 1) Class Loader System
---
![](images/2024-03-17-Java-JVM-Architecture-2.png)

Class Loader는 항상 메모리에 올라와 있음   
클래스 파일을 메모리로 가져오는 역할을 함

이를 동적 클래스 로딩(Dynamic Class Loading)이라고 함    
Class Loader는 런타임에 클래스 파일을 메모리에 올리고 링크한 후 초기화 까지 해줌

Class Loader는 파일을 메모리에 올리는 것외에도    
바이너리 데이터를 생성하고 클래스 각각에 대한 정보도 Method 영역에 올림


## 2) Runtime Data Area
---
이 영역은 JVM이 OS에서 실행될 때 할당되는 메모리 영역임

![](images/2024-03-17-Java-JVM-Architecture-3.png)

### Method 영역
Method 영역에는 Class Loader에 의해 파일의 바이너리 데이터와 클래스 각각의 대한 정보를 저장함
- 해당 클래스 및 부모 클래스의 정규화된 이름
- 클래스 파일이 클래스, 인터페이스, enum과 관련되어있나 여부
- 접근 제한자(public 등), static 변수, 메서드 정보 등

Method 영역은 스레드끼리 공유함 (공유 리소스)    
또한, Method 영역은 어디서든 접근 가능하고 JVM이 종료될 때 메모리에서 해제됨 (메모리 누수 주의)

한마디로 클래스에 대한 정보를 저장함

### Heap 영역
Heap 영역은 인스턴스가 생성되는 공간임

인스턴스 변수 및 배열, 인스턴스에 대한 정보가 저장되는 공간임   
따라서 흔히 Garbage Collector의 대상이 되는 공간임

Heap 영역도 스레드끼리 공유함 (공유 리소스)

### Stack 영역
메서드가 호출 되었을 때 Stack 영역에 메모리가 할당됨
이 할당된 공간을 Stack Frame이라고 함
이 Stack Frame이 Stack 영역에 쌓이는 구조임

![](images/2024-03-17-Java-JVM-Architecture-1.png)

위에서 볼 수 있듯이 Stack Frame은 세가지로 구성됨
- 지역변수 배열
- 피연산자 Stack
- 프레임 데이터
<del>(더 자세한 내용은 알아서 찾아보시길)</del>

한마디로 메서드가 작업을 수행할 때 필요한 매개변수 및 지역변수들을 저장함

당연히 Stack 영역은 공유 리소스가 아님   
스레드마다 독립적인 공간을 가짐

### PC Register
스레드가 시작 됐을 때 현재 명령어의 주소를 기억하기 위해 PC Register를 생성함     
(기본 메서드의 경우 생기지 않음)

당연히 스레드마다 PC Register가 생김

### Native Method Stack
Java 가 아닌 다른 언어 (C, C++) 로 구성된 메소드를 실행이 필요할 때 사용되는 공간

네이티브 코드도 Stack이 필요하기에 네이티브 코드를 위한 Stack이라고 보면 됨

당연히 스레드마다 독립적인 공간을 가짐


## 3) Execution Engine
---
바이트 코드의 실행이 여기서 일어남

Execution Engine이 여기서 한줄 씩 명령어를 읽어가며 실행함

![](images/2024-03-17-Java-JVM-Architecture-4.png)

### Interpreter
한국어로 해석하면 통역사임

Interpreter가 바이트코드를 해석하고 명령어를 실행하는 역할을 함   
한줄 한줄 해석하는건 잘함 근데 실행이 느림    
또한, 똑같은 메서드 여러번 호출되도 또 해석함

### Just-In-Time(JIT) Complier
똑같은 메서드를 여러번 호출하면서 생기는 문제를 JIT Complier가 해결해 줌

그 방법은 아래와 같음
1. 바이트 코드를 네이티브 코드(기계어)로 변환함
2. 중복된 메서드가 호출되면 네이티브 코드를 직접 제공함
3. 해당 네이티브 코드는 캐시에 저장됨 (당연히 더 빠름)

근데 JIT는 Interpreter보다 컴파일하는데 더 오래 걸림   
그래서 한번만 쓸 메서드는 JIT보다 Interpreter쓰는게 더 좋음    
그리고 캐시를 안쓴다는 이점도 생김   

그래서 JIT가 내부적으로 빈도를 측정하고 JIT사용 여부를 결정함

### Garbage Collector
JVM은 객체가 참조되고 있으면 살아있는 걸로 판단함

근데 아무 변수도 객체를 참조하고 있지 않으면 Garbage로 판단하고 객체 제거 후 메모리 회수를 함      
(System.gc()를 이용해 강제 메모리 회수도 가능)

## 4) JNI (Java Native Interface)
---
![](images/2024-03-17-Java-JVM-Architecture-5.png)

이 인터페이스는 실행에 필요한 Native Method Libraries와 상호 작용함

이걸 통해서 JVM이 C/C++ 라이브러리를 불러올 수 있음    
또한 하드웨어의 C/C++ 라이브러리에 의해 호출될 수 있음

## 5) Native Method Libraries
---
실행 엔진에 필요한 C/C++ 라이브러리 모음임

JNI를 통해서 접근이 가능함

