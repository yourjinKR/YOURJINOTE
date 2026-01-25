# Lock

[임계 영역](Critical-Section-임계-영역.md)을 하나의 스레드에서만 접근할 수 있도록 보호하는 방식

## 전략

### Fine-grained Lock

임계영역마다 Lock을 걸어서 제한

### Coarse-grained Lock

여러 개의 임계 영역을 묶어 Lock을 걸어 제한