## 수정 할 부분

- [ ] DTO와 Entity 분리
- [ ] 덮어쓰기 방식으로 만든 API를 부분 수정으로 변경
- [ ] RoutineSet 또한 Exercise의 MetricType 처럼 분리
- [ ] 루틴 리스트 조회시 로그에서 엄청나게 긴 쿼리에 대해 알아보기

## 인사이트

[멱등성](https://www.youtube.com/watch?v=6znXX1tnIAs&t=522s)  

> 재요청 중복 처리에 의한 결과가 치명적인 곳에서 적용 가능 ex) 결제, 구독, 유료 서비스 요청 (AI 서비스)