
# 3장 Function

  * 3.1

## 3.1 함수의 매개변수 기본값
자바 스크립트의 함수는 함수 정의에 선언된 매개변수의 갯수에 상관없이 몇 개의 매개변수든 전달할 수 있도록 허용하는 특징이 있다.

### 3.1.1 ECMAScript 5의 매개변수 기본값
__ECMA5__ 매개변수 기본값 정하기
```js
function makeRequest(url, timeout, callback) {
  timeout = timeout || 2000;
  callback = callback || function () {};
  
  // 나머지
  
}
```

timeout과 callback은 값이 없는 경우 기본값을 제공하기 때문에 __사실상 선택적인 매개변수__
하지만, 위와 같은 방식에는 문제점이 있는데 timeout에 0이 전달 될 경우 OR연산자의 앞 부분에서 false로 판단해 기본값 2000이 timeout에 할당된다.

```js
function makeRequest(url, timeout, callback) {
  timeout = (typeof timeout !== "undefined") ? timeout|| 2000;
  callback = (typeof callback !== "undefined") ? callback|| function () {};
  
  // 나머지
  
}
```
그래서 es5에서는 함수에서 기본값을 할당할 때 위와 같은 방식을 사용했다.

### 3.1.2 ECMAScript 6의 매개변수 기본값
__ECMA6__ 매개변수가 전달되지 않았을 때 실행되는 초기화 구문을 제공
```js
function makeRequest(url, timeout = 2000, callback = function (){}) {

  // 나머지
  
}
```
함수의 선언 시 인자의 위치에 관계없이 모든 인자에 기본 값을 명시할 수 있다.
```js
function makeRequest(url, timeout = 2000, callback) {

  // 나머지
  
}
```
이 함수 같은 경우는 두 번째 인자를 전달하지 않거나 명시적으로 undefined로 전달했을 때에만 사용될 것이다.

* **주의사항**
__null__ 값은 유효한 값
```js
makeRequest("/foo", null, function(body) {
  doSomething(body);
});
// null은 유효한 값으로 간주되어 timeout의 매개변수 기본값은 사용하지 않는다.
```
### 3.1.3 매개변수 기본값이 arguments 객체에 미치는 영향
매개변수 기본값이 존재할 때 arguments 객체는 다르게 동작한다.
__ECMA5__

__non-strict 모드__: arguments 객체는 매개변수가 변경되면 함께 변경된다.
```js
function mixArgs(first, second) {
  console.log(first === arguments[0]);
  console.log(second === arguments[1]);
  first = "c";
  second = "d";
  console.log(first === arguments[0]);
  console.log(second === arguments[1]);
}

mixArgs("a", "b");
```
```txt
//출력결과
true
true
true
true
```

__strict 모드__: arguments 객체는 매개변수가 변경되어도 변경x ("use strict";를 사용했을 때)
```txt
//출력결과
true
true
false
false
```


__ECMA6__
ECMA5 strict 모드와 동일하게 동작한다.
***매개변수 기본값의 존재는 arguments 객체를 매개변수로부터 분리한다.***
```js
// mixArgs("a")만 호출 했을 때!
function mixArgs(first, second = "b") {
  console.log(arguments.length); // 1
  console.log(first === arguments[0]); // true
  console.log(second === arguments[1]); // false, why? arguments[1] is "undefined"
  first = "c";
  second = "d";
  console.log(first === arguments[0]); // false
  console.log(second === arguments[1]); // false
}

mixArgs("a");
```

### 3.1.4 매개변수 기본값 표현식
primitive value, 함수, 객체를 매개변수 기본값으로 사용가능하다.

```js
function getValue() {
  return 5;
}

function add(first, second = getValue()) {
  return first + second;
}

console.log(add(1, 1)); // 2
console.log(add(1)); // 6
```
***!주의사항: 두 번째 인자없이 add()가 호출 됐을 때만 getValue()가 호출된다.***

```js
let value = 5;

function getValue() {
  return value++;
}

function add(first, second = getValue()) {
  return first + second;
}

console.log(add(1, 1)); // 2
console.log(add(1)); // 6
console.log(add(1)); // 7
```
***처음 add()를 호출 했을 때는 getValue()를 호출하지 않았다.***

