# Enum


# EnumMap


# 예시
## ExceptionMessage
```kotlin
enum class ErrorMessage(private val rawMessage: String) {
    ERROR("에러")

    private val prefix = "[ERROR]"
    val message: String
        get() = "$prefix $rawMessage"
}
```


## `List<Enum>`을 `EnumMap<Enum, 갯수>`로 변환

```kotlin
val quizRecordList: List<QuizRecord> = quizRecordRepository.findAll() 
```
### 방법1 : 기본
```kotlin
val timeRanges = quizRecordList.map { it.getSolvedTimeRange() }
val counted = timeRanges.groupingBy { it }.eachCount()

val fullMap = EnumMap<TimeRange, Int>(TimeRange::class.java).apply {
    TimeRange.entries.forEach { range ->
        this[range] = counted[range] ?: 0
    }
}
```
### 방법2 : associateWith 활용
```kotlin
val timeRanges = quizRecordList.map { it.getSolvedTimeRange() }
val counted = timeRanges.groupingBy { it }.eachCount()

val fullMap = EnumMap<TimeRange, Int>(TimeRange::class.java).apply {
    putAll(TimeRange.entries.associateWith { counted[it] ?: 0 })
}
```
### 방법3 : 한줄로 처리
```kotlin
val activePlayTimeRange = EnumMap(  
    TimeRange.entries.associateWith { range ->  
        quizRecordList  
            .map { it.getSolvedTimeRange() }  
            .groupingBy { it }  
            .eachCount()[range] ?: 0  
    }  
)