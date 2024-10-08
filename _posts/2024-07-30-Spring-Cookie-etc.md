---
title: "[Spring] 쿠키, 세션, 토큰, 캐시"
date: 2024-07-30 +09:00
categories:
  - Study
  - Spring
tags:
  - 스프링
  - 스프링부트
---
모두 데이터를 저장하는 방법이다.

## 쿠키
---
![](images/2024-07-30-Spring-Cookie-etc-1.png)

**쿠키는 브라우저에 저장되는 정보**이다.

로그인 하지 않고 장바구니에 담아두기, 최근 본 웹툰, 다크모드 설정과 같은 기능에서 쓰인다.

쿠키는 브라우저에 **텍스트 조각**으로 저장된다. 즉, 사용자가 가지고 있는 정보이다.     
당연히 **보안에 취약**할 수 밖에 없다. 탈취당하기도 쉽고 수정, 삭제가 가능하다.    
(설정에서 '쿠키 삭제'의 그 쿠키이다.)

당연히 쿠키에는 **사용자에게 맡겨도 되는 정보만 저장**해야한다.

## 세션
---
**세션은 서버가 나를 알아보는 방법**이다.

흔히 **인가 기능**을 구현할 때 세션을 이용한다.

로그인을 진행하고 여러 페이지를 볼 때, 우리는 해당 사용자가 이용할 수 있는 기능인지 검증해야한다.   
하지만 매 페이지가 바뀔 때 마다 **매번 아이디와 비밀번호를 제공해서 인가 받는 것은 번거롭다.**

세션은 SessionID라는 보증서를 사용한다.   
인가가 필요한 경우에 아이디와 비밀번호 대신에 **SessionID(보증서)를 제출**한다.    
물론 이 보증서를 서버의 보증서와 대조하여 검증한다.

세션을 발급하고 검증하는 과정은 아래와 같다.
1. 아이디와 비밀번호를 서버에 전달한다.
2. 서버는 이를 검증하고 검증 성공 시 **SessionID를 발급**한다.
3. SessionID를 **서버에 저장하고 클라이언트에게 전달**한다.
4. 클라이언트가 서버에 페이지를 요청할 때 **SessionID를 함께 전달**한다.
5. 서버는 **SessionID를 검증하고 요청을 처리**한다.

이처럼 세션은 쿠키와 달리 **보안에 민감한 작업에 사용**할 수 있다.    

## 토큰
---
**토큰은 세션과 비슷하지만 더욱 가볍다.**

세션은 발급한 세션에 대한 내용을 서버에 저장해두어야 한다.   
사용자가 제공한 SessionID를 검증하기 위해서다.    
하지만 매번 SessionID를 검증하기 위해 메모리나 DB에 접근해야한다면 공간 및 시간 효율성이 매우 떨어진다.

이러한 **속도 문제를 해결**하기 위해 토큰을 사용한다.

토큰은 **해싱된(암호화된) 텍스트**이다.    
서버는 이 텍스트를 **시크릿키(비밀키)를 통해 해석**하고 인가 기능을 수행한다.

토큰을 발급하고 검증하는 과정은 아래와 같다.
1. 아이디와 비밀번호를 서버에 전달한다.
2. 서버는 이를 검증하고 검증 성공 시 **토큰 생성 알고리즘**으로 토큰을 생성한다.
3. 생성된 **토큰을 클라이언트에게 전달**한다.
4. 클라이언트가 서버에 페이지를 요청할 때 **토큰을 함께 전달**한다.
5. 서버는 **시크릿 키를 통해 토큰을 검증하고 요청을 처리**한다.

세션과 가장 큰 차이점은 **서버에 토큰에 대한 정보를 저장하지 않는 것**이다.    
(세션은 SessionID가 날라가버릴 위험도 존재한다.)

#### 문제점
그렇다면 빠른 토큰이 무조건 좋은 것 아닌가? 라는 생각이 들 수 있다.

세션은 서버에도 SessionID를 저장해두고 검증한다.   
따라서, **서버에 저장된 SessionID를 삭제**해버린다면 사용자가 가진 SessionID는 무용지물이 된다.

즉, **강제로 로그아웃**을 시킬 수 있다.

하지만, 토큰은 정상적인 토큰인지만 체크하기 때문에 이러한 기능을 사용하지 못한다.

## 캐시
---
**캐시는 전송량은 줄이고 속도는 높여주는 역할을 한다.**

캐시는 다양한 분야에서 사용된다. 따라서, 의미하는 바가 다양하다.    
하지만, 모두 공통된 목적을 가지고 있다.

**반복적으로 사용하는 정보를 저장해두어 빠르게 이용**할 수 있다.

캐시 서버를 예로 들 수 있다. 해외의 사이트에서 정보를 받아온다고 가정해보자. 
매번 사이트에서 정보를 받아올 때 마다 우리는 해외 통신사에게 **망 사용료를 내야한다.**   
**많은 사용자들이 해당 정보를 요청**한다면 매번 망 사용료가 부과될 것이다.    
해외의 서버이기 때문에 **속도가 느려지는 것은 당연**하다.

이때, 캐시서버가 한번만 해외 서버에 요청을 보내고 이를 서버에 저장해둔다.   
사용자들이 해외 서버에 정보를 요청하려고 한다면 **캐시서버에 저장된 정보를 전달**해준다.  

이렇게 하면 망사용료도 줄일 수 있으며 속도는 높아진다.
