### close 메서드로 닫아야 하는 자원

- InputStream, OutputStream, java.sql.Connection
- 그러나, 놓치기 쉬워 안전망으로 finalizer를 활용 ⇒ 신뢰 ↓

</br>

- 자원의 닫힘을 보장하기 위해 try-fianlly 사용

```java
static String firstLineOfFile(String path) threow IOException {
	BufferedReader br = new BufferedReader(new FileReader(path));
	try {
		return br.readLine();
	} finally {
		br.close();
	}
}
```

- 기기의 물리적 문제 발생 시 readLine 메서드가 예외를 던지고 close 메서드도 실패
    
    ⇒ close 메서드가 readLine 메서드의 예외를 덮어씀
    
    ⇒ 디버깅 어려워짐

</br>

- 또한, 자원이 둘 이상이면 지저분해짐

```java
static String copy(String src, String dst) threow IOException {
	InputStream in = new FileInputStream(src);
	try {
		OutputStream out = new FileOutputStream(dst);
		try {
			byte[] buf = new byte[BUFFER_SIZE];
			int n;
			while((n=in.read(buf)) >= 0)
				out.write(buf, 0, n);
		} finally {
			out.close();
		}
	} finally {
		in.close();
	}
}
```

</br>

- try-with-resources 로 해결
- 닫아야 하는 자원에 AutoCloseable 인터페이스를 구현해야 함
    - 단순 void를 반환하는 close 메서드

```java
static String firstLineOfFile(String path) threow IOException {
	try (BufferedReader br = new BufferedReader(
		new FileReader(path))){
		return br.readLine();
	}
}
```

```java
static String copy(String src, String dst) threow IOException {
	try (InputStream in = new FileInputStream(src);
		OutputStream out = new FileOutputStream(dst)){
		byte[] buf = new byte[BUFFER_SIZE];
		int n;
		while((n=in.read(buf)) >= 0)
			out.write(buf, 0, n);
	}
}
```

- 이때 readLine 메서드와 close 메서드 둘 다 예외가 발생하면, readLine 메서드의 예외만 기록됨

</br>

- try-with-resources에서도 catch 절을 쓸 수 있음
    - try 문 중접 없이 다수의 예외 처리 가능

```java
static String firstLineOfFile(String path, String defaultVal) threow IOException {
	try (BufferedReader br = new BufferedReader(
		new FileReader(path))){
		return br.readLine();
	} catch(IOException e){
		return defaultVal;
	}
}
```

</br>

⇒ 꼭 회수해야 하는 자원을 다룰 때는 try-with-resources를 써라. 더 짧고, 분명하고, 예외까지.
