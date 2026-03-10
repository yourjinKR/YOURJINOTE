
## Eager Initialization 

```java
public class Singleton {  
  
    private static final Singleton instance = new Singleton();  
  
    private Singleton() { }  
  
    public static Singleton getInstance() {  
        return instance;  
    }  
}
```

- 인스턴스를 클래스 로딩 단계에서 생성
- 만약 해당 인스턴스를 사용하지 않더라도 로딩 단계에서 생성하기에 주의가 필요
	- 리소스가 비교적 적은 인스턴스에 대해 해당 방법을 사용하도록 권장
- 예외처리 불가능

## Static Block Initialization

```java
class StaticBlockSingleton {  
  
    private static StaticBlockSingleton instance;  
  
    private StaticBlockSingleton(){}  
  
    //static block initialization for exception handling  
    static{  
        try{  
            instance = new StaticBlockSingleton();  
        }catch(Exception e){  
            throw new RuntimeException("싱글톤 생성 중 에러 발생");  
        }  
    }  
  
    public static StaticBlockSingleton getInstance(){  
        return instance;  
    }  
}
```

- Eager Initialization 방식과 같이 클래스 로딩 시점에 인스턴스를 생성
- 예외처리 가능

## Lazy Initialization

```java
public class LazySingleton {  
  
    private static LazySingleton instance;  
  
    private LazySingleton(){}  
  
    public static Singleton getInstance(){  
        if(instance == null){  
            instance = new LazySingleton();  
        }  
        return instance;  
    }  
}
```

- 인스턴스 낭비 문제는 어느정도 해결
- 
