# 개요
자바 Spring Data에서 제공하는 페이징 기능

## Pageable 인터페이스

```java
public interface Pageable {
	static Pageable unpaged() { return unpaged(Sort.unsorted()); }

	static Pageable unpaged(Sort sort) { return Unpaged.sorted(sort); }

	static Pageable ofSize(int pageSize) { return PageRequest.of(0, pageSize); }

	default boolean isPaged() { return true; }

	default boolean isUnpaged() { return !isPaged(); }

	int getPageNumber();
	int getPageSize();
	long getOffset();

	// 생략
}
```
- Spring에서는 Pagination을 지원하는 `Pageable` 인터페이스를 제공한다.
    - `getPageNumber()` : 현재 페이지 번호를 반환(0부터 시작)
    - `getPageSize()` : 한 페이지당 최대 항목 수를 반환
    - `getOffset()` : 현재 페이지의 시작 위치를 반환
    - `getSort()` : 정렬 정보를 반환
    - `next()` : 다음 페이지 정보를 반환
    - `previous()` : 이전 페이지 정보를 반환
- `Pageable` 을 이용해서 페이지 번호, 페이지당 항목 수, 필요에 따라 정렬 정보를 추가로 지정할 수 있다.
- `Pageable` 로 지정한 정보들을 가지고 `Page` 객체를 반환할 수 있고, `Page` 객체는 조회된 데이터와 페이지 정보를 함께 갖게 된다.

```java
@GetMapping("/api/users")  
public ResponseEntity<Page<UserResponse>> getUserList(Pageable pageable) {  
    Page<UserResponse> responses = userService.getUserList(pageable);  
    return ResponseEntity.ok(responses);  
}
```
`Pageable`를 명시하면 아래와 같이 주소가 주어졌을때 **Controller에서 이를 인식하고 자동으로 매핑**

> `GET/user?size=5&page=3`

```java
public Page<User> getUserList(Pageable pageable) {  
    return userRepository.findAll(pageable);
}
```
- `JpaRepository`에서 이를 기준으로 `Pageable`만큼의 데이터를 반환 

## `@PageDefault` 어노테이션

```java
@GetMapping("/api/users")  
public ResponseEntity<Page<UserResponse>> getUserList(  
        @PageableDefault(size = 5) Pageable pageable)  
{  
    Page<UserResponse> responses = userService.getUserList(pageable);  
    return ResponseEntity.ok(responses);  
}
```

`@PageableDefault`를 활용하여 기본값을 설정 가능하다.
기본적으로는 `FallBackPage`를 기반으로 기본값 실행함.

> **DefaultPage vs FallbackPage**
> 
> DefaultPage의 경우, 개발자가 정한 기본 Page의 형식이다. 
> 별도로 어노테이션을 통해 설정해주지 않으면 FallbackPage의 설정으로 실행된다. 
> Fallback의 경우 적합한 방식이 없는 경우, 만일을 대비해 만들어 둔 설정이다. `PageableHandlerMethodArgumentResolverSupport`에는 FallbackPage만 설정되어 있다. 
> DefaultPage는 `@PageDefault` 어노테이션으로 설정한다.

## PageaRequest 클래스
- Spring Data JPA에서 제공하는 `Pageable` 구현체 중 하나로, 페이지 정보를 생성하는 클래스이다.
- 페이지 번호, 페이지당 항목 수, 정렬 정보를 이용하여 `Pageable` 인터페이스를 구현한다.
    - `page` : 조회할 페이지 번호(0부터 시작)
    - `size` : 한 페이지당 최대 항목 수
    - `sort` : 정렬 정보(생략 가능)
    - `direction` : 정렬 방향(ASC, DESC)
    - `properties` : 정렬 대상 속성명
- `PageRequest` 객체를 생성하고 JpaRepository 메서드 파라미터로 전달하면, `Page` 객체를 반환하므로, Pagination을 구현할 수 있다.

```java
// 생성자
PageRequest(int page, int size)
PageRequest(int page, int size, Sort sort)
PageRequest(int page, int size, Sort.Direction direction, String... properties)

// 객체 생성
PageRequest pageRequest = PageRequest.of(0, 10, Sort.by("id").ascending());
```


# 출처
[벨로그 : [JPA] Spring Data Jpa + Pageable로 Pagination 쉽게 구현하기](https://velog.io/@kimdy0915/JPA-Spring-Data-Jpa-Pageable%EB%A1%9C-Pagination-%EC%89%BD%EA%B2%8C-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0)

[테코블](https://tecoble.techcourse.co.kr/post/2021-08-15-pageable/)