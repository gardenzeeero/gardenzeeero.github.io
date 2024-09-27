---
title: "[WEB] HTTP와 WebSocket의 차이"
date: 2024-09-26 +09:00
categories:
  - Study
  - Web
tags:
  - Web
  - WebSocket
  - HTTP
---
## 😎 HTTP와 WebSocket의 차이점
---
둘다 클라이언트-서버 통신에 사용되는 프로토콜인 것은 동일하다.

하지만 여러 차이점이 있는데, 가장 큰 차이점은 **Connection 유지의 유무**이다. 이를 이해하기 위해서는 HTTP와 WebSocket의 동작방식에 대해 알아야한다.

###  HTTP 동작방식
---
HTTP는 **요청/응답(request/response) 프로토콜**이다. 클라이언트가 요청을 보내고 서버가 응답을 보낸다. HTTP connection이 열려있는 동안 **반이중 통신(half duplex)**를 하게된다.

> half duplex : 데이터가 양방향으로 이동가능하지만, 동시에는 안된다. 즉, 한번에 한방향으로만 전송가능하다. (수신할 때 송신이 안되고, 송신할 때 수신이 안된다.)

HTTP는 일반적으로 TCP를 사용한다. **HTTP/1.0**에서는 서버의 응답을 받으면 **Connection을 종료**한다. 따라서, 새로운 요청을 보내기 위해서는 **다시 TCP 연결**을 맺어줘야한다. 

즉, 3-way handshaking 과정을 또 해줘야한다. 이는 매우 비효율 적이다.

이런 문제를 해결하기 위해 **HTTP/1.1 에서는 persistent connection**을 사용한다. 이는 TCP connection을 재사용하는 방식이다. `Connection: keep-alive` 헤더를 통해 사용할 수 있다.

![](images/2024-09-26-WEB-WebSocket-2.png)

위의 그림처럼 통신을 주고 받다가 더이상 필요없게 되면 "Connection: close" 헤더를 통해 connection을 종료시킬 수 있다. HTTP/2.0 에서는 Multiplexing 기능으로 데이터를 병렬적으로 보낼 수 있게 되었고 속도면에서 향상되었다. 

하지만, TCP connection은 재사용하지만 계속해서 request를 보내야WebSocket의 대체제라고 보긴 어렵고 실시간 통신을 위해 만들어진 기능또한 아니다.

물론 Polling, Long Polling, Streaming 방식으로 실시간인 것 처럼 작동하게 만들 순 있다.

아래의 그림을 보고 읽으면 조금더 이해가 쉬울 듯하다. (만드느라 힘들었다.)
![](images/2024-09-26-WEB-WebSocket-5.png)
#### Polling
일정 시간마다 새로운 정보가 있는지 request하는 방식이다.

1. 새로운 정보가 있는지 request
2. 없으면 없다, 있으면 있다 response
3. 새로운 정보 유입
4. 일정 시간 후 새로운 정보가 있는지 request
5. 있다 response

불필요한 request/response가 오가기 때문에 비효율적이다.

#### Long Polling
일반 Polling과는 다르게 응답을 지연시킨다. 응답을 하는 시점은 새로운 정보가 유입되어 response할 내용이 생겼을 때이다. 또한, 클라이언트는 response를 받자마자 다시 request를 보낸다.

1. 클라이언트의 request
2. 새로운 정보가 올때까지 대기 -> 새로운 정보 유입
3. 서버의 response
4. 클라이언트가 response 받자마자 다시 클라이언트의 request

불필요한 request가 오기 때문에 비효율적이다. 

#### Streaming
결국 Streaming도 Http 이기 때문에 request/response 구조이다. 하지만, 불필요한 request를 줄이기 위해 완전한 응답이 아닌 부분적인 응답만을 발행한다.

하지만, header, cookie 등 불필요한 데이터가 오간다는 점에서는 아직 비효율적이다.

### WebSocket의 동작방식
---
**웹소켓**(WebSocket)은 하나의 TCP 접속에 양방향 전이중 통신(full duplex) 채널을 제공한다.

즉, Socket Connection을 유지한 채 동시에 양방향 통신이 가능하다는 것이다. 

아래의 그림에서 볼 수 있듯이 request/response의 형태가 아니다. 따라서, 요청 없이도 클라이언트와 서버가 언제든 주고 받을 수 있다.

![](images/2024-09-26-WEB-WebSocket-3.png)

기본적으로 WebSocket으로 통신하기 위해서는 HTTP handshaking이 선행된다. 이 과정은 아래와 같다.


#### Http handshaking
```http
GET /hello HTTP/1.1
Host: localhost
Connection: upgrade
Upgrade: websocket, foo
```

서버는 `Connection: upgrade`를 보고 다른 프로토콜로 전환해달라는걸 알게된다. 이후 `Upgrade: websocket, foo`를 보고 Websocket 프로토콜로 바꾼다. 변경이 성공적으로 이루어 지면 서버가 아래와 같은 `101 Switching Protocols`를 보낸다.

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
```

만약 해당 요청의 프로토콜로 변경할 수 없으나 다른 프로토콜이 가능하다면 아래와 같은 `426 Upgrade Required`를 반환한다.

```http
HTTP/1.1 426 Upgrade Required
Upgrade: HTTP/2.0
Connection: Upgrade
Content-Length: 53
Content-Type: text/plain

This service requires use of the HTTP/2.0 protocol
```

이때 부터 WebSocket을 통해 통신이 가능해진다. 불필요한 http header 등의 오버헤드를 줄일 수 있다.


## 🤔 어떤걸 써야할까?
---
상황마다 다르다.

#### HTTP가 더 좋은 상황
- 정보 요청 : 실시간이 필요없는 정보의 현재 상태를 필요로 할 때 좋다.
- 캐싱된 정보 요청 : 자주 조회하는 정보는 캐싱해두곤 한다. 웹소켓은 캐싱을 지원하지 않는다.
- Rest API : HTTP method(Post 등..)가 Rest API와 찰떡궁합이다.
- 이벤트 동기화 : HTTP는 상태코드를 제공하기 때문에 요청에 대한 결과와 이후 동작을 결정하기 좋다.

#### WebSocket이 더 좋은 상황
- 실시간 업데이트 : 실시간으로 정보가 오가야하는 화상채팅과 같은 서비스에 적합하다.
- 양방향 통신 : 요청/응답 구조가 아니기 때문에 즉각적인 데이터 전송이 필요할 때 적합하다. 채팅, 게임, 화상통화 같은 경우에 많이 사용한다.
- 고성능 및 낮은 지연시간 : HTTP에 대한 오버헤드가 적기때문에 성능이 향상된다.

#### Reference
---
https://sendbird.com/developer/tutorials/websocket-vs-http-communication-protocols

https://ably.com/topic/websockets-vs-http