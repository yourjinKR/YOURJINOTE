좋습니다 👍  
이제 스트림 학습의 **정점 구간**,  
👉 **63. 다운스트림 컬렉터 1 (groupingBy / partitioningBy)** 로 들어갑니다.

여기부터는 **“스트림 = SQL 대체”** 라고 느끼게 되는 파트입니다.

---

# 🧩 63. 다운스트림 컬렉터 1

`groupingBy` / `partitioningBy`

## 🎯 이 단계의 목표

- `groupingBy`의 **구조를 정확히 이해**
    
- 다운스트림 컬렉터가 **왜 필요한지** 체감
    
- SQL의 `GROUP BY` / `CASE WHEN`을 스트림으로 구현
    

---

## 1️⃣ 다운스트림 컬렉터란?

> **그룹핑된 각 그룹에 대해 또 한 번 collect를 수행하는 구조**

### 구조 한 줄 요약

```java
groupingBy(분류 기준, 다운스트림 컬렉터)
```

---

## 2️⃣ `groupingBy()` 기본 형태

### 예제: 나이 기준 그룹핑

```java
Map<Integer, List<Member>> map =
    members.stream()
           .collect(Collectors.groupingBy(Member::getAge));
```

📌 결과 구조

```java
Map<나이, List<회원>>
```

---

## 3️⃣ 다운스트림 컬렉터 적용 (⭐ 핵심)

### 예제: 나이별 인원 수

```java
Map<Integer, Long> countByAge =
    members.stream()
           .collect(Collectors.groupingBy(
               Member::getAge,
               Collectors.counting()
           ));
```

📌 구조

```java
Map<나이, 인원 수>
```

👉 **List 대신 원하는 결과로 바로 수집**

---

## 4️⃣ 자주 쓰는 다운스트림 패턴

### ① 그룹별 합계

```java
Map<Integer, Integer> sumAge =
    members.stream()
           .collect(Collectors.groupingBy(
               Member::getAge,
               Collectors.summingInt(Member::getAge)
           ));
```

---

### ② 그룹별 평균

```java
Map<Integer, Double> avgAge =
    members.stream()
           .collect(Collectors.groupingBy(
               Member::getAge,
               Collectors.averagingInt(Member::getAge)
           ));
```

---

### ③ 그룹별 최대값

```java
Map<Integer, Optional<Member>> oldestByAge =
    members.stream()
           .collect(Collectors.groupingBy(
               Member::getAge,
               Collectors.maxBy(
                   Comparator.comparingInt(Member::getAge)
               )
           ));
```

---

## 5️⃣ `partitioningBy()` – 참/거짓 분기

> **조건(boolean) 기준으로 딱 두 그룹**

### 예제: 성인/미성년 분리

```java
Map<Boolean, List<Member>> map =
    members.stream()
           .collect(Collectors.partitioningBy(
               m -> m.getAge() >= 20
           ));
```

📌 결과

```java
true  -> 성인
false -> 미성년
```

---

### 다운스트림 적용

```java
Map<Boolean, Long> count =
    members.stream()
           .collect(Collectors.partitioningBy(
               m -> m.getAge() >= 20,
               Collectors.counting()
           ));
```

---

## 6️⃣ groupingBy vs partitioningBy (면접 단골)

|구분|groupingBy|partitioningBy|
|---|---|---|
|기준|모든 값|boolean|
|그룹 수|가변|항상 2|
|반환|Map<K, V>|Map<Boolean, V>|

---

## 7️⃣ 실무에서 자주 보는 형태

### SQL

```sql
SELECT age, COUNT(*)
FROM member
GROUP BY age;
```

### Stream

```java
members.stream()
       .collect(Collectors.groupingBy(
           Member::getAge,
           Collectors.counting()
       ));
```

👉 **스트림 = 인메모리 집계 SQL**

---

## 8️⃣ 면접 지뢰 질문 ⚠️

> Q. groupingBy의 기본 반환 타입은?

❌ 틀린 답

> “Map<K, V>”

✅ 정확한 답

> “Map<K, List>이며, 다운스트림을 지정하면 V가 바뀝니다.”

---

## 9️⃣ 63번 핵심 요약

- groupingBy는 **그룹핑 + 수집**
    
- 다운스트림 컬렉터로 결과 형태 제어
    
- partitioningBy는 boolean 전용
    
- 이 파트는 **실무 집계의 핵심**
    

---

다음 목차는 **64. 다운스트림 컬렉터 2**입니다.  
여기서는:

- `mapping`
    
- `reducing`
    
- **다단계 grouping**
    
- 실무에서 가장 어려운 스트림 패턴
    

