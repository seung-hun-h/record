## 가변인수는 신중히 사용하라
---
- 가변인수 메서드는 명시한 타입의 인수를 0개 이상 받을 수 있다
- 가변인수 메서드를 호출하면, 인수의 개수와 길이가 같은 배열을 만들고 인수들을 이 배열에 저장하여 가변인수 메서드에 건네준다

```Java
static int min(int... args) {
	if (args.length == 0) {
		throw new IllegalArgumentException("인수가 1개 이상 필요합니다");
	}
	int min = args[0];
	for (int i = 1; i < args.length; i++) {
		if (args[i] < min) {
			min = args[i];
		}
	}

	return min;
}
```
- 가변인수를 사용한 이 방식에는 몇가지 문제점이 존재한다
	- 인수를 0개를 넣어 호출하면 컴파일타임이 아닌 런타임에 가서야 예외가 발생한다
	- 코드가 지저분하다
		- args 유효성 검사를 명시적으로 해야 한다
		- min의 초기값을 `Integer.MAX_VALUE`로 설정하지 않고는 for-each문을 사용할 수도 없다

- 첫 번째로는 평범한 매개변수로 받고, 가변인수를 두 번째로 받으면 위 문제를 해결할 수 있다

```Java
static int min(int firstArg, int... remainingArgs) {
	int min = firstArgs;
	for (int arg : remainingArgs) {
		if (arg < min) {
			min = arg;
		}
	}
	return min;
}
```

- 위 예제에서 볼 수 있듯이 가변인수는 인수의 개수가 정해지지 않았을 때 매우 유용하다
- 하지만 성능에 민감한 상황이라면 가변인수가 걸림돌이 될 수 있다
	- 메서드가 호출될 때마다 배열을 새로 생성하고 초기화하기 때문이다

- 만약 인자의 개수가 5개 이상인 경우가 5% 미만이라면, 아래와 같이 다중정의하여 비용을 줄일 수 있다
```Java
public void foo() {}
public void foo(int a1) {}
public void foo(int a1, int a2) {}
public void foo(int a1, int a2, int a3) {}
public void foo(int a1, int a2, int a3, int a4) {}
public void foo(int a1, int a2, int a3, int a4, int args) {}
```
- 메서드 호출 중 5% 미만이 배열을 새로 생성하게된다
