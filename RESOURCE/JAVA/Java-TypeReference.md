# TypeReference

런타임에 제네릭 타입 정보를 보존해서(캡처해서) JSON 같은 걸 정확히 역직렬화하기 위한 장치이다.  

대표적으로 Jackson의 역직렬화 과정에서 많이 사용된다.  
(`com.fasterxml.jackson.core.type.TypeReference<T>`)

```java
User u = objectMapper.readValue(json, User.class);
```

이와 같이 `readValue`에 클래스 정보를 매개변수로 받아 어떤 타입의 데이터로 변환시킬지 알려 줄 수 있다.

```java
List<User> list = objectMapper.readValue(json, List.class); // User 정보 없음
```

그러나 위와 같이 제네릭 타입의 데이터 형태에서는 한계가 존재한다.

> 제네릭의 타입 소거로 인해 런타입 과정에서 제네릭 타입이 `Object`로 변하기 때문에

## 사용 방법

```java
new TypeReference<List<User>>() {}
```

내부에 익명 클래스를 만들면 런타임에서 해당 타입에 맞게 리플렉션을 통해 가져올 수 있다.

```java
Map<String, List<User>> data = objectMapper.readValue(
    json,
    new TypeReference<Map<String, List<User>>>() {}
);
```

<br>

# ParameterizedTypeReference

Spring Framework에서 제공하는 클래스이다.    
Jackson의 `TypeReference`와 동일하다.  

Spring에서는 JSON 파싱 이전에 아래와 같은 프레임워크 레벨에서 타입 정보를 알아야 하기에 Jackson에 종속되지 않는 타입 캡처 도구를 만든 것이다.

- HTTP 응답 바디
- HttpMessageConverter
- WebClient / RestTempleate

## 차이점 비교

```java
ResponseEntity<List<User>> response =
    restTemplate.getForEntity(url, List.class);
```

```java
ResponseEntity<List<User>> response =
    restTemplate.exchange(
        url,
        HttpMethod.GET,
        null,
        new ParameterizedTypeReference<List<User>>() {}
    );
```

## 내부 동작

- 익명 클래스를 생성
- 부모 클래스의 제네릭 시그니처가 클래스 메타 데이터에 남음
- Spring이 리플렉션으로 꺼냄


## 주의사항

**ParameterizedTypeReference**를 사용할 때 주의해야 할 점은 익명 클래스를 생성

