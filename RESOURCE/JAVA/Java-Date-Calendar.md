
# Date

- 날짜와 시간을 다룸
- JDK 1.0부터 제공된 클래스

# Calendar

- 기존에 `Date`의 기능이 부족하여 추후 `Calendar` 클래스가 출시
- 많은 부분이 개선되었지만 몇몇 단점 존재
- 해당 클래스는 추상 클래스이기에 아래와 같은 클래스를 구현하여 사용했음 (사용지역에 따른 구분)
	- `BuddhistCalendar` : 태국
	- `GregorianCalendar` : 나머지 지역


# Date와 Calendar의 변환 

Calendar를 추가하면서 Date는 `deprecated`되었음.  
각 객체를 변환할 때는 아래와 같이 사용한다.

### Calendar -> Date

```java
Calendar cal = Calendar.getInstance();
Date d = new Date(cal.getTimeInMillis());
```

### Date -> Calendar

```java
Date d = new Date();
Calendar cal = Calendar.getInstance();
cal.setTime(d);
```


