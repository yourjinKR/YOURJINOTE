# String

```java
String name = "유어진";
```

- 문자열의 본질은 **문자 배열**이며 문자열은 인코딩 규칙에 영향을 받음 
	- `char[]`, `String` 
	- 문자 배열은 겹따옴표를 이용한 리터럴 표기 가능 
	- Java 9이후 `char[]`에서 `byte[]`로 변경 ⭐
- `String` 클래스는 **불변 클래스**이며 **논리적 의미로 기본 형식**에 속하는 특성을 보임 
- 덧셈 연산의 결과로 **임시 객체**가 생기는 문제가 있음 
- 큰 문자열을 다룰 경우 효율이 더 떨어짐

> 문자열은 원시 타입, 객체, 배열의 특성을 모두 지니고 있음.  
> 또한 본질은 클래스(객체)이기 때문에  [Heap](Java-Runtime-Data-Areas.md)에 저장된다.

## String은 왜 불변으로 설계 되었는가?
### 데이터 캐싱
JVM에서 String Constant Pool이라는 독립적인 영역을 만들고 문자열을 상수화하여 다른 변수 혹은 객체들과 공유하도록 설계했다.
### 동기화 문제
멀티 스레드 환경에서 동기화 문제가 발생하지 않기 위해
### 보안
특정 메모리 주소의 문자열의 값을 변경할 수 있다면 외부로부터 보안 측면에서 취약하기에 이를 방지
# Java가 문자열을 관리하는 구조

- 모든 문자열은 상수 풀로 관리
- C/C++로 개발된 PE 파일과 유사한 구조

> exe 파일의 내부에 문자열이 포함  
> 실행 코드가 저장되는 정적메모리 영역에 문자열 상수 저장  
> 같은 문자열 상수에 대한 포인터의 주소는 모두 동일

- 코드상 존재하는 모든 문자열 상수는 Class가 로딩될 때 Runtime constant pool에 등록된 후 힙 영역에 존재하는 String constant pool에도 추가

![Pasted image 20251210230000](GALLERY/Pasted%20image%2020251210230000.png)

### `String.intern()`

```java
public native String intern();
```

> `native` : C/C++로 구현된 메서드에 붙이는 예약어

- 문자열 상수 풀에서 문자열을 조회하고 반환
	- 문자열 상수와 일치하는 문자열을 상수 풀에서 검색
	- 찾으면 이미 **생성되어 있는 String 인스턴스** 반환
	- 없으면 새로 **String 객체 생성 후 풀에 추가**하고 반환

> 문자열 상수 풀은 힙 영역에 속하며 GC 대상이다.

# 문자열 비교
- 상등 연산으로 두 String 클래스 인스턴스를 비교할 경우 심각한 논리적 문제가 발생할 수 있다.
	- 문자열 상수 풀
	- JVM이 클래스 로딩 시 미리 인스턴스 생성
- `equals()` 메서드를 활용하거나 `compareTo()` 메서드로 문자열 비교하는 것이 적절

```java
String s1 = "Hello";
String s2 = "Hello"; 
System.out.println(s1 == s2) // true
System.out.println(s1.equals(s2)); // true

String s3 = new String("World");
String s4 = new String("World");
System.out.println(s3 == s4); // false
System.out.println(s3.equals(s4)); // true
System.out.println(s3.compareTo(s4) == 0); // true
```

# 문자열 주요 메서드
[인프런 - 널널한 개발자](https://www.inflearn.com/course/%EB%8F%85%ED%95%98%EA%B2%8C-%EC%8B%9C%EC%9E%91%ED%95%98%EB%8A%94-java-part2/dashboard)

[점프 투 자바](https://wikidocs.net/205)

[티스토리 - String 타입 이해하기](https://inpa.tistory.com/entry/JAVA-%E2%98%95-String-%ED%83%80%EC%9E%85-%ED%95%9C-%EB%88%88%EC%97%90-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-String-Pool-%EB%AC%B8%EC%9E%90%EC%97%B4-%EB%B9%84%EA%B5%90)