## Spring Bean의 개념과 생명주기에 대해 설명해주세요

[Spring Bean](../../../../RESOURCE/SPRING/Spring%20Bean.md)이란 Spring IoC Container에서 관리되는 객체들을 말합니다.  
Spring Bean은 크게 객체 생성 → 의존성 주입 → 초기화 → 사용 → 소멸 과정을 거칩니다.  


### 꼬리 질문

- **Q: 왜 생성자에서 초기화 로직을 안 넣고 `@PostConstruct`를 쓰나요?**
    
    - **A:** 생성자 시점에는 의존성 주입(DI)이 완료되지 않았기 때문입니다. 주입된 빈을 안전하게 사용하려면 모든 주입이 끝난 '초기화' 단계에서 실행해야 합니다.
        
- **Q: 빈 스코프(Scope) 종류에 대해 아는 대로 말해보세요.**
    
    - **A:** 가장 기본인 `singleton`, 요청 시마다 생성되는 `prototype`, 그리고 웹 환경에서의 `request`, `session`, `application` 스코프 등이 있습니다.
        
- **Q: 빈을 프록시로 만드는 시점은 정확히 언제인가요?**
    
    - **A:** `BeanPostProcessor`의 `postProcessAfterInitialization` 단계에서 타깃 빈을 프록시 객체로 바꿔치기하여 반환합니다.