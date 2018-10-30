
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

var c = function() {};
console.log(c.bind.name); // "bound c", bind()를 사용해 만든 함수는 이름 앞에 "bound"와 공백이 붙는다.
console.log((new Function()).name); // Function 생성자를 이용해 만든 함수는 "anonymous"이다.

```
참고: [Object.getOwnPropertyDescriptor()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptor)

**getter**와 **setter**는 get과 set을 앞에 붙이고 반환할 때에는 Object.getOwnPropertyDescriptor()를 사용하여 반환해야 한다.

## 3.6 함수의 두 가지 용도를 명확히 하기
```js
function Person(name) {
 this.name = name;
}

var person = new Person("Nicholas");
var notAPerson = Person("Nicholas");

console.log(person); // "[Object object]"
console.log(notAPerson); // "undefined"
```
- notAPerson을 만들 때 new 없이 호출하면 undefined를 반환한다.
- 대문자로 시작하는 Person은 자바스크립트 프로그래밍에서 보통 new를 사용해 호출하는 함수를 가리킨다.
  - 이러한 함수의 이중적인 역할은 혼란을 일으킬 수 있기 때문에 es6에서는 몇 가지 개선이 이루어 졌다.

**자바스크립트 함수에는 [[call]]과 [[Construct]]라는 두 가지 다른 내부 전용 메서드가 있다.**
- 함수를 new 없이 호출하면 call 호출
- 함수를 new와 함께 호출하면 construct 호출 // construct 메소드를 가진 함수를 생성자라고 한다.

### 3.6.1 ECMAScript 5에서 함수 호출 방식을 결정하는 요인
- __instaceof__ 사용
```js
function Person(name) {
 if (this instanceof Person) {
  this.name = name;
 } else {
  throw new Error("You must use new with Person");
 }
}

var person = new Person("victor");
var notAPerson = Person("victor"); // error 발생
```
- this 값을 사용하여 인자가 생성자의 인스턴스인지 확인하고, 만약 그렇다면 계속 실행한다. 그리고 this가 Person의 인스턴스가 아니라면 에러를 발생시킨다.
- 이런한 접근이 가능한 이유는 [[Construct]] 메서드가 Person의 새로운 인스턴스를 만들어 this에 할당하기 때문에 동작함
- ~~하지만 아래 예제의 경우처럼 this가 new를 사용하지 않고도 Person의 인스턴스가 될 수 있기 때문에 이 접근법은 불완전~~

```js
function Person(name) {
 if (this instanceof Person) {
  this.name = name;
 } else {
  throw new Error("You must use new with Person");
 }
}

var person = new Person("victor");
var notAPerson = Person.call(person, "dio"); // 정살 실행
```

- Person.call()은 person 변수를 첫 번째 인자로 전달하여 Person 함수 내부의 this에 person을 설정한다.
- 결국, person 함수에서 person 인스턴스가 new? Person.apply()? 무었을 통해 호출 되었는지 확인 불가능!

### 3.6.2 new.target 메타 프로퍼티
__new.target__
- 작동원리
  - [[Construct]]가 호출되면 new.target에는 new 연산자의 실행대상이 할당
  - [[Call]]이 호출되면 new.target은 undefined

- new.target 사용
```js
function Person(name) {
 if (new.target === Person) {
  this.name = name;
 } else {
  throw new Error("You must use new with Person");
 }
}

var person = new Person("victor");
var notAPerson = Person.call(person, "dio"); // ~~정살 실행 x~~, 에러 발생!
```

- new.target으로 호출한 특정 생성자 확인하기

```js
function Person(name) {
 if (this instanceof Person) {
  this.name = name;
 } else {
  throw new Error("You must use new with Person");
 }
}

function AnotherPerson(name) {
 Person.call(this, name);
}

