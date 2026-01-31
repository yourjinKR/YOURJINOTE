String 객체는 한번 생성되면 할당된 공간이 변하지 않지만 StringBuffer나 [StringBuilder](../../../RESOURCE/JAVA/Java-StringBuilder.md)의 경우 객체의 공간이 부족해지는 경우 버퍼의 크기를 유연하게 늘려줍니다. 이러한 특징을 일컬어 String은 불변하고 StringBuffer와 StringBuilder는 가변하다라고 합니다.

## StringBuffer-vs-StringBuilder

StringBuffer는 `synchronized` 키워드를 사용하여 멀티스레딩 환경에서 동기화를 지원합니다.  
반면 StringBuilder는 비교적 성능은 좋으나 단일 스레드에서 사용하도록 설계되어있습니다.

<br>

# 출처

[String과 StringBuffer, StringBuilder의 차이점](https://coding-factory.tistory.com/546)
