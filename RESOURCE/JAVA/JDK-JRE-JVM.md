# JDK vs JRE vs JVM

- JDK (Java Development Kit)
- JRE (Java Runtime Enviroment)
- JVM (Java Virtual Machine)

![jdk_jre_jvm_image](../../GALLERY/jdk_jre_jvm_image.png)

## [JDK](JDK.md)

자바를 개발 시 필요한 라이브러리들과 함께 다양한 개발 도구를 포함

## JRE

Java로 프로그램을 직접 개발하려면 JDK가 필요하고, 컴파일된 Java를 실행시키기 위해서는 JRE가 필요

## [JVM](JVM.md)

자바 애플리케이션을 실행하는 가상머신

## 비교

|         | JDK                       | JRE                      | JVM                                    |
| ------- | ------------------------- | ------------------------ | -------------------------------------- |
| 목적      | 자바 애플리케이션 개발에 사용됩니다.      | 자바 애플리케이션을 실행하는 데 사용됩니다. | 자바 바이트코드를 실행합니다.                       |
| 플랫폼 종속성 | 플랫폼에 따라 다릅니다(운영체제별).      | 플랫폼에 따라 다릅니다(운영체제별).     | JVM은 운영체제에 종속적이지만, 바이트코드는 플랫폼에 독립적입니다. |
| 포함되는 사항 | JRE + 개발 도구(javac, 디버거 등) | JVM + 라이브러리(예: rt.jar)   | 클래스 로더, JIT 컴파일러, 가비지 컬렉터              |
| 사용 사례   | 자바 코드 작성 및 컴파일            | 시스템에서 Java 애플리케이션 실행     | 바이트코드를 네이티브 머신 코드로 변환합니다.              |


# 출처
[geeksforgeeks](https://www.geeksforgeeks.org/java/differences-jdk-jre-jvm/)