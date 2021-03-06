## 목차
- [객체](#객체)
  - [객체 생성](#객체-생성)
  - [리터럴과 프로퍼티](#리터럴과-프로퍼티)
    - [trailing comma](#trailing-comma)
  - [대괄호 표기법](#대괄호-표기법)
    - [계산된 프로퍼티](#계산된-프로퍼티)
  - [단축 프로퍼티](#단축-프로퍼티)
  - [프로퍼티 이름의 제약 사항](#프로퍼티-이름의-제약-사항)
  - ['in' 연산자로 프로퍼티 존재 여부 확인하기](#in-연산자로-프로퍼티-존재-여부-확인하기)
  - ['for ...in' 반복문](#for-in-반복문)
    - [객체 정렬 방식](#객체-정렬-방식)
- [참조에 의한 객체 복사](#참조에-의한-객체-복사)
  - [참조에 의한 비교](#참조에-의한-비교)
  - [객체 복사, 병합과 Object.assign](#객체-복사-병합과-objectassign)
- [가비지 컬렉션](#가비지-컬렉션)
  - [가비지 컬렉션 기준](#가비지-컬렉션-기준)
  - [내부 알고리즘](#내부-알고리즘)
    - [최적화 기법](#최적화-기법)
- [메서드와 this](#메서드와-this)
  - [메서드](#메서드)
    - [메서드 단축 구문](#메서드-단축-구문)
  - [메서드와 this](#메서드와-this-1)
  - [자유로운 this](#자유로운-this)
    - [자유로운 this가 만든 결과](#자유로운-this가-만든-결과)
  - [this가 없는 화살표 함수](#this가-없는-화살표-함수)
- [new 연산자와 생성자 함수](#new-연산자와-생성자-함수)
  - [생성자 함수](#생성자-함수)
  - [생성자와 return문](#생성자와-return문)
  - [생성자 내 메서드](#생성자-내-메서드)
- [옵셔널 체이닝 '?.'](#옵셔널-체이닝-)
  - [옵셔널 체이닝이 필요한 이유](#옵셔널-체이닝이-필요한-이유)
  - [옵셔널 체이닝 등장](#옵셔널-체이닝-등장)
  - [단락 평가](#단락-평가)
- [심볼형](#심볼형)
  - [심볼](#심볼)
    - [심볼의 형 변환](#심볼의-형-변환)
  - ['숨김' 프로퍼티](#숨김-프로퍼티)
    - [Symbols in a literal](#symbols-in-a-literal)
    - [심볼은 for ...in에서 배제된다](#심볼은-for-in에서-배제된다)
  - [전역 심볼](#전역-심볼)
    - [Symbol.keyFor](#symbolkeyfor)
  - [시스템 심볼](#시스템-심볼)


# 객체
자바스크립트의 데이터틑 원시형(Primitive type) 데이터와 참조형 (Reference type) 데이터로 나뉜다. 원시형 데이터는 오직 하나의 데이터만 담을 수 있다. 원시형 데이터를 제외한 모든 데이터는 참조형 데이터인데, 객체는 자바스크립트의 대표적인 참조형 데이터이다.

객체는 원시형과 달리 키로 구분된 데이터 집합이나 복잡한 개체를 저장할 수 있다.

## 객체 생성
객체를 생성하는 방법에는 '객체 생성자' 문법과 '객체 리터럴' 문법이 있다. 객체를 생성할 때 일반적으로 객체 리터럴 문법을 사용한다

```javascript
let user = new Object(); // 객체 생성자 문법
let user = {} // 객체 리터럴 문법
```

객체는 '키(key):값(value)' 쌍으로 구성된 프로퍼티로 구성된다. 키에는 문자형, 값에는 모든 자료형이 허용된다. 프로퍼티 키는 프로퍼티 이름이라고 부른다.

## 리터럴과 프로퍼티
중괄호 `{...}`안에는 '키:값' 쌍으로 구성된 프로퍼티가 들어간다. 


```javascript
let user = {
  name: 'Jhon',
  age: 30
}
```

프로퍼티 값을 읽을 때는 점 표기법을 사용하고, 프로퍼티를 삭제하기 위해서는 `delete` 키워드를 사용한다

```javascript
let user = {
  name: 'Jhon',
  age: 30
};

console.log(user.name);
delete user.name;
```

### trailing comma
객체의 프로퍼티를 나열 할 때는 쉼표(`,`)를 사용한다. 

```javascript
let user = {
  name: "John",
  age: 30,
}
```
위 처럼 프로퍼티의 끝을 쉼표로 끝낼 수 있는데 이러한 표기법을 'trailing(길게 늘어지는)', 'hanging(매달리는)' 쉼표라고 부른다. 이렇게 끝에 쉼표를 붙이면 모든 프로퍼티가 유사한 형태를 보이기 때문에 프로퍼티를 추가, 삭제, 이동하는 게 쉬워진다.

## 대괄호 표기법
여러 단어를 조합해 프로퍼티 키를 만든 경우엔, 점 표기법을 사용해 프로퍼티 값을 읽을 수 없다

```javascript
user.likes birds = true;
```

점표기법은 '유효한 변수 식별자'인 경우에만 사용할 수 있다. 유효한 변수 식별자란 다음과 같다
- 공백이 없어야 한다
- 숫자로 시작하지 않아야 한다
- `$`, `_`를 제외한 특수 문자가 없어야 한다

유효한 변수 식별자가 아닌 경우에는 대괄호 표기법을 사용할 수 있다
```javascript
user['likes birds'] = true;
```

대괄호 표기법을 사용하면 프로퍼티 키가 런타임에 평가되어 코드를 유연하게 작성할 수 있다

```javascript
let user = {
  name: "John",
  age: 30
};

let key = prompt("사용자의 어떤 정보를 얻고 싶으신가요?", "name");

// 변수로 접근
alert( user[key] ); // John (프롬프트 창에 "name"을 입력한 경우)
alert( user.key ) // undefined
```

### 계산된 프로퍼티
객체를 만들 때 객체 리터럴 안의 프로퍼티 키가 대괄호로 둘러싸여 있는 경우, 이를 계산된 프로퍼티(computed property)라고 부른다.

```javascript
let fruit = prompt("어떤 과일을 구매하시겠습니까?", "apple");

let bag = {
  [fruit]: 5, // 변수 fruit에서 프로퍼티 이름을 동적으로 받아 옵니다.
};

alert( bag.apple ); // fruit에 "apple"이 할당되었다면, 5가 출력됩니다.

/**
 * bag[fruit] = 5 와 같이 작성도 가능하다
*/
```

위 예시에서 `[fruit]`는 프로퍼티 이름을 변수 `fruit`에서 가져오겠다는 것을 의미한다.

대괄호 표기법은 프로퍼티 이름과 값의 제약을 없애주기 때문에 점 표기법 보다 훨씬 강력하지만 작성하기 번거롭다는 단점이있다. 따라서 프로퍼티 이름이 확정된 상황이고, 단순한 이름이라면 처음엔 점 표기법을 사용하다가 복잡한 상황이 발생했을 때 대괄호 표기법으로 바꾸는 경우가 많다

## 단축 프로퍼티
프로퍼티 이름과 변수의 이름이 동일한 경우 프로퍼티 이름만 작성할 수 있다.

```javascript
function makeUser(name, age) {
  return {
    name, // name: name 과 같음
    age,  // age: age 와 같음
    // ...
  };
}
```

## 프로퍼티 이름의 제약 사항
변수 이름에는 `for`, `let`, `return` 같은 예약어를 사용하면 안되지만, 객체 프로퍼티에는 이런 제약이 없다

```javascript
// 예약어를 키로 사용해도 괜찮습니다.
let obj = {
  for: 1,
  let: 2,
  return: 3
};

alert( obj.for + obj.let + obj.return );  // 6
```
이처럼 프로퍼티 이름에는 특별한 제약이 없다. 문자형, 심볼형 값도 프로퍼티 키가 될 수 있고, 문자형이나 심볼형에 속하지 않는 값은 문자형으로 자동 변환된다.

하지만 조금 특별한 프로퍼티인 `___proto___`가 있다. `___proto___`에 값을 할당해도 무시된다

```javascript
let obj = {};
obj.__proto__ = 5; // 숫자를 할당합니다.
alert(obj.__proto__); // [object Object] - 숫자를 할당했지만 값은 객체가 되었습니다. 의도한대로 동작하지 않네요.
```

## 'in' 연산자로 프로퍼티 존재 여부 확인하기
자바스크립트 객체의 중요한 특징 중 하나는 다른 언어와는 달리, 존재하지 않는 프로퍼티에 접근하려 해도 에러가 발생하지 않고 `undefined`를 반환하는 것이다.

```javascript
let user = {};
console.log(user.noProperty === undefined);
```
`undefined`를 비교하는 것 외에 `in` 연산자를 사용할 수 있다. `in` 연산자를 사용하면 프로퍼티 존재 여부를 확인할 수 있다.


```javascript
let user = { name: "John", age: 30 };

alert( "age" in user ); // user.age가 존재하므로 true가 출력됩니다.
alert( "blabla" in user ); // user.blabla는 존재하지 않기 때문에 false가 출력됩니다.
```

`undefined`를 비교하는 방법은 특별한 경우에 의도한대로 동작하지 않을 수 있다. 따라서 프로퍼티의 존재 여부를 확실히 확인하기 위해서는 `in`을 사용한다


```javascript
let obj = {
  test: undefined
};

alert( obj.test ); // 값이 `undefined`이므로, 얼럿 창엔 undefined가 출력됩니다. 그런데 프로퍼티 test는 존재합니다.
alert( "test" in obj ); // `in`을 사용하면 프로퍼티 유무를 제대로 확인할 수 있습니다(true가 출력됨).
```

## 'for ...in' 반복문
`for ...in` 반복문을 사용하면 객체의 모든 키를 순회할 수 있다

```javascript
for (key in object) {
  // 각 프로퍼티 키(key)를 이용하여 본문(body)을 실행합니다.
}
```

### 객체 정렬 방식
객체는 특별한 방식으로 정렬된다. 정수 프로퍼티는 자동으로 정렬되고, 그 외 프로퍼티는 객체에 추가한 순서 그대로 정렬된다.
- 정수 프로퍼티
  - 변형 없이 정수에서 변환되는 문자열
  - +42, 1.2는 해당 안됨

# 참조에 의한 객체 복사
객체와 원시 타입의 근본적인 차이 중 하나는 객체는 '참조에 의해(by reference)' 저장되고 복사된다는 것이다. 반면에 원시 값(문자열, 숫자, 불린 값)은 값 그대로 저장, 할당, 복사 된다.

참조에 의해 저장되고 복사된다는 것은, 변수에 객체가 저장되어있는 '메모리 주소'인 객체에 대한 '참조 값'이 저장된다는 것이다. 따라서 객체가 할당된 변수를 복사할 때는 객체의 참조 값이 복사되고 객체는 복사되지 않는다.

## 참조에 의한 비교
객체 비교 시 동등 연산자 `==`와 일치 연산자 `===`는 동일하게 동작한다.

```javascript
let a = {};
let b = a; // 참조에 의한 복사

alert( a == b ); // true, 두 변수는 같은 객체를 참조합니다.
alert( a === b ); // true

b = {};
alert( a == b ); // false
```

obj1 > obj2 같은 대소 비교나 obj == 5 같은 원시값과의 비교에서는 객체가 원시형으로 변환된다.

## 객체 복사, 병합과 Object.assign
자바스크립트는 객체 복제 내장 메서드를 지원하지 않아 조금 어렵다. 사실 객체를 복제해야 할 일은 거의 없다. 하지만 복제가 필요한 상황이라면 새로운 객체를 만든 다음 기존 객체의 프로퍼티들을 순회해 원시 수준까지 복사하면 된다.

```javascript
let user = {
  name: "John",
  age: 30
};

let clone = {}; // 새로운 빈 객체

// 빈 객체에 user 프로퍼티 전부를 복사해 넣습니다.
for (let key in user) {
  clone[key] = user[key];
}

// 이제 clone은 완전히 독립적인 복제본이 되었습니다.
clone.name = "Pete"; // clone의 데이터를 변경합니다.

alert( user.name ); // 기존 객체에는 여전히 John이 있습니다.
```

`Object.assign`을 사용할 수도 있다.

```javascript
Object.assign(dest, [src1, src2, src3...])
```

- `dest`는 목표로 하는 객체이다
- 인수 `src1, ,src2 ... ,srcN`은 복사하고자 하는 객체이다.
- `src1, ,src2 ... ,srcN`의 프로퍼티를 `dest`에 복사한다.
- 만약 목표 객체에 동일한 이름을 가진 프로퍼티가 있는 경우 기존 값이 덮어씌워진다.
- 마지막으로 `dest`를 반환한다.

# 가비지 컬렉션
자바스크립트는 더이상 사용하지 않는 메모리는 가비지 컬렉션에 의해서 관리된다

## 가비지 컬렉션 기준
자바스크립트는 '도달 가능성(reachability)'이라는 개념을 사용해서 메모리를 관리한다. 도달 가능한 값은 어떻게든 접근이 가능한 값이다.
1. 태생부터 도달 가능한 값들
   1. 현재 함수의 지역 변수와 매개 변수
   2. 중첩 함수의체인에 있는 함수에서 사용되는 변수와 매개 변수
   3. 전역 변수
   4. 등등
2. 루트가 참조하는 값이나 체이닝으로 루트에서 참조할 수 있는 값
\
전역 변수에 객체가 저장되어 있고, 이 객체의 프로퍼티가 다른 객체를 참조하고 있다면 프로퍼티가 참조하는 객체는 도달 가능한 객체이다.


## 내부 알고리즘
가비지 컬렉션은 'mark-and-sweep'이라 불리는 기본 알고리즘에 의해 동작한다
- 가비지 컬렉터는 루트 정보를 수집하고 이를 기억(mark)한다
- 루트가 참조하고 있는 모든 객체를 방문하고 이것들을 mark한다
- mark한 모든 객체를 방문하고 그 객체들이 참조하는 객체들도 mark한다
- 루트에서 도달 가능한 모든 객체를 방문할 떄까지 위 과정을 반복한다
- mark되지 않은 모든 객체를 메모리에서 삭제한다

### 최적화 기법
- generational collection
  - 객체를 새로운 객체와 오래된 객체로 나눈다. 객체 상당수는 생성 이후 제 역할을 빠르게 수행해 금방 쓸모 없어지는데, 이런 객체를 새로운 객체로 분류한다. 가비지 컬렉터는 이런 객체를 공격적으로 메모리에서 삭제하고, 일정 시간 이상 동안 살아남은 객체는 오래된 객체로 분류하고 가비지 컬렉터가 감시한다
- incremental collection
  - 방문 해야 할 객체가 많다면 mark에 상당한 시간이 소모된다. 이런 현상을 개선하기 위해 가비지 컬렉션을 여러 부분으로 나눈 다음, 각 부분을 별도로 수행한다. 작업을 분리하고 변경 사항을 추적하는 추가 작업이 필요하지만 긴 지연을 짧은 지연 여러개로 분산 시킬 수 있다는 장점이 있다.
- idle-time collection
  - 실행에 주는 영향을 최소화하기 위해서 CPU가 유휴 상태일 때에만 가비지 컬렉션을 수행한다


# 메서드와 this
## 메서드
객체는 사용자나 주문처럼 실제 존재한 개체를 표현하고자 할 때 사용한다. 객체는 개체의 속성을 표현하기 위해서 상태를 가지게 되는데, 예를 들어 사용자 객체는 이름이나 나이 같은 상태를 가진다.
```javascript
let user {
  name: 'john',
  age: 30
};
```

객체는 상태 뿐 아니라 행동할 수 있는 능력도 가지고있다. 이 행동을 메서드라고 한다.

```javascript
let user {
  name: 'john',
  age: 30
};

user.sayHi = function() {
  alert('Hi!');
}

user.sayHi();
```

`user` 객체의 `sayHi`가 메서드이다.

### 메서드 단축 구문
객체 리터럴 안에 메서드를 선언할 때는 `function`을 생략한 단축 구문을 사용할 수 있다.
```javascript
user = {
  sayHi: function() {
    alert("Hello");
  }
};

// 단축 구문을 사용하니 더 깔끔해 보이네요.
user = {
  sayHi() { // "sayHi: function()"과 동일합니다.
    alert("Hello");
  }
};
```

## 메서드와 this
대부분의 메서드는 객체 내부 프로퍼티 값을 활용하여 동작한다. 메서드 내부에서 `this` 키워드를 사용하면 객체에 접근할 수 있다

```javascript
let user = {
  name: 'john',
  age: 30,
  sayHi() {
    // this는 현재 객체를 나타낸다
    console.log(this.name);
  }
}

user.sayHi();
```
`user.sayHi()`가 실행되는 동안 `this`는 `user`를 나타낸다. this를 사용하지 않고 변수 이름으로 접근할 수 있지만, 변수에 할당된 값이 변경되는 경우 치명적인 오류가 발생할 수 있다.

```javascript
let user = {
  name: 'john',
  age: 30,
  sayHi() {
    // this는 현재 객체를 나타낸다
    console.log(this.name);
  }
}

let another = user;
user = null; // user를 null로 덮어쓴다.

another.sayHi();
user.sayHi();
```

## 자유로운 this
자바스크립트의 `this`는 다른 프로그래밍 언어의 `this`와 동작 방식이 다르다. 자바스크립트에선 모든 함수에 `this`를 사용할 수 있다.
```javascript
function sayHi() {
  alert(this.name); // 문법 에러가 발생하지 않는다
}
```

`this`의 값은 런타임에 결정되기 떄문에 컨텍스트에 따라 달라지게 된다. 동일한 함수라도 다른 객체에서 호출했다면 `this`가 참조하는 값이 달라진다.
```javascript
let user = { name: "John" };
let admin = { name: "Admin" };

function sayHi() {
  alert( this.name );
}

// 별개의 객체에서 동일한 함수를 사용함
user.f = sayHi;
admin.f = sayHi;

// 'this'는 '점(.) 앞의' 객체를 참조하기 때문에
// this 값이 달라짐
user.f(); // John  (this == user)
admin.f(); // Admin  (this == admin)

admin['f'](); // Admin (점과 대괄호는 동일하게 동작함)
```
### 자유로운 this가 만든 결과
다른 언어를 사용하는 개발자는 자바스크립트의 this가 항상 메서드가 정의된 객체를 참조할 것이라고 착각한다. 이런 개념을 'bound this'라 한다. 하지만 자바스크립트의 `this`는 런타임에 결정된다. 메서드가 어디서 정의되었는지에 상관 없이 '점 앞의' 객체가 무엇인가에 따라 자유롭게 결정된다.

`this`가 런타임에 결정되면 함수를 하나 만들어 여러 객체에서 재사용할 수 있다는 장점이 있지만, 이런 유연함이 실수로 이어질 수 있다는 단점이 존재한다.

## this가 없는 화살표 함수
화살표 함수는 일반 함수와 달리 '고유한' `this`를 가지지 않는다. 화살표 함수에서 `this`를 참조하면, 화살표 함수가 아닌 '평범한'외부 함수에서 `this`를 가져온다.

아래 예시에서 함수 `arrow()`의 `this`는 외부 함수 `user.sayHi()`의 `this`가 된다

```javscript
let user = {
  firstName: "보라",
  sayHi() {
    let arrow = () => alert(this.firstName);
    arrow();
  }
};

user.sayHi(); // 보라
```
별개의 `this`가 만들어지는 것을 원하지 않고, 외부 컨텍스트에 있는 `this`를 이용하고 싶은 경우 화살표 함수가 유용하다.

# new 연산자와 생성자 함수
객체 리터럴(`{ ... }`)를 사용하면 객체를 쉽게 만들 수 있다. 하지만 개발을 하면서 유사 객체를 여러개 만들어야 할 때가 있는데, 이떄 `new` 연산자와 생성자 함수를 사용하면 유사한 객체를 여러개 만들 수 있다.

## 생성자 함수
생성자 함수는 일반 함수와 기술적인 차이는 없지만, 다음과 같은 관례를 따른다
1. 함수 이름의 첫 글자는 대문자로 시작된다
2. 반드시 `new` 연산자와 함께 사용한다.

```javascript
function User(name) {
    this.name = name;
    this.isAdmin = false;
}

let user = new User("철수");

console.log(user.name); // 철수
console.log(user.isAdmin); // false
```

`new User(...)`를 사용해 함수를 실행하면 아래와 같은 알고리즘이 동작한다.
1. 빈 객체를 만들어 `this`에 할당한다.
2. 함수 본문을 실행한다. `this`에 새로운 프로퍼티를 추가해 `this`를 수정한다
3. `this`를 반환한다

```javascript
function User(name) {
    // this = {};
    this.name = name;
    this.isAdmin = false;
    // return this;
}

let user = new User("철수");

console.log(user.name); // 철수
console.log(user.isAdmin); // false
```

이제 손쉽게 유사한 사용자 객체를 여러개 만들 수 있다. 객체 리터럴 문법으로 일일이 객체를 만드는 것보다 간단하고 읽기 쉽게 만들 수 있게 되었다. 생성자의 의의는 재사용할 수 있는 객체 생성 코드를 구현하는 것이다.

## 생성자와 return문
생성자 함수에는 보통 `return`문이 없다. 반환해야 할 것들은 모두 `this`에 저장되고, `this`는 자동으로 반환되기 떄문에 반환문을 명시적으로 써 줄 필요가 없다. 그런데 만약 `return`문이 있다면 다음 규칙을 따른다
- 객체를 return 한다면 this 대신 객체가 반환된다
- 원시형을 return 한다면 return 문이 무시된다

## 생성자 내 메서드
생성자 함수를 사용하면 매개변수를 이용해 객체 내부를 자유롭게 구성할 수 있다.

```javascript
function User(name) {
  this.name = name;

  this.sayHi = function() {
    alert( "제 이름은 " + this.name + "입니다." );
  };
}

let bora = new User("이보라");

bora.sayHi(); // 제 이름은 이보라입니다.

/*
bora = {
   name: "이보라",
   sayHi: function() { ... }
}
*/
```

# 옵셔널 체이닝 '?.'
옵셔널 체이닝 `?.`을 사용하면 프로퍼티가 없는 중첩 객체를 에러 없이 안전하게 접근할 수 있다.

## 옵셔널 체이닝이 필요한 이유
사용자가 여러 명 있는데 그 중 몇 명은 주소 정보를 가지고 있지 않다고 가정하면, `user.address.street`을 사용해 주소 정보에 접근하면 에러가 발생할 수 있다.

```javascript
let user = {}; // 주소 정보가 없는 사용자

alert(user.address.street); // TypeError: Cannot read property 'street' of undefined
```

그리고 브라우저에 동작하는 코드를 개발할 때, `querySelector()`를 사용해 페이지에 존재하지 않는 요소에 접근해 요소의 정보를 가져오려면 에러가 발생한다.

```javascript
// querySelector(...) 호출 결과가 null인 경우 에러 발생
let html = document.querySelector('.my-element').innerHTML;
```

옵셔널 체이닝이 추가되기 전에는 이러한 문제를 해결하기 위해서 `&&` 연산자를 사용했다.

```javascript
let user = {}; // 주소 정보가 없는 사용자

alert( user && user.address && user.address.street ); // undefined, 에러가 발생하지 않습니다.
```

## 옵셔널 체이닝 등장
`?.`는 '앞'의 평가 대상이 `undefined`나 `null`이면 평가를 멈추고 `undefined`를 반환한다. 

```javascript
let user = {}; // 주소 정보가 없는 사용자

alert( user?.address?.street ); // undefined, 에러가 발생하지 않습니다.
```

옵셔널 체이닝은 존재하지 않아도 괜찮은 대상에만 사용해야한다. 만약 실수로 변수에 값을 할당하지 않는 경우에는 바로 알아낼 수 있도록 해야한다. 

## 단락 평가
`?.`는 왼쪽 평가 대상에 값이 없으면 즉시 평가를 멈춘다. 이런 평가 방법을 단락 평가라고 부른다. 따라서 함수 호출을 비롯한 `?.` 오른쪽에 있는 부가 동작은 `?.`의 평가가 멈췄을 때 더는 일어나지 않는다.


# 심볼형
자바스크립트는 객체 프로퍼티 키로 오직 문자형과 심볼형만을 허용한다. 

## 심볼
'심볼(Symbol)'은 유일한 식별자를 만들고 싶을 때 사용한다. `Symbol()`을 사용하면 심볼 값을 만들 수 있다.

```javascript
let id = Symbol();
```

심볼을 만들 때 심볼 이름이라는 설명을 붙일 수도 있다. 심볼 이름은 디버깅 시 매우 유용하다.

```javascript
let id = Symbol("id");
```

심볼은 유일성이 보장되는 자료형이기 때문에 설명이 동일한 심볼을 여러 개 만들어도 각 심볼 값은 다르다. 심볼에 붙이는 설명은 어떤 것에도 영향을 주지 않는 이름표 역할만 한다.

### 심볼의 형 변환
심볼형 값은 다른 자료형으로 암시적 형 변환되지 않는다. 문자열과 심볼은 근본이 다르기 때문에 우연히라도 서로의 타입으로 변환돼서는 안 된다. 자바스크립트에서는 언어 차원의 보호 장치를 바련해 심볼형이 다른 형으로 변환되지 않게 막아준다. 심볼을 반드시 출력 해줘야하는 상황이면 `.toString()` 메서드를 호출 해주면 된다. 그리고 `symbol.description` 프로퍼티를 이용하면 설명만 보여주는 것도 가능하다.

## '숨김' 프로퍼티
심볼을 이용하면 '숨김(Hidden)' 프로퍼티를 만들 수 있다. 숨김 프로퍼티는 외부 코드에서 접근이 불가능하고 값도 덮어 쓸 수 없는 프로퍼티이다. 

```javascript
let user = { // 서드파티 코드에서 가져온 객체
  name: "John"
};

let id = Symbol("id");

user[id] = 1;

alert( user[id] ); // 심볼을 키로 사용해 데이터에 접근할 수 있습니다.
```
`user`가 서드파티 코드에서 가져온 객체라고 가정할 때 문자열 'id'를 대신 'Symbol("id")'를 사용한 이유는 서드파티 코드가 접근할 수 없게 하기 위함이다. 

### Symbols in a literal
객체 리터럴 `{...}`을 사용해 객체를 만든 경우, 대괄호를 사용해 심볼형 키를 만들어야 한다.

```javascript
let id = Symbol("id");

let user = {
  name: "John",
  [id]: 123 // "id": 123은 안됨
};
```

### 심볼은 for ...in에서 배제된다
키가 심볼인 프로퍼티는 `for...in` 반복문에서 배제된다.

`Object.keys(user)`에서도 키가 심볼인 프로퍼티는 배제된다. 그런데 `Object.assign`은 키가 심볼인 프로퍼티를 배제하지 않고 객체 내 모든 프로퍼티를 복사한다.

## 전역 심볼
전역 심볼 레지스트리안에 심볼을 만들고 해당 심볼에 접근하면, 이름이 같은 경우 항상 동일한 심볼을 반환해준다. 레지스트리 안에 있는 심볼을 읽거나, 새로운 심볼을 생성하려면 `Symbol.for(key)`를 사용하면된다.

이 메서드를 호출하면 이름이 `key`인 심볼을 반환한다. 조건에 맞는 심볼이 레지스트리 안에 없으면 새로운 심볼을 만들고 레지스트리 안에 저장한다.
```javascript
// 전역 레지스트리에서 심볼을 읽습니다.
let id = Symbol.for("id"); // 심볼이 존재하지 않으면 새로운 심볼을 만듭니다.

// 동일한 이름을 이용해 심볼을 다시 읽습니다(좀 더 멀리 떨어진 코드에서도 가능합니다).
let idAgain = Symbol.for("id");

// 두 심볼은 같습니다.
alert( id === idAgain ); // true
```

### Symbol.keyFor
`Symbol.ketFor(sym)`은 심볼의 이름을 얻을 수 있다.

```javascript
// 이름을 이용해 심볼을 찾음
let sym = Symbol.for("name");
let sym2 = Symbol.for("id");

// 심볼을 이용해 이름을 얻음
alert( Symbol.keyFor(sym) ); // name
alert( Symbol.keyFor(sym2) ); // id
```

## 시스템 심볼
'시스템 심볼'은 자바스크립트 내부에서 사용되는 심볼이다. 시스템 심볼을 활용하면 객체를 미세 조정할 수 있다.
- Symbol.hasInstance
- Symbol.isConcatSpreadable
- Symbol.iterator
- Symbol.toPrimitive

