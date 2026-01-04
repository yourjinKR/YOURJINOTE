# JIT 컴파일러

JIT 컴파일러는 런타임시 바이트 코드를 원시 시스템 코드로 컴파일하여 Java 애플리케이션 성능을 향상시키는 런타임 환경의 컴포넌트이다.

![java-jvm](https://blog.kakaocdn.net/dna/b1w4cW/btrIK7IEXEe/AAAAAAAAAAAAAAAAAAAAABuuH8Dy892Tx8lNatVstFQ0HLtyQ2R-5cXGAYryuFn1/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1764514799&allow_ip=&allow_referer=&signature=M2rBm5Eiwhw%2B88PKY9zlX16zuU0%3D)

기존의 자바는 인터프리터 방식으로 명령어를 하나씩 실행하게끔 이루어져 있어 실행 속도가 느렸다.  
하지만 하드웨어가 발전하면서 자바 컴파일러도 JIT 컴파일러 방식으로 개선되어 속도적인 측면에서 상당한 개선을 이루었다.  
  
JIT 컴파일러는 같은 코드를 매번 해석하지 않고, 실행할 때 컴파일을 하면서 해당 **코드를 캐싱** 한다. 이후에는 바뀐 부분만 컴파일하고 나머지는 캐싱된 코드를 사용한다.  **동적 번역(dynamic translation)** 이라고도 불리는 이 기법은 이전의 자바 해석기(Java interpreter) 방식보다 성능이 10배 ~ 20배 정도 더 좋다.

출처: [https://inpa.tistory.com/entry/JAVA-☕-JDK-JRE-JVM-개념-구성-원리-💯-완벽-총정리](https://inpa.tistory.com/entry/JAVA-%E2%98%95-JDK-JRE-JVM-%EA%B0%9C%EB%85%90-%EA%B5%AC%EC%84%B1-%EC%9B%90%EB%A6%AC-%F0%9F%92%AF-%EC%99%84%EB%B2%BD-%EC%B4%9D%EC%A0%95%EB%A6%AC) [Inpa Dev 👨‍💻:티스토리]