var person = new Person("victor");
var anotherPerson = new AnotherPerson("dio"); // 에러 발생! [[Construct]]를 사용하지 않고 [[Call]] 메서드 사용
```
* 에러가 나는 이유
 * new.target이 Person이어야 한다. 그런데 AnotherPerson 내부에서 Person의 call()을 사용해 호출하고 있으니 new.target은 undefined
 * 에러 발생!
 
## 3.7 블록 레벨 함수

```js
"use strict";
if (true) {
 // es5에서는 문법 에러가 발생, es6에서는 발생하지 않음
 function doSomething() {
  //...
 }
 //...
}
```
- 에러가 맞지만 사용되고 있었는데 es6에서는 doSomthing()을 블록 레벨 선언으로 간주, 블록 내에서만 접근 및 호출 가능하게 만듬

### 3.7.1 블록 레벨 함수의 사용 시기
- 블록 레벨 함수는 정의된 블록 밖에서는 제거 된다는 점에서 let 함수의 표현식과 동일
- 차이점
  - 블록 레벨 함수는 최상단으로 호이스팅
  - let 함수 표현식은 x

```js
"use strict";

if (true) {
 console.log(typeof doSomething) // "function"
 function doSomething() { }
}

if (true) {
 console.log(typeof doSomething); // 에러 발생!

let doSomething = function () { }
}
```

### 3.7.2 Non-Strict 모드의 블록 레벨 함수

```js
// ECMAScript 6 non-strict 모드

if (true) {
 console.log(typeof doSomething) // "function"
 function doSomething() { }
}

 console.log(typeof doSomething) // "function"

```
- 블록 내부 뿐 아니라 전역에서도 최상단으로 doSomething이 호이스팅 되어 블록 밖에서도 호출 가능하다.
 - 이유 기본에 호환되지 않는 브라우저 동작들을 제거하기 위해 이러한 동작을 표준화했음

## 3.8 화살표 함수
__=>__ 을 사용해 표현

- 기존 자바스크립트 함수와 다른점
   1. this와 super, arguments, new.target 바인딩:
      - 함수 내에서 this와 super, arguments, new.target의 값은 그 화살표 함수를 가장 근접하게 둘러싸고 있는 일반함수에 의해 정의
   2. new를 사용할 수 없음:
      - [[Construct]] 메소드가 없어 생성자로 사용 불가능, 사용 시 Error 발생
   3. 프로토 타입 없음:
      - new를 사용할 수 없기 때문에 프로토타입이 필요 없음. 화살표 함수에는 prototype 프로퍼티가 없다.
   4. this를 변경할 수 없음:
      - 함수 내부의 this 변경 X. this 값은 함수 전체 생명 주기 내내 같은 값
   5. arguments 객체 없음:
      - arguments 바인딩이 없음. 명시한 매개변수와 ...(나머지) 매개변수만 사용가능
   6. 같은 이름의 매개변수를 중복하여 사용 불가능:
      - 기존: strict에서만 중복 사용 X
      - 화살표: **non 및 strict** 중복 사용 x

**이유:** this 바인딩이 자바스크립트 에러의 주요 원인 >> 최적화 가능

### 3.8.1 화살표 함수 문법

```js
var reflect = value => value;
var reflect = function(value) {
 return value;
}

// 인자 2개 이상
var sum = (a, b) => a + b;
var sum = function(a, b) {
 return a + b;
};

// 인자 없을 때
var getName = () => "Egotistical Bastard";

// 하나 이상의 표현식
var sum = function(a, b) {
 console.log(a + b);
 return a + b;
};

// 아무것도 하지 않는 함수
var nothing = () => {};
var nothing = function() {};

// 객체 리터럴을 반환 할 시 ()로 감싸야 한다. 
var getTempItem = id => ({ id: id, name: "Temp"});
var getTempItem = function(id) {
 return {
  id: id,
  name: "bastard"
 };
};

```

### 3.8.2 즉시 실행 함수 표현식 만들기

```js
let person = function(name) {
 return {
  getName: function() {
   return name;
  }
 };
}("victor");

