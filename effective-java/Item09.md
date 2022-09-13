## ITEM09 try-finally 보다는 try-with-resources를 사용하라

> 
> 꼭 회수해야 하는 자원을 다룰 때는 try-finally 말고 try-with-resources를 사용하자. 예외는 없다. 코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용하다. try-finally로 작성하면 실용적이지 못할 만큼 코드가 지저분해지는 경우라도, try-with-resources로 정확하고 쉽게 자원을 회수할 수 있다
> 


- 전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 `try-finally`가 쓰였다

```Java
static String firstLineOfFile(String path) throws IOException {
	BufferedReader br = new BufferedReader(new FileReader(path));
	try {
		return br.readLine();
	} finally {
		br.close();
	}
}
```

- 자원을 하나 더 사용하면 아래와 같이 코드를 작성한다

```Java
static void copy(String src, String dst) throws IOException {
	InputStream in = new FileInputStream(src);
	try {
		OutputStream out = new FileOutputStream(dst);
		try {
			byte[] buf = new byte[BUFFER_SIZE];
			int n;
			while((n = in.read(buf)) >= 0) {
				out.write(buf, 0, n);
			}
		} finally {
			out.close();
		}
	} finally {
		in.close();
	}
}
```

- 기기의 물리적인 문제가 발생하면 `firstLineOfFile`에서 `readLine`에서 예외가 발생하고 동일한 이유로 `close`에서 예외가 발생한다
	- 이 경우 두번째 예외가 첫 번째 예외를 삼켜 스택 트레이스에서 확인할 수 없다
	- 스택 추적 내역에 첫 번째 예외에 관한 정보가 남지 않아 디버깅이 어려워진다

### try-with-resources
- `try-with-resources` 를 사용하기 위해서는 해당 자원이 Autoclosable을 구현해야 한다
- 자바 라이브러리와 수 많은 서드파티 라이브러리들의 수 많은 클래스는 이미 Autoclosable을 구현하거나 확장 해뒀다

```Java
static String firstLineOfFile(String path) throws IOException {
	try (BufferedReader br = new BufferedReader(new FileReader(path))){
		return br.readLine();
	};
}
```

```Java
static void copy(String src, String dst) throws IOException {
	try(InputStream in = new FileInputStream(src);
		OutputStream out = new FileOutputStream(dst)) {
		byte[] buf = new byte[BUFFER_SIZE];
		int n;
		while((n = in.read(buf)) >= 0) {
			out.write(buf, 0, n);
		}
	}
}
```
- try-finally를 사용할 때보다 짧고 읽기 수월할 뿐아니라 문제를 진단하기 훨씬좋다
- readLine과 close 호출 양 쪽에 예외가 발생하면 close에서 발생한 예외는 숨겨지고 readLine에서 발생한 예외가 기록된다
- 숨겨진 예외들도 버려지지 않고 자바 7에서 추가된 `Throwable`의 `getSuppressed`메서드를 사용하면 프로그램 코드에서 가져올 수도 있다

### try-with-resources에서 예외처리
- try-with-resources에서도 catch를 사용할 수 있다

```Java
static String firstLineOfFile(String path, String defaultVal) throws IOException {
	try (BufferedReader br = new BufferedReader(new FileReader(path))){
		return br.readLine();
	} catch (IOException e) {
		return defaultVal;
	}
}
```