**나중에 선언 된 매개변수의 개본값으로 먼저 선언된 인자를 사용할 수 있다. !단, * 반대는 불가능 * **
```js
function getValue() {
  return 5;
}

function add(first, second = first) {
  return first + second;
}
console.log(add(1, 1)); // 2
console.log(add(1)); // 2
```

### 3.1.5 TDZ에서의 매개변수 기본값

TDZ: Temporal Dead Zone
**let 선언과 유사하게 초기화 이전에 매개변수를 기본값으로 다음 인자를 호출해 사용할 수 없다!**
```js
function getValue(value) {
  return value + 5;
}

function add(first = getValue(second), second) {
  return first + second;
}
console.log(add(1, 1)); // 2
console.log(add(undefined, 1)); // 에러 발생
```
let 바인딩과 유사하게 동작

## 3.2 이름을 명시하지 않은 매개변수 다루기
```js

// object에서 필요한 프로퍼티만 가져오는 함수
// like Underscore.js 라이브러리의 pick() method
function pick(object) {
  let result = Object.create(null); // {}
  
  // 두 번째 매개변수에서 시작 arguments[1~]
  for (let i = 1, len = arguments.length; i < len; i++) {
    result[arguments[i]] = object[arguments[i]];
  }
  
  return result;
}

let book = {
  title: "Understanding ECMAScript 6",
  author: "Nicholas C. Zakas",
  year: 2016
};

let bookData = pick(book, "author", "year");
console.log(bookData.author); //  "Nicholas C. Zakas"
console.log(bookData.year); // 2016
console.log(bookData.title); // undefined

```
매개변수를 하나 이상 처리할 수 있다는 부분이 명확x

arguments[1]부터 탐색...

### 3.2.2 나머지 매개변수
__ECMA6__ 
**나머지 매개변수: ** ...
```js
function pick(object, ...keys) {
  let result = Object.create(null); // {}
  
  // 두 번째 매개변수에서 시작 arguments[1~]
  for (let i = 0, len = keys.length; i < len; i++) {
    result[keys[i]] = object[keys[i]];
  }
  
  return result;
}
```
이렇게 사용할 수 있다.

***!주의사항: 나머지 매개변수는 함수 선언에 명시한 매개변수의 수를 나타내는 함수의 length 프로퍼티에 영향을 주지 않는다. 이 예제에서 pick()의 length 값은 object만 포함하기 때문에 1이다.***

**나머지 매개변수의 제한**:
* 무조건 하나여야 한다.
* 마지막 위치에만 사용할 수 있다.

**나머지 매개변수가 arguments 객체에 미치는 영향**
* 나머지 매개변수와 상관없이 arguments 객체는 함수에 전달된 인자를 항상 정확하게 반영한다.

## 3.3 Function 생성자의 확장된 역활
```js
var add = new Function("first", "second", "return first + second");
var add = new Function("first", "second = first", "return first + second");
var add = new Function("...args", "return args[0]");
```
## 3.4 전개 연산자
전개 연산자: spread operator
```js
let values = [25, 50, 75, 100];
console.log(Math.max.apply(Math, values)); // 100
console.log(Math.max(...values)); // 100 

values = [-20, -50, -75, -100];
console.log(Math.max(...values, 0)); // 0

```

**
ECMA6 나머지 연산자를 사용하면 복잡한 this 바인딩(Math.max.apply())를 사용할 필요가 없다.
또한, 나머지 연산자를 다른 인자와 사용할 수 있다.
**
use Rest parameters other than apply()~~~

## 3.5 name 프로퍼티
__for Trace or Debuging__ 자바스크립트에서 함수 식별의 어려움 >> __name__ porperty 추가

### 3.5.1 적절한 이름 선택하기
```js
function doSomething() { }
var doAnotherThing = function() { }
var a = function b() { }

console.log(doSomething.name); // doSomething
console.log(doAnotherThing.name); // doAnotherThing
console.log(a.name) // b

var person = {
 get firstName(){
  return "Nicholas";
 },
 sayName: function() {
  console.log(this.name);
 }
};
console.log(person.sayName.name); // "sayName"

var descriptor = Object.getOwnPropertyDescriptor(person, "firstName");
console.log(descriptor.get.name); // "get firstName"

```
**!참고: **[Object.getOwnPropertyDescriptor()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptor)


