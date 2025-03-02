자바 라이브러리에는 close 메서드를 호출해 직접 닫아줘야 하는 자원이 많다. <br />
ex) InputStream, OutputStream, .. 등

### 대안 1: finalizer
item 8에서 말했듯이, 안전하지 않음

### 대안 2: try-catch
- 자원을 하나 사용하는 경우엔 괜찮지만, 여러 개 사용하는 순간 depth가 늘어나면서 코드가 지저분해짐 <br />
- 예외는 try 블록과 finally 블록 모두에서 발생할 수 있다. -> 자원을 해제하는 close 메서드를 호출할 때

```java
static void copy(String src, String dst) throws IOException {
	InputStream in = new FileInputStream(src);
	try {
		OutputStream out = new FileOutputStream(dst);
		try {
			byte[] buf = new byte[BUFFER_SIZE];
			int n;
			while ((n = in.read(buf)) >= 0)
			out.write(buf, 0, n);
		} finally {
			out.close();
		}
	} finally {
		in.close();
	}
}
```

### 최종 Good 방안: try-with-resources
`try-with-resources`는 Java 7에서 도입된 구문으로, 자동으로 리소스를 닫아주는 기능을 제공한다.
- 이 구조를 사용하려면 AutoCloseable 인터페이스를 반드시 구현하는 것 추천
- 읽기 쉽고, 문제 파악 Good
```java
static void copy(String src, String dst) throws IOException {
	try (InputStream in = new FileInputStream(src);
		OutputStream out = new FileOutputStream(dst)) {
		byte[] buf = new byte[BUFFER_SIZE];
		int n;
		while ((n = in.read(buf)) >= 0)
		out.write(buf, 0, n);
	}
}
```

try-catch 절로 try 문을 더 중첩하지 않고도 다수의 예외 처리 가능
```java
public static String firstLineOfFile(String path) throw IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    } catch (Exception e) {
        return defaultVal;
    }
}
```

꼭 회수해야 하는 자원을 다룰 때는, try-finally 말고 try-with-resources 사용하자!
- 코드는 더 짧고 분명해지고, 예외 정보는 훨씬 유용
- try-finally로 작성하면 코드 지저분해지는 경우 있는데, try-with-resources로는 정확하고 쉽게 자원 회수할 수 있다.
