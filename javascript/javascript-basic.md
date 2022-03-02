## 목차
- [자바스크립트 기본](#자바스크립트-기본)
  - [자바스크립트의 탄생](#자바스크립트의-탄생)
    - [ECMAScript](#ecmascript)
    - [자바 스크립트의 특징](#자바-스크립트의-특징)
  - [변수](#변수)
    - [변수의 중복선언](#변수의-중복선언)
    - [동적 타이핑](#동적-타이핑)
    - [변수 호이스팅](#변수-호이스팅)
    - [var의 문제점](#var의-문제점)
  - [연산자](#연산자)
    - [동등 / 일치 연산자](#동등--일치-연산자)
  - [타입 변환](#타입-변환)
    - [암묵적 타입 변환](#암묵적-타입-변환)
    - [명시적 타입 변환](#명시적-타입-변환)
    - [falsy](#falsy)
    - [trusy](#trusy)
  - [객체(Object)](#객체object)
    - [객체 생성 방법](#객체-생성-방법)
    - [for ...in](#for-in)
  - [함수](#함수)
    - [함수 정의](#함수-정의)
    - [함수 호이스팅](#함수-호이스팅)
  - [함수 객체의 프로퍼티](#함수-객체의-프로퍼티)
    - [arguments](#arguments)
    - [caller](#caller)
    - [length](#length)
    - [name](#name)
    - [\_\_proto__ 접근자 프로퍼티](#__proto__-접근자-프로퍼티)
    - [prototype](#prototype)
  - [함수의 다양한 형태](#함수의-다양한-형태)
    - [즉시 실행 함수](#즉시-실행-함수)
    - [내부 함수](#내부-함수)
    - [콜백 함수](#콜백-함수)
  - [프로토타입](#프로토타입)
    - [[[Prototype]] vs prototype 프로퍼티](#prototype-vs-prototype-프로퍼티)
# 자바스크립트 기본
## 자바스크립트의 탄생
자바스크립트는 HTML, CSS로 이루어진 정적인 웹 페이지를 동적으로 표현하기 위해 탄생했다. 초기 JS는 HTML, CSS 렌더링의 단순 보조 수단이었다. 하지만 1999년 JS를 이용한 브라우저-서버간 비동기 통신 기능인 Ajax가 나타났다. Ajax를 통해서 변경이 있는 부분만 서버로 부터 재전송 받아 필요한 부분만 한정적으로 렌더링하여 퍼포먼스를 크게 향상시켰다.

### ECMAScript
마이크로소프트에서는 JScript를 개발하여 IE에 독자적으로 탑재했다. 하지만 JScript가 IE 이외 브라우저에서는 제대로 동작하지 않는 **크로스 브라우징** 이슈가 발생했고, JS에 대한 표준 명세에 대한 요구가 발생했고 ECMAScript가 탄생했다.

ECMAScript는 자바스크립트의 표준 명세인 ECMA-262를 말하며 프로그래밍 언어의 타입, 값, 객체외 프로퍼티, 빌트인 객체 등 핵심 문법을 규정한다. 자바스크립트는 ECMAScript와 브라우저가 별도로 지원하는 클라이언트 사이드 Web API(DOM, BOM, Canvas, XMLHttpRequest, Fetch ...)를 아우르는 개념이다.

### 자바 스크립트의 특징
- 웹 브라우저에서 동작하는 유일한 프로그래밍 언어
- 개발자가 별도의 컴파일 작업을 수행하지 않는 인터프리터 언어
  - 대부분 모던 자바스크립트 엔진은 인터프리터와 컴파일러의 장점을 결합하여 느린 인터프리터 언어의 한계점을 해결했다
- 명령형, 함수형, 프로토타입 기반 객체지향 프로그래밍을 지원하는 멀티 패러다임 프로그래밍 언어

## 변수
변수는 var, let, const 키워드를 사용하여 선언하고 할당 연산자를 사용해 값을 할당한다. 그리고 식별자은 변수명을 사용해 변수에 저장된 값을 참조한다.

### 변수의 중복선언
- var 키워드로 선언한 변수는 중복 선언이 가능하다
- let, const 키워드로 선언한 변수는 중복 선언이 불가능하다.

### 동적 타이핑
- 변수의 타입 지정 없이 값이 할당되는 과정에서 값의 타입에 의해 자동으로 타입이 결정된다
  - 같은 변수에 여러 타입의 값을 할당할 수 있다

### 변수 호이스팅
- var 선언문이나 function 선언문 등 모든 선언문이 해당 Scope의 선두로 옮겨진 것처럼 동작하는 특성
  - var, let, const, function, function*, class가 선언되기 이전에 참조가 가능하다

- 변수의 생성 3 단계
  1. 선언 단계: 변수 객체에 변수를 등록한다
     - 이 변수 객체는 스코프가 참조하는 대상이 된다
  2. 초기화 단계: 변수 객체에 등록된 변수를 메모리에 할당한다
     - 변수는 undefined로 초기화된다
  3. 할당 단계: undefined로 초기화된 변수에 실제 값을 할당한다

- var 키워드로 선언된 변수는 선언 단계와 초기화 단계가 한 번에 이루어진다.
  - 스코프에 변수가 등록되고 변수는 메모리에 공간을 확보한 후 undefined로 초기화된다
- let, const 키워드는 호이스팅이 안되는 것은 아니다
  - 변수 객체에 변수를 등록하고, 초기화가 이루어지지 않은 것이다.
  - 이에 반해 var는 변수가 등록됨과 동시에 초기화가 이루어진다

### var의 문제점
- 함수 레벨 스코프
  - 전역 변수의 남팔
  - for loop에서 사용한 변수를 어디서나 사용할 수 있다
- var 키워드 생략 허용
- 중복 선언 허용
- 변수 호이스팅


## 연산자
### 동등 / 일치 연산자
- 동등 연산자: ==, !=
  - x와 y가 값이 같다
- 일치 연산자: ===, !==
  - x와 y의 값과 타입이 같다

## 타입 변환
JS의 모든 값은 타입이 있고, 개발자에 의하거나 자바스크립트 엔진에 의해서 타입이 변환될 수 있다. 의도적으로 타입을 변환하는 것을 명시적 타입 변환이라하고, 엔진에 의해 암묵적으로 타입이 변환되는 것을 암묵적 타입 변환이라고 한다.

### 암묵적 타입 변환
- 자바스크립트 엔진은 표현식을 평가할 때 컨텍스트를 고려하여 암묵적 타입 변환을 실행한다.
- 변수 값을 재할당해서 변경하는 것이 아니라 엔진이 표현식을 에러없이 평가하기 위해 기존 값을 바탕으로 새로운 타입의 값을 만들어 한 번 사용하고 버린다.
- 버려진 값은 GC에 의해 처리된다

### 명시적 타입 변환
- 각종 메서드나 연산자를 사용해 의도적으로 타입을 변환한다.

### falsy
- false
- undefined
- null
- 0, -0
- NaN
- ""

### trusy
- falsy를 제외한 모든 값

## 객체(Object)
자바스크립트는 원시 타입의 값들을 제외한 모든 값들이 객체이다. 자바스크립트의 객체는 키와 값으로 구성된 프로퍼티들의 집합이다. 자바스크립트의 함수는 일급 객체이므로 값으로 취급할 수 있다. 따라서 프로퍼티 값으로 함수를 사용할 수 있고, 프로퍼티 값이 함수인 경우 일반적인 함수와 구분하기 위해서 메서드라고 한다.

### 객체 생성 방법
- 객체 리터럴
- new 연산자 + Object 생성자 함수
  - 일반 함수와 생성자 함수를 구분하기 위해 생성자 함수의 이름을 파스칼 케이스(Pascal Case)로 하는 것이 일반적이다.
  - 객체가 소유하고 있지 않는 프로퍼티 키에 값을 할당하면 객체에 프로퍼티를 추가하고 값을 할당한다

- 생성자 함수
  - 생성자 함수의 이름은 일반적으로 대문자로 시작한다
  - 프로퍼티 또는 메소드명 앞에 기술한 this는 생성자 함수가 생성할 인스턴스를 가리킨다
  - this에 바인딩되어 있는 프로퍼티와 메서드는 public이다
  - 생성자 함수 내에서 선언된 일반 변수는 private이다

### for ...in
- for-in문을 사용하면 객체에 포함된 모든 프로퍼티에 대해 루프를 수행할 수 있다

## 함수
어떤 작업을 수행하기 위해 필요한 문(statement)들의 집합을 정의한 코드 블록이다. 함수는 이름과 매개변수를 가지며 필요한 때에 호출하여 코드 블록에 담긴 문들을 일괄적으로 실행할 수 있다.

### 함수 정의
- 함수 선언문
```javascript
function foo(param) {
    console.log('hello');
    return param * 2;
}
```

- 함수 표현식
  - 자바스크립트의 함수는 일급 객체이다
    - 무명의 리터럴로 표현이 가능하다
    - 변수나 자료구조에 저장할 수 있다
    - 함수의 파라미터로 전달할 수 있다
    - 반환값으로 사용할 수 있다
```javascript
// 익명 함수
var foo = function(number) {
    console.log('hello');
    return number * 2;
}

// 기명 함수
// 변수에는 함수를 참조하는 참조 값이 저장된다.
var foo = function double(number) {
    console.log('hello');
    return number * 2;
}
```
  - 변수가 아닌 기명 함수의 함수명을 사용해 호출하면 에러가 발생한다.
    - 함수 표현식에서 사용한 함수명은 외부 코드에서 접근이 불가능하기 때문이다

- Function 생성자 함수
  - 함수 선언문과 함수 표현식은 모두 함수 리터럴 방식으로 함수를 정의한다. 
  - 이것은 결국 내장 함수 Function 생성자 함수로 함수를 생성하는 것을 단순화 시킨 것이다.


### 함수 호이스팅
```javascript
var res = square(5);

function square(number) {
  return number * number;
}
```
- 함수 선언문으로 함수가 정의 되기전 함수 호출이 가능하다
- 함수 선언의 위치와는 상관 없이 코드 내 어느곳에서든지 호출이 가능한 것을 **함수 호이스팅**이라한다.
- 함수 선언문으로 정의된 함수는 자바스크립트 엔진이 스크립트가 로딩되는 시점에 바로 초기화 하고 이를 VO(Variable Object)에 저장한다
  - 함수 선언, 초기화, 할당이 한 번에 이루어진다
```javascript
var res = square(5); // TypeError: square is not a function

var square = function(number) {
  return number * number;
}
```
- 이 처럼 함수 표현식으로 정의한 경우 에러가 발생한다
- 이는 함수 호이스팅이 아니라 변수 호이스팅이 발생하기 때문이다
  - 변수 호이스팅은 변수 생성 및 초기화와 할당이 분리되어 진행된다.
  - 호이스팅된 변수는 undefined로 초기화되고 실제 값의 할당은 할당문에서 이루어진다

## 함수 객체의 프로퍼티
함수는 객체이므로 프로퍼티를 가질 수 있다.
```javascript
function square(number) {
  return number * number;
}

square.x = 10;
square.y = 20;

console.log(square.x, square.y);
```
### arguments
- 함수 호출 시 전달된 인수들의 정보를 담고 있는 순회 가능한(iterable) 유사 배열 객체이며 함수 내부에서 지역 변수처럼 사용된다
- 유사 배열 객체란 실제 배열이 아니고, length 프로퍼티를 가진 객체를 의미한다


### caller
- 자신을 호출한 함수

### length
- 함수 정의 시 작성된 매개 변수의 개수

### name
- 함수 명
- 기명 함수의 경우 함수 명을 값으로 갖고, 익명 함수의 경우 빈문자열을 값으로 갖는다
### \_\_proto__ 접근자 프로퍼티
- 모든 객체는 `[[Prototype]]`이라는 내부 슬롯이 있다
- `[[Prototype]]`은 프로토타입 객체을 가리킨다
- 프로토타입 객체란 프로토타입 기반 객체지향 프로그래밍의 근간을 이루는 객체로, 객체간 상속을 구현하기위해 사용된다

### prototype
- 함수 객체만이 소유하는 프로퍼티이다.
```javascript
// 함수 객체는 prototype 프로퍼티를 소유한다.
console.log(Object.getOwnPropertyDescriptor(function() {}, 'prototype'));
// {value: {…}, writable: true, enumerable: false, configurable: false}

// 일반 객체는 prototype 프로퍼티를 소유하지 않는다.
console.log(Object.getOwnPropertyDescriptor({}, 'prototype'));
// undefined
```

## 함수의 다양한 형태
### 즉시 실행 함수
- 함수의 정의와 동시에 실행되는 함수
- 최초 한 번만 호출되며 다시 호출할 수는 없다
```javascript
// 기명 즉시 실행 함수(named immediately-invoked function expression)
(function myFunction() {
  var a = 3;
  var b = 5;
  return a * b;
}());

// 익명 즉시 실행 함수(immediately-invoked function expression)
(function () {
  var a = 3;
  var b = 5;
  return a * b;
}());

// SyntaxError: Unexpected token (
// 함수선언문은 자바스크립트 엔진에 의해 함수 몸체를 닫는 중괄호 뒤에 ;가 자동 추가된다.
function () {
  // ...
}(); // => };();

// 따라서 즉시 실행 함수는 소괄호로 감싸준다.
(function () {
  // ...
}());

(function () {
  // ...
})();
```
- 자바스크립트는 파일이 분리되어있더라도 글로벌 스코프가 하나이며, 글로벌 스코프에 선언된 변수나 함수는 코드 내의 어디서든지 접근이 가능하다
  - 따라서 즉시 실행 함수 내에 처리 로직을 모아두면 변수명 혹은 함수명의 충돌을 방지할 수 있다

### 내부 함수
- 함수 내부에 정의된 함수
```javascript
function parent(param) {
  var parentVar = param;
  function child() {
    var childVar = 'lee';
    console.log(parentVar + ' ' + childVar); // Hello lee
  }
  child();
  console.log(parentVar + ' ' + childVar);
  // Uncaught ReferenceError: childVar is not defined
}
parent('Hello');
```
### 콜백 함수
- 함수를 명시적으로 호출하는 것이 아니라 특정 이벤트가 발생헀을 때 시스템에 의해 호출되는 함수
```html
<!DOCTYPE html>
<html>
<body>
  <button id="myButton">Click me</button>
  <script>
    var button = document.getElementById('myButton');
    button.addEventListener('click', function() {
      console.log('button clicked!');
    });
  </script>
</body>
</html>
```

## 프로토타입 
자바스크립트의 모든 객체는 자신의 부모 역할을 담당하는 객체와 연결되어 있다. 그리고 이것은 마치 객체 지향의 상속 개념과 같이 부모 객체의 프로퍼티 또는 메소드를 상속받아 사용할 수 있게 한다. 이러한 부모 객체를 프로토타입 객체 또는 프로토타입이라 한다.

- ECMAScript Spec에서 자바스크립트의 모든 객체는 `[[Prototype]]`이라는 인터널 슬롯을 가진다
- `[[Prototype]]`의 값은 null 또는 객체이며 상속을 구현하는데 사용된다
- `[[Prototype]]` 객체의 데이터 프로퍼티는 get은 허용하나 set은 허용하지 않는다
- `[[Prototype]]`은 `__proto__`프로퍼티로 접근할 수 있다
  - `__proto__`에 접근하면 내부적으로 `Object.getPrototypeOf`가 호출된다


### [[Prototype]] vs prototype 프로퍼티
- [[Prototype]]
  - 함수를 포함한 모든 객체가 가지고 있는 인터널 슬롯
  - 객체 입장에서 부모 역할을 하는 프로토타입을 가리키며, 함수 객체의 경우 Function.prototype을 가리킨다

- prototype 프로퍼티
  - 함수 객체만 가지고있는 프로퍼티이다
  - 함수 객체가 생성자로 사용될 때 이 함수를 통해 생성될 객체의 부모 역할을 하는 객체를 가리킨다