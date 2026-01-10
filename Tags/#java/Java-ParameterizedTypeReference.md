# ParameterizedTypeReference

`ParameterizedTypeReference`은 컴파일 시점에 완전한 제네릭 타입 정보를 캡처하고 보존하여 턴타임에 사용할 수 있도록 한다.

> Java의 제네릭의 특징 중 타입 소거로 인해 런타임에 제네릭 타입이 제거된다. 

## 사용법

```java
ParameterizedTypeReference<Map<String, String>> responseTypeMap = new ParameterizedTypeReference<Map<String, String>>() {};
ResponseEntity<Map<String, String>> responseMap = restTemplate.exchange("http://example.com/api/config", HttpMethod.GET, null, responseTypeMap);
Map<String, String> config = responseMap.getBody();
```

## 주의사항

**ParameterizedTypeReference**를 사용할 때 주의해야 할 점은 익명 클래스를 생성

