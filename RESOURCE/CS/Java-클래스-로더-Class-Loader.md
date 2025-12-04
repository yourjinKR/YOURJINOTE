# 클래스 로더란?
클래스 로더는 런타임 중에 동적으로 Java 클래스를 JVM의 메소드 영역에 로드하는 역할을 수행한다.
클래스 로더에는 로딩, 링크, 초기화 단계로 나뉘어져 있다.
## 로딩
자바 바이트 코드(.class)를 메서드 영역에 저장한다.
각 자바 바이트 코드는 JVM에 의해 메서드 영역에 다음 정보들을 저장한다.
- 로드된 클래스를 비롯한 그의 부모 클래스 정보
- 클래스 파일이나 Class, Interface, Enum의 관련 여부
- 변수나 메소드 등의 정보
## 링크
링크는 아래와 같이 세가지 단계를 거친다.
### 검증 (Vertification)
읽어 들인 클래스가 자바 언어 명세 및 JVM 명세에 명시된 대로 잘 구성되어 있는지 검사한다.
### 준비 (Pereparation)
클래스가 필요로 하는 메모리를 할당하고, 클래스에서 정의된 필드, 메소드, 인터페이스를 나타내는 데이터 구조를 준비한다. 
### 분석 (Resolution)
심볼릭 메모리 레퍼런스를 메소드 영역에 있는 실제 레퍼런스로 교체한다.

> **심볼릭 레퍼런스란?**
> 심볼릭 레퍼런스는 자바와 같은 고급 프로그래밍 언어에서 사용되는 개념이다. **프로그램 내에서 클래스, 메서드, 필드 등을 직접적으로 식별하는 데 사용되는 이름이나 식별자**를 말한다. 심볼릭 레퍼런스는 주로 컴파일된 바이트코드 내에 존재하며, 프로그램 실행 시점에 실제 메모리 주소(물리적 주소)로 변환된다. 이를 통해 프로그램이 메모리 내에서 객체나 클래스에 접근할 수 있게 된다.

## 초기화
클래스 변수들을 적절한 값으로 초기화 한다. 즉, static 필드들이 설정된 값으로 초기화한다.
# 클래스 로더의 종류
클래스 로더는 한 둘이 아니다.
![Pasted image 20251127174301](../../GALLERY/Pasted%20image%2020251127174301.png)
## 부트 스트랩 클래스 로더 (Bootstrap Class Loader)
JVM 시작 시 **가장 최초로 실행되는 클래스 로더**이다.
부트스트랩 클래스 로더는 자바 클래스를 로드하는 것이 아닌, 자바 클래스를 로드할 수 있는 자바 자체의 클래스 로더와 최소한의 자바 클래스만을  로드한다.
> java.lang.Object, Class, ClassLoader만을 로드함.
### Java 8
`jre/lib/rt.jar` 및 기타 핵심 라이브러리와 같은 JDK의 내부 클래스를 로드한다.
### Java 9 이후
더 이상 `/re.jar`이 존재하지 않으며, `/lib` 내에 모듈화되어 포함됐다. 이제는 정확하게 ClassLoader 내 최상위 클래스들만 로드한다.
## 플랫폼 클래스 로더, 확장 클래스 로더 (Extension Class Loader)
확장 클래스 로더는 부트스트랩 클래스 로더를 부모로 갖는 클래스 로더로서, 확장 자바 클래스들을 로드한다. java.ext.dirs 환경 변수에 설정된 디렉토리의 클래스 파일을 로드하고, 이 값이 설정되어 있지 않은 경우 `${JAVA_HOME}/jre/lib/ext` 에 있는 클래스 파일을 로드한다.
### Java 8
URLClassLoader를 상속하며, jre/lib/ext 내 모든 클래스를 로드한다.
### Java 9 이후
Platform Loader로 변경되었으며, URLClassLoader가 아닌 BuiltinClassLoader를 상속한다. Inner Static 클래스로 구현되어 있다.
## 어플리케이션, 시스템 클래스 로더 (System Class Loader)
자바 프로그램 실행 시 지정한 Classpath에 있는 클래스 파일 혹은 jar에 속한 클래스들을 로드한다. 
쉽게 말하자면, 우리가 만든 .class 확장자 파일을 로드한다.
# 클래스 로더의 동작 방식
다음과 같은 방식으로 로드를 수행한다.
1. JVM의 메소드 영역에 클래스가 로드되어 있는지 확인한다. 만일 로드되어 있는 경우 해당 클래스를 사용한다.
2. 메소드 영역에 클래스가 로드되어 있지 않을 경우, 시스템 클래스 로더에 클래스 로드를 요청한다.
3. 시스템 클래스 로더는 확장 클래스 로더에 요청을 위임한다.
4. 확장 클래스 로더는 부트스트랩 클래스 로더에 요청을 위임한다.
5. 부트스트랩 클래스 로더는 부트스트랩 Classpath(JDK/JRE/LIB)에 해당 클래스가 있는지 확인한다. 클래스가 존재하지 않는 경우 확장 클래스 로더에게 요청을 넘긴다.
6. 확장 클래스 로더는 확장 Classpath(JDK/JRE/LIB/EXT)에 해당 클래스가 있는지 확인한다. 클래스가 존재하지 않을 경우 시스템 클래스 로더에게 요청을 넘긴다.
7. 시스템 클래스 로더는 시스템 Classpath에 해당 클래스가 있는지 확인한다. 클래스가 존재하지 않는 경우 `ClassNotFoundException`을 발생시킨다.
# 클래스 로더가 지켜야 할 세가지 원칙
### 위임 원칙
- 클래스 로더는 클래스 또는 리소스를 찾기 위해 요청을 받았을 때, 상위 클래스 로더에게 책임을 위임하는 위임 모델을 따른다.
- 위에서 설명한 클래스 로더 동작 방식을 보면, 위임 법칙을 따른다는 것을 확인할 수 있다.
### 가시 범위 원칙
- 하위 클래스 로더는 상위 클래스 로더가 로드한 클래스를 볼 수 있지만, 반대로 상위 클래스 로더는 하위 클래스 로더가 로드한 클래스를 알 수 없다.
- 이로 인해 java.lang.Object 클래스 등 상위 클래스 로더에서 로드한 클래스도 하위 클래스 로더인 시스템 클래스 로더 등에서 사용할 수 있다.
### 유일성의 원칙
- 하위 클래스 로더가 상위 클래스 로더에게 로드한 클래스를 다시 로드하지 않아야 한다는 원칙이다.
- 위임 원칙에 의해서 위쪽으로 책임을 위임하기 때문에 고유한 클래스를 보장할 수 있다.
# 동적 클래스 로딩
자바의 클래스 로딩은 클래스 참조 시점에 JVM에 코드가 링크되고, 실제 런타임 시점에 로딩되는 동적 로딩을 거친다. 런타임에 동적으로 클래스를 로딩한다는 것은 **JVM이 미리 모든 클래스에 대한 정보를 메소드 영역에 로딩하지 않는다**는 것을 의미한다.

