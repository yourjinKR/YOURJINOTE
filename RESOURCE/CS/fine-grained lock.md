## fine-grained-lock

- 데이터를 아주 작은 단위로 쪼개어 각각 [잠그는 방식](Lock.md)
- 여러 사용자가 서로 다른 데이터를 동시에 수정할 수 있어 **동시성 높음**
- lock/unlock이 자주 발생하며 오버헤드가 크다
- [데드락](Deadlock.md) 위험이 높음
- 데이터베이스의 Row Lock, 자바의 `ConcurrentHashMap`이 대표적 예시