console.log(person.getname()); // "victor"

let person = (
 (name) => {
  return {
   getName: function() {
    return name;
   }
  };
 }
) // ()가 화살표 함수 정의 부분만 감싸고 있다.
("victor");

console.log(person.getname()); // "victor"
```

### 3.8.3 No this 바인딩

```js
let PageHandler = {
 id: "7777777",
 
 init: function() {
  document.addEventListener("click", function(event) {
   this.doSomething(event.type); // 에러 발생! why? this가 객체에 묶인 것이 아닌 event에 바인딩 되어 있음!
  }, false);
 },
 
 doSomething: function(type) {
  console.log("Handling " + type + " for " + this.id);
 }
 
};

// bind()를 사용해 해결
let PageHandler = {
 id: "7777777",
 
 init: function() {
  document.addEventListener("click", (function(event) {
   this.doSomething(event.type); // 에러 발생! why? this가 객체에 묶인 것이 아닌 event에 바인딩 되어 있음!
  }).bind(this), false); // function 밖에 있는 객체로 this를 바인딩 시켜줌
 },
 
 doSomething: function(type) {
  console.log("Handling " + type + " for " + this.id);
 }
 
};

// 화살표 함수
// 화살표 함수는 this 바인딩을 하지 않으므로, 화살표 함수 내 this 값은 스코프 체인을 통해서만 결정된다.
// 만약, 화살표 함수 in 일반 함수 >> this 값은 화살표 함수를 감싸는 함수에서의 값과 같을 것이고, 
// not in 일반 함수 >> 전역 스코프의 this 값과 같다.
let PageHandler = {
 id: "7777777",
 
 init: function() {
  // this의 값은 init 내의 값과 같고 bind(this)를 사용한 것과 비슷하게 동작
  // doSomething은 반환값이 없지만 함수 본문 안에 실행문만 있기에 중괄호로 감쌀 필요가 없다.
  document.addEventListener("click", event => this.doSomething(event.type), false); 
 },
 
 doSomething: function(type) {
  console.log("Handling " + type + " for " + this.id);
 }
 
};

```
* [prototype 프로퍼티](https://medium.com/@bluesh55/javascript-prototype-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-f8e67c286b67)
- 화살표 함수는 __일회성__, 새로운 함수를 정의하기 위해 사용 X
   - 이는 보통 함수에 있는 prototype 프로퍼티가 없음
```js
var arrow = () => {}, 
    obj = new arrow(); // 에러 발생! No [[Constructor]]
```

### 3.8.5 No arguments 바인딩
- 화살표 함수 has no arguments object, but
- 화살표 함수 can access to mother function's arguments object

```js
function args() {
 // args() 함수의 arugments 객체를 전달 받아 사용
 return () => arguments[0];
}

var arrow = args(5);
console.log(arrow()); // 5
```

### 3.8.6 화살표 함수 식별하기
- typeof 화살표 함수 // "Function"
- 화살표 함수 instanceof Function // true
- call(), bind(), apply() 사용 가능 // 단, this 바인딩은 변하지 않음

```js
var sum = (a, b) => a + b;

console.log(sum.call(null, 1, 2)); // 3
console.log(sum.apply(null, [1, 2])); // 3

var boundSum = sum.bind(null, 1, 2);
console.log(boundSum()); // 3 
```

## 3.9 꼬리 호출 최적화
- [꼬리 호출 최적화 관련 동영상](https://www.youtube.com/watch?v=h80tLv0fn88&t=348s)

- ECMA6 함수의 가장 흥미로운 변화: 꼬리 호출 시스템을 변경하는 엔진 최적화
__꼬리 호출__: 함수가 다른 함수의 마지막에 호출될 때를 말한다.
```js
function doSomething() {
 return doSomeThingElse(); // 꼬리 호출
}
```