동적으로 클래스를 로딩하는 방식은 두 가지가 있다.
## 로드 타임 동적 로딩 (Load-time Dynamic Loading)
하나의 클래스를 로딩하는 과정에서 동적으로 다른 클래스를 로딩하는 것을 로드 타임 동적 로딩이라고 한다.
### 예시 코드
```java
public class HelloWorld { 
	public static void main(String[] args) { 
		System.out.println("안녕하세요!"); 
	} 
}
```
위 코드의 경우 다음과 같이 동작한다.

1. JVM이 시작되고 부트스트랩 클래스 로더가 생성된 후에 모든 클래스가 상속받고 있는 Object 클래스를 읽어온다.
2. 클래스 로더는 명령 행에서 지정한 HelloWorld 클래스를 로딩하기 위해, HelloWorld.class 파일을 읽는다.
3. HelloWorld 클래스를 로딩하는 과정에서 필요한 클래스인 java.lang.String과 java.lang.System을 로딩한다.

## 런타임 동적 로딩 (Run-time Dynamic Loading)
클래스를 로딩할 때가 아닌, 코드를 실행하는 순간에 클래스를 로딩하는 것을 런타임 동적 로딩이라고 한다.
### 예시 코드
```java
public class RuntimeLoading { 
	public static void main(String[] args) { 
		try { 
			Class cls = Class.forName(args[0]); 
			Object obj = cls.newInstance(); 
			Runnable r = (Runnable) obj; 
			r.run(); 
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}
```
위 코드에서 `Class.forName(className)` 은 파라미터로 받은 className에 해당하는 클래스를 로딩한 후에 객체를 반환하며, 다음과 같이 동작한다.

1. `Class.forName()` 메소드가 실행되기 전까지는 RuntimeLoading 클래스에서 어떤 클래스를 참조하는지 알 수 없다.
2. 따라서 RuntimeLoading 클래스를 로딩할 때는 어떤 클래스도 읽어오지 않고, RuntimeLoading 클래스의 `main()` 메소드가 실행되고 `Class.forName(args[0])` 를 호출하는 순간에 비로소 `args[0]` 에 해당하는 클래스를 로딩한다.

# 출처
클래스의 로딩, 초기화 시점에 대한 이야기와 예상 면접 질문 및 답변은 링크를 참고하자 
[f-lab](https://f-lab.kr/insight/understanding-jvm-class-loader-20250529?gad_source=1&gad_campaignid=22368870602&gbraid=0AAAAACGgUFdfKzkzzvnlSSkanha9oUlBE&gclid=CjwKCAiA86_JBhAIEiwA4i9JuynLqNNXbnPWq0WQA474xiQxdWCQzbx4TDzy44KdlxbtO742gca9RhoCyYgQAvD_BwE)

[](https://steady-coding.tistory.com/593#google_vignette)