# Stop-the-world 란?
GC를 실행하기 위해 JVM이 애플리케이션 실행을 멈추는 것이다.

Stop-the-world가 발생하면 GC를 실행하는 쓰레드를 제외한 나머지 쓰레드는 모두 작업을 멈춘다.  
GC 작업을 완료한 이후에야 중단했던 작업을 다시 시작한다.

GC 튜닝을 통해 stop-the-world 시간을 줄일 수 있다.
