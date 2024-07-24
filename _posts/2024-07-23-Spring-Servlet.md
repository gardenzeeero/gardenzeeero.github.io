---
title: "[Spring] Servlet 및 Dispatcher Servlet"
date: 2024-07-23 +09:00
categories:
  - Study
  - Spring
tags:
  - 스프링
  - 스프링부트
  - 서블릿
  - 디스패처서블릿
---
## Servlet
---
Servlet을 정의하는 말은 다양하다.
- **웹페이지를 동적으로 생성하는 작은 서버측 프로그램**
- 클라이언트의 요청을 처리하고 반환하는 자바 웹 프로그래밍 기술
- 동적 페이지를 만들기 위한 기술

결론은,
**클라이언트와 쉽게 소통하기 위해 만들어 놓은 서버측 프로그램(또는 기술)**이다.

## Servlet의 장점
---
원래 클라이언트와 소통하기 위해서는 아래와 같은 작업들이 이루어져야한다.

1. TCP/IP 대기, 소켓 연결
2. HTTP 요청을 잘라서 해석하기
	- Post인지, 같이 담겨온 값은 무엇인지 등 파싱
3. 비즈니스 로직 실행
	- DB에 저장하고~ 불러오고~ 등
4. HTTP 응답 메세지를 규약에 맞게 작성
	- 헤더 만들고~ 바디에 HTML 넣고~ 등
5. TCP/IP 응답 전달, 소켓 종료

위와 같은 작업을 매번 해줘야했다.   

하지만, Servlet의 규약에 맞게 Servlet을 정의해주면 **HTTP 요청을 쉽게 사용하고 반환할 수 있게 된다.**

**HttpServeltRequest 객체와 HttpServletResponse 객체**를 이용해 파라미터, 헤더, 바디 등을 쉽게 다룬다.
- 파라미터 읽기 : httpServletRequest.getParameter("username")
- 요청 종류 판단 : httpServletRequest.getMethod();

따라서, Servlet을 사용하면 **개발자는 비즈니스 로직에 더욱더 집중할 수 있게된다.**

## HttpServlet의 구현
---
Servlet은 HttpServlet 클래스를 상속받아 만들 수 있다.

```java
@WebServlet(name = "NewServlet", urlPatterns = "/new")
public class NewServlet extends HttpServlet {

	@Override
	protected void service(HttpServletRequest request, HttpServletResponse response){
		//reqeust를 이용해 비즈니스 로직 관리
	}
}
```

위와 같이 **service() 메서드를 Override하여 요청을 처리**할 수 있다.
- HttpServletRequest를 이용해 요청 정보를 사용
- HttpServletResponse를 이용해 응답 정보를 입력

>WebServlet 어노테이션으로 Servlet을 매핑해주는 것은 최신 버전이다.    
>구버전에서는 설정 파일에 적었다.

## Servlet Container
---
**Servlet을 관리하고 실행시키는 프로그램**이다.    
예를 들어, 톰캣처럼 Servlet을 지원하는 WAS를 Servlet Container라고 할 수 있다.

Servlet Container는 아래와 같이 작동한다.
1. ServletRequest 객체와 ServletResponse 객체 생성
2. Servlet 매핑
3. Servlet 인스턴스 존재 유무를 확인하고 init() 메서드를 통해 생성 (싱글톤 객체이기 때문)
4. Servlet Container에 쓰레드를 생성하고 res, req를 인자로 service() 메서드 실행
5. 응답 후 request, response 객체 소멸 

> Servlet은 싱글톤 객체이기 때문에 소멸은 WAS가 종료될 때 destory()메서드를 호출해 진행한다.

즉, **서블릿의 생명주기를 관리하는 객체**이다.

그렇다면 여러 요청이 한번에 들어오면 어떻게 할까?    
4번 단계에서 볼 수 있듯이 쓰레드를 생성하여 멀티스레드를 사용한다.    

#### 요청마다 쓰레드 생성시 장단점
- 장점
	- 동시 요청 처리 가능하다.
	- 하나의 쓰레드가 오래걸려도 다른 쓰레드들은 정상 동작한다.
- 단점
	- 생성 비용이 너무 비쌈 -> 응답속도가 느려진다.
	- Context Switching 비용이 생긴다.
	- 쓰레드 생성에 제한이 없어서 서버가 터져버릴 수 있다.

#### 쓰레드풀   
쓰레드 풀은 쓰레드를 만들어두고 **요청이 들어오면 나누어 주는 역할**을 한다.  

쓰레드 풀은 정해진 개수의 쓰레드만 만들어 두기 때문에 **쓰레드 생성에 제한**을 둘 수 있다.    
(개수(톰캣은 기본 200개)를 넘어선 이후엔 대기 또는 거절 상태로 만들 수 있음)    
사용이 끝나면 쓰레드를 삭제하는 것이 아니라 **쓰레드 풀에 반납**한다.

쓰레드 풀을 사용하면 쓰레드 생성 및 종료 비용이 줄어들고 응답이 빨라진다.    
또한, 기존 요청은 안전하게 처리가 가능하다.

## Dispatcher Servlet
---
 요청 경로마다 Servlet을 정의하는 것은 핸들러마다 공통된 로직을 중복작성하게 된다는 것이다.
(한글 인코딩 처리 등이 있다고 한다.)

이런한 공통 로직을 줄이기 위해 **공통된 로직을 하나의 Servlet만으로 처리**하면 어떨까?

![](images/2024-07-23-Spring-Servlet-2.png)

개발자는 개별적인 로직에만 집중할 수 있게 된다.   
이러한 디자인 패턴을 **프론트 컨트롤러 패턴**이라고 한다.

Spring MVC도 이 프론트 컨트롤러 패턴을 따르고    
**공통 로직을 처리하는 Servlet을 Dispatcher Servlet** 이라고 하는 것 이다.

![](images/2024-07-23-Spring-Servlet-1.png)

Dispatcher Servlet이 모든 공통 로직을 처리하기에는 똑같이 무리가 생긴다.   
따라서, Dispatcher Servlet은 로직들을 **다른 Servlet에 위임하고 반환받는 역할만** 한다.

Dispatcher Servlet의 간략한 동작 과정은 아래와 같다.

![](images/2024-07-23-Spring-Servlet-3.png)

#### 레퍼런스
---
[블로그](https://velog.io/@suhongkim98/Front-Controller-%ED%8C%A8%ED%84%B4%EA%B3%BC-spring-MVC)    
[유튜브](https://www.youtube.com/watch?v=calGCwG_B4Y&t=590s)

