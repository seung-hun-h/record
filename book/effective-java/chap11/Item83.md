## 지연 초기화는 신중히 사용하라
---
- 지연 초기화는 필드의 초기화 시점을 그 값이 처음 필요할 때까지 늦추는 기법이다
- 필요할 때까지 지연 초기화 기법은 사용하지 않는 것이 좋다
- 지연 초기화가 필요한 경우가 있는데, 이는 지연 초기화 적용 전후의 성능을 측정 해보아야 한다
- 대부분의 상황에서 일반적인 초기화가 지연 초기화보다 낫다

### 예제
#### 일반적인 초기화
```Java
private final FieldType field = computeFieldValue();
```

#### synchronized 생성자
- 지연 초기화가 초기화 순환성을 깨뜨릴거 같으면 synchronized 생성자를 사용하면된다
```Java
private final FieldType field;

private synchronized FieldType getField() {
	if (field == null) {
		field = computeFieldValue();
	}
	return field;
}
```


#### 지연 초기화 홀더 클래스
- 성능 때문에 정적 필드를 지연 초기화 해야 한다면 지연 초기화 홀더 클래스 관용구를 사용하자
	- 클래스가 처음 쓰일 때 비로소 초기화 된다는 특성을 이용한 관용구다
```Java
private static class FieldHolder {
	static final FieldType field = computeFieldValue();
}

private static FieldType getField() { return FieldHolder.field;}
```

#### 이중검사
- 성능 때문에 인스턴스 필드를 지연 초기화해야 한다면 이중 검사 관용구를 사용하라
- 초기화된 필드에 접근할 때의 동기화 비용을 줄여준다
- 한 번은 동기화 없이 검사하고, 두 번째는 동기화하여 검사한다

```Java
private volatile FieldType field;

private FieldType getField() {
	FieldType result = field;
	if (result != null) {
		return result;
	}

	synchronized(this) {
		if (field == null) {
			field = computeFieldValue();
		}
		return field;
	}
}
```
- `result`를 사용한 이유
	- 필드가 이미 초기화된 상황에서는 그 필드를 딱 한 번만 읽도록 보장하는 역할
	- 반드시 필요하지는 않지만 성능을 높여주고, 저수준 동시성 프로그래밍에 표준적으로 사용되는 우아한 방법이다
