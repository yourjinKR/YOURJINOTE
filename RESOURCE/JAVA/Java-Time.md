
# java.time 패키지 등장 배경

자바에서 날짜와 시간을 다룰 때 [기존의 방식](Java-Date-Calendar.md)은 다음과 같은 문제점을 가지고 있었다.

- **불변 객체가 아님** → 멀티스레드 환경에서 안전하지 않음
- 월(month)이 0부터 시작하는 등 **직관적이지 않은 설계**
- 시간대(TimeZone) 처리의 복잡성
- API 일관성 부족

이러한 문제를 해결하기 위해 **Java 8**에서 JSR-310 표준을 기반으로 한 새로운 날짜/시간 API인 `java.time` 패키지가 도입되었다.
<br><br>
# 하위 패키지

해당 패키지는 아래와 같은 하위 패키지로 관리된다.

- java.time : 날짜와 시간을 다루는데 필요한 핵심 클래스를 제공
- java.time.chorono : 표준(ISO)이 아닌 달력 시스템을 위한 클래스들을 제공
- jvav.time.format : 날짜와 시간을 파싱하고, 형식화하기 위한 클래스들을 제공
- java.time.temporal : 날짜와 시간의 필드와 단위를 위한 클래스들을 제공
- java.time.zone 시간대와 관련된 클래스들을 제공
<br><br>
# 핵심 설계 철학

### 불변 객체 (Immutable)

- 모든 날짜/시간 클래스는 불변 객체
- 값 변경 시 **새 객체를 반환**

```java
LocalDate today = LocalDate.now();
LocalDate tomorrow = today.plusDays(1);
// today는 변하지 않음
```

### 역할 기반 클래스 분리
| 개념            | 클래스             |
| ------------- | --------------- |
| 날짜            | `LocalDate`     |
| 시간            | `LocalTime`     |
| 날짜 + 시간       | `LocalDateTime` |
| 날짜 + 시간 + 타임존 | `ZonedDateTime` |
| UTC 기준 시점     | `Instant`       |

<br><br>
# 주요 클래스

### LocalDate

- 날짜만 표현
- 시간, 타입존 정보 없음

```java
LocalDate date = LocalDate.of(2025, 12, 15);  
LocalDate now = LocalDate.now();  
  
int year = date.getYear();  
DayOfWeek dayOfWeek = date.getDayOfWeek();  
  
System.out.println(dayOfWeek); // MONDAY
```

> 생일, 기념일, 마감일 등에 적합하다.

### LocalTime

- 시간만 표현

```java
LocalTime time = LocalTime.of(14, 30);  
LocalTime now = LocalTime.now();  
  
int hour = time.getHour();  
System.out.println(hour); // 14
```

> 알람  시간 등에 적합하다.


### LocalDateTime

- 날짜 + 시간
- 타임존 없음

더 자세한 내용은 [여기](Java-LocalDateTime.md)를 확인

```java
LocalDateTime dateTime = LocalDateTime.of(2025, 12, 15, 9, 0);  
LocalDateTime now = LocalDateTime.now();  
  
System.out.println(dateTime); // 2025-12-15T09:00
```

> 서버 내부 로직, DB 저장용으로 자주 사용

### ZonedDateTime

- 날짜 + 시간 + 타임존

```java
ZonedDateTime seoulTime = ZonedDateTime.now(ZoneId.of("Asia/Seoul"));  
ZonedDateTime utcTime = ZonedDateTime.now(ZoneId.of("UTC"));  
  
System.out.println(seoulTime); // 2025-12-15T09:26:49.688150500+09:00[Asia/Seoul]
```

> 글로벌 서비스에서 필수

### Instant

- UTC 기준 [Timestamp](../CS/Timestamp.md)
- 내부 구조
	- epochSecond
	- nanoAdjustment

> 여기서 **epoch** 이란 단어를 번역하자면, 중요한 사건이나 변화가 있었던 시대(era)를 의미하는데, Unix와 POSIX 등의 시스템에서 날짜와 시간의 흐름을 나타낼 때 기준을 삼는 시간이다.

```java
public static void main(String[] args) {  
    Instant now = Instant.now();  
    long epochSecond = now.getEpochSecond();  
  
    System.out.println(now); // 2025-12-15T00:40:24.064130800Z  
}
```

> 로그 시간, 서버 간 시간 동기화를 다룰 때 주로 사용한다.

<br><br>
# 시간 비교

날짜를 비교할 때는 `Period`를 사용하고 시간을 비교할 때는 `Duration`을 사용한다.

### Period

```java
LocalDate start = LocalDate.of(2025, 1, 1);  
LocalDate end = LocalDate.of(2025, 12, 31);  
  
Period period = Period.between(start, end);  
int days = period.getDays();  
  
System.out.println(days); // 30
```

### Duration

```java
LocalTime t1 = LocalTime.of(10, 0);  
LocalTime t2 = LocalTime.of(12, 30);  
  
Duration duration = Duration.between(t1, t2);  
long minutes = duration.toMinutes();  
  
System.out.println(minutes); // 150
```
<br><br>
# 날짜/시간 포맷팅

기존 `java.text`에 있는 `SimpleDateFormat`은 스레드 안정성에 문제가 있었고 이를 `java.time.format`의 `DateTimeFormatter`로 해결했다.

```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm");  
  
String formatted = LocalDateTime.now().format(formatter);  
LocalDateTime parsed = LocalDateTime.parse("2025-12-15 09:00", formatter);  
  
System.out.println(formatted); // 2025-12-15 10:01  
System.out.println(parsed); // 2025-12-15T09:00
```
<br><br>
# 실무 활용

### JPA

- **Entity**: `LocalDateTime` 사용 권장

```java
@Column(nullable = false)
private LocalDateTime createdAt;
```

> JPA 2.2 부터  java.time 타입을 기본적으로 표준 지원한다.

### Spring

- **Controller 요청/응답**: jackson이 `LocalDateTime`을 ISO-8601 기본 포맷 자동 처리

```
// 기본 ISO-8601
2025-12-15T09:00:00
```

```java
@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
private LocalDateTime createdAt;
```

> 다음과 같은 포맷 커스터마이징은 DTO 계층에서 수행하자

### DB

- **DB 컬럼**: `TIMESTAMP`와 매핑


> 또한 타임존은 지역에 따라 연산이 필요하기에 DB는 고정된 LocalDateTime으로 관리하고,  
> UI 단계에서 타임존을 적용하는 방향성으로 개발하자.

<br><br>
# 출처

[위키백과 - timestamp](https://ko.wikipedia.org/wiki/%ED%83%80%EC%9E%84%EC%8A%A4%ED%83%AC%ED%94%84)  
[네이버 블로그 - epoch time의 유래](https://m.blog.naver.com/techtrip/221672212122)  
[티스토리 - SimpleDateFormat을 사용하면 안되는 이유](https://woonys.tistory.com/236)  