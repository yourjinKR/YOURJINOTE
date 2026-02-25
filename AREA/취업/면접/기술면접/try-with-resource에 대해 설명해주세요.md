[Java-try-with-resource](RESOURCE/JAVA/Java-try-with-resource.md)는 리소스를 다뤄야 할때 일일히 `close()` 작업을 수행해야 할때 더욱 간결한 문법을 제공 합니다.  

```java
try (BufferedReader br = new BufferedReader(new FileReader("test.txt"))) {
    System.out.println(br.readLine());
} catch (IOException e) {
    e.printStackTrace();
}
```

해당 문법은 `AutoClosable`를 상속하는 클래스에만 적용됩니다.