을 다룹니다.

👉 **64번으로 바로 진행할까요?**


# 🧩 64. 다운스트림 컬렉터 2

`mapping / reducing / 다단계 grouping`

## 🎯 이 단계의 목표

- 다운스트림에서 **결과 구조를 자유자재로 설계**
    
- “groupingBy 안에 groupingBy” 같은 **고급 패턴 이해**
    
- 실무·면접에서 가장 헷갈리는 부분 정리
    

---

## 1️⃣ `mapping()` – 그룹 내부에서 값 변환

> **groupingBy 내부에서는 map을 못 쓴다**  
> → 그 역할을 대신하는 게 `mapping`

### 예제: 나이별 이름 목록

```java
Map<Integer, List<String>> namesByAge =
    members.stream()
           .collect(Collectors.groupingBy(
               Member::getAge,
               Collectors.mapping(
                   Member::getName,
                   Collectors.toList()
               )
           ));
```

📌 구조

```java
Map<나이, 이름 리스트>
```

---

## 2️⃣ `mapping()` vs `map()` 차이 (면접 포인트)

|구분|map|mapping|
|---|---|---|
|위치|스트림 중간 연산|다운스트림 컬렉터|
|용도|요소 변환|**그룹 내부 값 변환**|

---

## 3️⃣ 다단계 grouping (⭐⭐ 실무 핵심)

### 예제: 나이 → 성별 → 인원 수

```java
Map<Integer, Map<Gender, Long>> result =
    members.stream()
           .collect(Collectors.groupingBy(
               Member::getAge,
               Collectors.groupingBy(
                   Member::getGender,
                   Collectors.counting()
               )
           ));
```

📌 구조

```java
Map<
  나이,
  Map<
    성별,
    인원 수
  >
>
```

👉 SQL로 치면 **GROUP BY age, gender**

---

## 4️⃣ 다단계 + mapping 조합

### 예제: 나이별 이름 Set

```java
Map<Integer, Set<String>> namesByAge =
    members.stream()
           .collect(Collectors.groupingBy(
               Member::getAge,
               Collectors.mapping(
                   Member::getName,
                   Collectors.toSet()
               )
           ));
```

---

## 5️⃣ `reducing()` 다운스트림 (고급 · 주의)

### 예제: 나이별 총 나이 합

```java
Map<Integer, Integer> sumByAge =
    members.stream()
           .collect(Collectors.groupingBy(
               Member::getAge,
               Collectors.reducing(
                   0,
                   Member::getAge,
                   Integer::sum
               )
           ));
```

📌 주의

- `summingInt`로 대체 가능하면 **reducing 지양**
    
- reducing은 **커스텀 누적이 필요할 때만**
    

---

## 6️⃣ Optional 제거 패턴 (`collectingAndThen`)

```java
Map<Integer, Member> oldestByAge =
    members.stream()
           .collect(Collectors.groupingBy(
               Member::getAge,
               Collectors.collectingAndThen(
                   Collectors.maxBy(
                       Comparator.comparingInt(Member::getAge)
                   ),
                   Optional::get
               )
           ));
```

📌 결과

```java
Map<나이, 가장 나이 많은 회원>
```

⚠️ Optional이 비어 있지 않다는 보장이 있을 때만 사용

---

## 7️⃣ 실무 설계 사고 방식

> ❓ grouping 결과에서
> 
> - List가 필요한가?
>     
> - 개수만 필요한가?
>     
> - 변환된 값이 필요한가?
>     

👉 그에 맞는 **다운스트림 컬렉터 선택**

---

## 8️⃣ 면접 지뢰 질문 ⚠️

> Q. groupingBy 안에서 map을 쓰면 안 되나요?

❌ 틀린 답

> “쓸 수 없습니다”

✅ 정확한 답

> “스트림 중간 연산인 map은 사용할 수 없고,  
> 대신 Collectors.mapping을 사용해야 합니다.”

---

## 9️⃣ 64번 핵심 요약

- mapping = 그룹 내부 map
    
- 다단계 grouping은 Map 중첩 구조
    
- reducing은 최후의 수단
    
- collectingAndThen으로 결과 후처리 가능
    
- 이 파트는 **스트림 최고 난이도**
    

---

다음이 **목차의 마지막**입니다 🎯  
👉 **65. 정리 (스트림 전체 구조 복습)**

여기서는:

- 스트림 전체 흐름 한 장으로 정리
    
- 실무/면접 관점 요약
    
- “언제 스트림을 쓰고, 언제 안 쓰는가”
    

를 다룹니다.

**65번으로 바로 진행할까요?**