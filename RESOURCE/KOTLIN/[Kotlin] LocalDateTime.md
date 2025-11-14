# 코틀린 : LocalDateTime

## 유틸 커스텀 함수 예시
### 연속일 구하기
```kotlin
// 날짜 비교  
fun getDayGap(today: LocalDateTime, tomorrow: LocalDateTime): Long {  
    return abs(ChronoUnit.DAYS.between(today, tomorrow))  
}  
  
println(getDayGap(list[0], list[2])) // 2  
  
// 연속일 구하기  
fun getStreakDay(dateList: List<LocalDateTime>): Int {  
    var peakDay = INIT_STREAK_DAY;  
    for (i in 0..<dateList.size - 1) {  
        val isTomorrow = getDayGap(dateList[i], dateList[i + 1])  
  
        when {  
            isTomorrow == 0L -> peakDay  
            isTomorrow == 1L -> peakDay++  
            isTomorrow > 1L -> peakDay = INIT_STREAK_DAY  
        }  
    }  
    return peakDay  
}  
  
println(getStreakDay(list))
```
### 시간 구하기
```kotlin
// 시간 구하기  
fun getSolveTime(d1: LocalDateTime, d2: LocalDateTime): Long {  
    val duration = Duration.between(d1, d2)  
    return duration.seconds  
}