# Spring Bean

Spring IoC Container에 의해 인스턴스화 되고 설정되고 조립되어 관리되는 객체를 말한다.

## Bean Scope

컨테이너에서 Bean의 생명주기 및 범위를 정하는 방식이다.  
Spring에서 지원하는 스코프는 다음과 같다.  

- singletone
- prototype
- request
- session
- application
- websocket

## Bean 생명주기

- Initialization
- Destruction
- Graceful Shutdown