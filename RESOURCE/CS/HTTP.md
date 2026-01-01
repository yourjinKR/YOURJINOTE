# HTTP

> HTTP (HyperText Transfer Protocol)

인터넷 상에서 클라이언트와 서버가 자원을 주고 받을 때 쓰는 규약입니다.

> HTTP는 HTML 문서와 같은 리소스들을 가져올 수 있도록 해주는 [프로토콜](Protocol.md)입니다

## 보안 취약점

HTTP는 여러 보안 취약점 이슈를 가지고 있어 이에 대한 방안으로 HTTPS를 사용합니다.

## HTTPS

> HTTPS(HyperText Transfer Protocol Secure)

인터넷 상에서 SSL 프로토콜을 사용하여 클라이언트와 서버가 자원을 주고 받을 때 쓰는 통신 규약이다.  
HTTPS는 텍스트를 암호화한다.

> 기존의 HTTP에 암호화라는 보안 계층을 추가한 것이다.

### HTTPS 통신 흐름

전체적인 통신 흐름은 크게 [Hanshake](HandShaking.md) - 데이터 전송 - 연결 종료의 단계를 거친다.

#### 1. TLS HandShake 

클라이언트와 서버가 서로를 확인하고 데이터를 암호화할 '비밀 키'를 공유하는 과정이다.

1. Client Hello : 브라우저가 서버에 접속하면, 자신이 원하는 암호화 방식과 무작위 문자열을 보낸다.
2. Server Hello : 서버는 브라우저가 보낸 방식 중 하나를 선택하고, 자신의 인증서와 무작위 문자열을 응답으로 보낸다.
3. 인증서 확인 : 브라우저는 서버가 보낸 인증서가 신뢰할 있는 기관(CA)에서 발급된 것인지 확인한다.
4. Pre-Master-Secret-Key 생성 : 브라우저는 서버의 인증서 안에 들어있는 공개키로 무작위 비밀번호(Pre-Master-Secret)를 암호화하여 서버에 보낸다.
5. Session Key 생성 : 서버는 자신의 개인키로 이를 복호화 한다. 이제 클라이언트와 서버는 동일한 비밀번호를 가지게 되며, 이를 바탕으로 실제 통신에 사용할 대칭키(Session KEy) 를 만든다.
6. HandShake 종료 : 서로 확인이 끝나면이제부터 이 대칭키로 데이터를 주고받을 준비가 되었다고 알린다.

#### 2. 데이터 전송

Handshake가 성공적으로 끝나면, 이제 실제 HTTP 데이터를 주고받습니다.

- 대칭키 암호화 : 앞서 만든 Session Key를 사용하여 데이터를 암호화하고 복호화한다.
- 효율성 : 공개키 방식은 연산량이 많아 느리기 때문에, 실제 데이터를 주고받을 때는 속도가 빠른 대칭키 방식을 사용한다.

### HTTP가 안전한 이유

공개키와 대칭키 방식을 혼합해서 사용하기 때문이다.

|**구분**|**역할**|**특징**|
|---|---|---|
|**공개키 방식**|대칭키를 안전하게 전달할 때 사용|보안성이 매우 높지만 연산이 느림|
|**대칭키 방식**|실제 데이터를 암호화하여 통신할 때 사용|연산이 빠르지만 키가 유출되면 위험함|

<br>

# 출처

[Github](https://github.com/gyoogle/tech-interview-for-developer/blob/master/Computer%20Science/Network/HTTP%20%26%20HTTPS.md)  
[유튜브 - Https 어떻게 동작하는걸까요?](https://www.youtube.com/watch?v=kBlQiwXSx8A)