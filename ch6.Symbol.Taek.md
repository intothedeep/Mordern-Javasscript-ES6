
# 6장 심벌과 심벌 프로퍼티

  * [6.1 심벌 만들기](#6-1)
  * [6.2 심벌 사용하기](#6-2)
  * [6.3 심벌 공유하기](#6-3)
  * [6.4 심벌 타입 변환](#6-4)
  * [6.5 심벌 프로퍼티 탐색](#6-5)
  * [6.6 상용 심벌과 내부 연산자 노출하기](#6-6)
    * [6.6.1 Symbol.hasInstance](#6-6-1)
    * [6.6.2 Symbol.isConcatSpreadable 프로퍼티](#6-6-2)
  * [6.7 요약](#6-7)


```txt
primitive value: string, number, boolean, null, undefined

# es6
symbol
- 새로운 원시 타입
- 비공개 객체 멤버를 만들기 위한 한 가지 방법으로 출발했다.
- 기존: 문자열 이름을 가진 모든 프로퍼티에 그 이름을 알고 있는지와 상관없이 쉽게 접근할 수 있었다.
- '비공개 이름' -> 개발자가 문자열이 아닌 프로퍼티 이름을 만들 수 있게 됨 -> 일반적인 방식으로는 비공개 이름을 알아낼 수 없다.
- 이런 개념이 es6에서 symbol로 발전

- 심벌을 통해 문자열이 아닌 프로퍼티 이름을 추가했지만, 외부 접근 방지를 위한 목표는 제외
- 심벌 프로퍼티는 다른 객체 프로퍼티와는 별개로 분류된다.
```


## <a id='6-1'></a>[6.1 심벌 만들기](#6-1)

__boolean의 true나 숫자의 30처럼 문자 리터럴 형태가 없다.__
```js
var person = {};
person["firstName"] = "victor";

let firstName = Symbol();
let person = {};
person[firstName] = "victor";
console.log(person[firstName]); // victor


// 심벌의 서술 문자열
// 심벌을 쉽게 읽고 디버깅할 수 있도록 항상 제공하는 것이 좋다.
firstName = Symbol("first name");
console.log("first name" in person); // false
console.log(person[firstName]); // "victor"
console.log(firstName); // "Symbol(first name)"

// typeof 사용
console.log(typeof firstName); // "symbol"
```
__장점__:
- 프로퍼티를 할당할 때 심벌을 사용하면, 같은 프로퍼티에 접근하려 할 때마다 그 심벌을 사용해야 한다.

__특징__:
- 심벌의 서술 문자열은 [[Description]] 프로퍼티에 저장되고 심벌의 toString() 메소드가 호출될 때 사용

## <a id='6-2'></a>[6.2 심벌 사용하기](#6-2)

Object.defineProperty()
Object.defineProperties()
계산된 객체 리터럴 프로퍼티 이름


```js
let firstName = Symbol("first name");

// 계산된 객체 리터럴 프로퍼티에 사용
let person = {
  [firstName]: "victor"
};

// 프로퍼티를 읽기 전용으로 만듦
Object.defineProperty(person, firstName, {writable: false});

let lastName = Symbol("last name");
Object.defineProperties(person, {
  [lastName]: {
    value: "Lim",
    writable: false
  }

});
console.log(person[firstName]); // "victor"
console.log(person[lastname]); // "Lim"
console.log(Reflect.ownKeys(person));  // ['key', 'method', Symbol(symbol), Symbol(symbol2)]

```


## <a id='6-3'></a>[6.3 심벌 공유하기](#6-3)


__전역 심벌 저장소__:
- 심벌을 저장해 놓고 검색해서 사용
- Symbol.for(String 심벌 서술 문자열): 있으면 새로운 해당 심벌을 검색해서 가져오고 없으면 새로운 심벌을 만든다


```js
const uid = Symbol.for("uid 111"); // 새로운 심벌
let obj = {
  [uid]: "12345"
};

console.log(obj[uid]); // "12345"
console.log(uid); // "Symbol(uid)"

let uid2 = Symbol.for("uid 111"); // 기존 uid 심벌~ from global symbol storage
console.log(uid === uid2); // "12345"
console.log(obj[uid2]); // "12345"
console.log(uid2); // "Symbol(uid)"
```

__Symbol.keyFor(String 심벌 서술 문자열)__: 전역 심벌 저장소에서 해당 심벌 서술 문자열을 가진 심벌과 관련된 키를 찾을 수 있다.

```js
const uid = Symbol.for("uid 111");
console.log(Symbol.keyFor(uid)); // "uid"

const uid2 = Symbol.for("uid 111");
console.log(Symbol.keyFor(uid2)); // "uid"

const uid3 = Symbol("uid 111"); // 심벌 전역 저장소 사용x
console.log(Symbol.keyFor(uid3)); // undefined
```

__주의사항__: 전역 심벌 저장소는 전역 스코프처럼 공유된 환경이다. 이는 그 환경에 키가 이미 존재하는지 아닌지에 대해 추정할 수 없다는 의미이다. 써드파티 컴포넌트를 사용할 때 이름 충돌의 가능성을 줄이기 위해서는 심벌 키에 네임스페이스를 사용해야 한다.
```js
const isArray = Symbol('isArray');
const isRegExp = Symbol('isArray');
const isDate = Symbol('isArray');
const isObject = Symbol('isArray');
Array[isArray] = arg => {
  // 코드 작성
};
RegExp[isRegExp] = arg => {
  // 코드 작성
};
Date[isDate] = arg => {
  // 코드 작성
};
Object[isObject] = arg => {
  // 코드 작성
};
```
전역공간에 노출 시키기 싫을 때!

```js
const $SYMBOL = { // 전역 공간에 노출되므로 그 어떤 이름과도 충돌이 나지 않게 지어야한다.
  isArray: Symbol('isArray'),
  isRegExp: Symbol('isArray'),
  isDate: Symbol('isArray'),
  isObject: Symbol('isArray')
};
Array[$SYMBOL.isArray] = arg => {
  // 코드 작성
};
RegExp[$SYMBOL.isRegExp] = arg => {
  // 코드 작성
};
Date[$SYMBOL.isDate] = arg => {
  // 코드 작성
};
Object[$SYMBOL.isObject] = arg => {
  // 코드 작성
};
```
**reference**: [Symbol의 사용 사례](https://blog.perfectacle.com/2017/04/16/es6-symbol/#Symbol%EC%9D%98-%EC%82%AC%EC%9A%A9-%EC%82%AC%EB%A1%80)


## <a id='6-4'></a>[6.4 심벌 타입 변환](#6-4)
- 심벌은 타입 변환에 유연하지 않다. why? 다른 타입들과 논리적 통치가 성립 X

```js
let uid = Symbol.for("uid");
let desc = String(uid); // "Symbol(uid)"

desc = uid + ""; // 에러
desc = uid / 1; // 에러

if (uid) {
  // 실행~
};
```

## <a id='6-5'></a>[6.5 심벌 프로퍼티 탐색](#6-5)


__Object.keys()__: 열거 가능한 모든 프로퍼티 이름을 반환
__Object.getOwnPropertyNames()__: 열거 가능 여부와 상관없이 모든 프로퍼티 이름 반환

, but 심벌 프로퍼티 반환 x (es5와 호환성 문제)

__Object.getOwnPropertySymbols()__: 객체가 소유한 프로퍼티 심벌을 반환


```js
let uid = Symbol.for("uid victor");
let obj = {
  [uid]: "lim~~~",
  nick: "dio",
  age: 34,
  get age() {
    return this.age;
  },
  sayHello() {
    console.log(this.nick + " is saying hello!");
  },
  "job": "developer"
};

let keys = Object.keys(obj);
let values = Object.values(obj);
let propertyNames = Object.getOwnPropertySymbos(obj);
let propertySymbols = Object.getOwnPropertySymbols(obj);

console.log(keys);
console.log(values);
console.log(propertyNames);
console.log(propertySymbols);

```


## <a id='6-6'></a>[6.6 상용 심벌과 내부 연산자 노출하기](#6-6)
__es5__: 자바 스크립트의 '마법 같은'부분 일부를 노출하고 정의하는 것
__es6__: 특정 객체의 기본 동작 정의에 심벌 프로토타입 프로퍼티를 사용하여, 이전에는 언어의 내부 로직이었던 부분을 훨씬 더 많이 노출하는 방식으로 전통을 이어나감
- __결과__: 내부 전용 연산자로 여겨지던 공통 동작을 상용 심벌(well-known symbols)로 미리 정의
  - __상용 심벌__: Symbol.match처럼 Symbol 객체의 프로퍼티로 나타냄
    - Symbol.hasInstance
    - Symbol.isConcatSpreadable
    - Symbol.iterator
    - Symbol.match
    - Symbol.replace
    - Symbol.search
    - Symbol.species
    - Symbol.split
    - Symbol.toPrimitive
    - Symbol.toStringTag
    - Symbol.unscopables


### <a id='6-6-1'></a>[6.6.1 Symbol.hasInstance](#6-6-1)
__instanceof__: __Array[Symbol.hasInstance](obj)__ >> 동일
```js
obj instanceof Array

Array[Symbol.hasInstance](obj)
```
- es6에서는 기본적으로 instanceof 연산자가 이 메서드 호출의 단축 문법으로 재정의
- __*instanceof가 메서드 호출과 연결되었으므로 instanceof의 동작을 변경할 수 있다!!!!*__

```js
function MyObject() {
  // ...
}

Object.defineProperty(MyObject, Symbol.hasInstance, {
  value: function(v) {
    return false;
  }
});

let obj = new MyObject();
console.log(obj instanceof MyObject);

```
- __쓰기 불가능한 프로퍼티를 덮어쓰기 위해서는 Object.defineProperty()__ >> 해보기

```js
function SpecialNumber() {
  // 비어 있음
}

Object.defineProperty(SpecialNumber, Symbol.hasInstance, {
  value: function(v) {
    return (v instanceof Number) && (v >=1 && v <= 100);
  }
});

let two = new Number(2), zero = new Number(0);

console.log(two instanceof SpecialNumber); // true
console.log(zero instanceof SpecialNumber); // false
```


### <a id='6-6-2'></a>[6.6.2 Symbol.isConcatSpreadable 프로퍼티](#6-6-2)

```js
let colors1 = ["red", "green"];
let colors2 = colors1.concat(["blue", "black"]);

console.log(colors.length); // 4
console.log(colors2); // ["red", "green", "blue", "black"]

// 배열이 아닌 인자도 받을 수 있다! 끝에 추가
colors2 = colors1.concat(["blue", "black"], "brown");
console.log(colors2.length); // 5
console.log(colors2); // ["red", "green", "blue", "black", "brown"]

```

- 어떤 타입이든 concat() 호출 시 배열처럼 동작하도록 정의할 수 있다.
```js
let collection = {
  0: "Hello",
  1: "world",
  length: 2,
  [Symbol.isConcatSpreadable]: true;
};

let messages = ["Hi"].concat(collection);
console.log(messages.length); // 3
console.log(messages); // ["Hi", "Hello", "world"]
```
- 특징
  - collection 객체에 length 프로퍼티 두 숫자 키를 갖도록 하여 배열로 보이게 설정
  - [Symbol.isConcatSpreadable]: true >> 배열에 개별 요소로 추가 되도록 했다.


### <a id='6-6-3'></a>[6.6.3 Symbol.match와 Symbol.replace, Symbol.search, Symbol.split 프로퍼티](#6-6-3)
- 문자열 타입은 정규표현식을 인자로 받는 몇 가지 메서드를 가지고 있다.
  - match(regex)
  - replace(regex, replacement)
  - search(regex)
  - split(regex)
__이 네 가지 메서드에 해당하는 심벌들을 정의하고, 고유 동작을 내장 객체인 RegExp에 효과적으로 위탁한다__

Symbol.match, Symbol.replace, Symbol.search, Symbol.split 심벌은 각각
- match(regex)
- replace(regex, replacement)
- search(regex)
- split(regex)
위 메서드의 첫 번째 인자에 전달되는 정규 표현식 인자의 메서드를 나타낸다.
네 가지 심벌 프로퍼티는 문자열 메서드가 사용해야 하는 기본 구현으로, RegExp.prototype에 정의 된다.

```txt
Symbol.match: 인자 받고 매칭되는 배열 반환, 없으면 null
Symbol.replace: 문자열 인자와 대체 문자열을 받고 대체된 문자열 반환
Symbol.search: 문자열 인자를 받아 매칭된 숫자 인덱스를 반환, 없으면 -1
Symbol.split: 문자열 인자를 받아 매칭되는 문자열로 나눠진 문자열 배열을 반환
```


```js
// 사실상 /^.{10}$/와 같다. >> .에 해당하는 10개로만 이루어진 문자열
let hasLengthOf10 = {
    [Symbol.match]: function(value) {
      return value.length === 10 ? [value] : null;
    },
    [Symbol.replace]: function (value) {
      return value.length === 10 ? replacement : value;
    },
    [Symbol.search]: function (value) {
      return value.length === 10 ? 0 : -1;
    },
    [Symbol.split]: function (value) {
      return value.length === 10 ? ["", ""] : [value];
    }
};


```


### <a id='6-6-4'></a>[6.6.4 Symbol.toPrimitive 메서드](#6-6-4)
- 자바스크립트에서는 종종 특정 연산이 적용될 때 암묵적으로 객체를 원시값으로 변환하려고 시도한다. (e.g. ==)
- Symbol.toPrimitive를 통해서 제어 가능
  - Symbol.toPrimitive 메서드는 각 표준 타입의 포로토타입에 정의되고 객체가 원시값으로 변환될 때 무엇을 해야 할지 규정한다.
  - 원시값 변환이 필요할 때 명세에서 hint로 불리는 인자와 함께 호출
    - __hint__
      - string
      - Number
      - default

- 숫자 모드
  - valueof() 메서드를 호출하고, 그 결과가 원시 값이면 반환한다.
  - 그렇지 않으면, toString() 메서드를 호출하고, 그 결과가 원시 값이면 반환한다.
  - 그렇지 않으면, 에러

- 문자열 모드
  - toString() 메서드를 호출하고, 그 결과가 원시 값이면 반환한다.
  - 그렇지 않으면, valueOf() 메서드를 호출하고, 그 결과가 원시 값이면 반환
  - 아니면 에러

- 기본 모드
  - 많은 경우 숫자 모드와 동일하게 처리
  - Date >> 문자열 모드
  - ==, +, Date 생성자에 인자 하나를 전달하는 경우에만 사용된다.
  - __Symbol.toPrimitive 메서드를 정의하여, 이러한 기본 타입 변환 동작을 오버라이드 할 수 있다.__

```js
function Temperature(degrees) {
  this.degrees = degrees;
}

Temperature.prototype[Symbol.toPrimitive] = function(hint) {

  switch (hint) {
    case "string":
      return this.degrees + "\u00b0"; // 온도 기호
    case "number":
       return this.degrees;
    case "default":
      return this.degrees + " degrees";
  }

};
let freezing = new Temperature(32);

console.log(freezing + "!"); // "32 degrees!"
console.log(freezing / 2); // 16
console.log(String(freezing)); // "32'"
```

### <a id='6-6-5'></a>[6.6.5 Symbol.toStringTag 프로퍼티](#6-6-5)
- 다양한 전역 실행환경 존재
  - 데이터 전달 문제 x
  - 객체가 다른 객체들 사이에서 전달된 후에 처리하려는 객체의 타입을 식별하려 할 때 발생
    - iframe to 부모 or vice versa
      - why? 영역(Realm)을 각각 다르게 나타낸다.
      - 각 영역은 전역 객체의 복사본을 소유한 전역 스코프를 가진다.
        - 배열이 어느 영역에서 만들어졌든지, 그것은 분명히 하나의 배열이다.
          - 그러나 한 영역에서 만들어진 배열이 다른 영역에 전달되었을 때 instanceof Array를 호출하면 false 반환
            - why???????????? >> __배열이 다른 영역의 생성자로 만들어졌고 Array는 현재 영역의 생성자를 나타내기 때문이다__

#### 식별 문제를 우회하는 방법
- 기존: toString() 호출하면 예측 가능한 문자열 반환
```js
function isArray(value) {
  return Object.prototype.toString.call(value) === "[object Array]";
}
console.log(isArray([])); // true
```
- 번역을 이상하게 한듯...


- es5 이전
  - JSON 전역 객체를 만들 때 더글러스 크락포드(Douglas Crockford)의 json2.js 사용
```js
// 전역 객체가 자바 스크립트 환경에서 제공되는지 아니면 다른 라이브러리를 통해서 제공되는지 파악하는 것이 필수적이게 되었다.
// isArray() 방식으로 파악하는 함수를 사용하게 됨
function supportsNativeJSON() {
  return typeof JSON !== "undefined" && Object.prototype.toString.call(JSON) === "[Object JSON]";
}
```
- native: [Object JSON]
- 아닐 때: [Object object]

####ECMAScript 6의 객체 문자열 태그 정의
- Symbol.toStringTag 심벌을 통해 위에서 사용한 toString()을 재정의 가능
```js
function Person(name) {
  this.name = name;
}

Person.prototype[Symbol.toStringTag] = "Person";
let me = new Person("victor");
console.log(me.toString()); // "[object Person]"
console.log(Object.prototype.toString.call(me)); // "[object Person]"

Person.prototype[Symbol.toStringTag] = "Array";

Person.prototype.toString = function() {
  return this.name;
};

me = new Person("victor");

console.log(me.toString()); // "victor"
console.log(Object.prototype.toString.call(me)); // "[object Array]"

Array.prototype[Symbol.toStringTag] = "Magic";
let values = [];

console.log(Object.prototype.toString.call(values)); // "[object Magic]"
```

신기하네~~~~


### <a id='6-6-6'></a>[6.6.6 Symbol.unscopables 프로퍼티](#6-6-6)
- with 문: 반복을 피하려는데.. 복잡해... // strict 모드 사용 불가

```js
var values = [1, 2, 3];
var colors = ["red", "green", "blue"];
var color = "black";

with(colors) {
  push(color);
  push(...values);
}

console.log(colors); // ["red", "green", "blue", "black", 1, 2, 3]
```
- with 문: push를 지역 바인딩 >>> colors
- __es6에서 Array에 values 메소드 추가__ >> 결과적으로 with 문 내에서 ...values는 밖의 values가 아닌 배열의 values 메서드 참조


- 해결: Symbol.unscopables: with문 내 어떤 프로퍼티 바인딩을 만들면 안 되는지 명시하기 위해 Array.prototype 에서 사용
```js

Array.prototype[Symbo.unscopables] = Object.assign(Object.create(null), {
  copyWithin: true,
  entries: true,
  fill: true,
  find: true,
  findIndex: true,
  keys: true,
  values: true
});

```
- 이 메서드의 바인딩은 with문 내에 만들어지지 않고, 아무런 문제 없이 오래된 코드에서 잘 동작하도록 해준다.


## <a id='6-7'></a>[6.7 요약](#6-7)
- 새로운 원시값
- 심벌 참조 없이 접근할 수 없는 열거 불가능 프로퍼티를 만드는데 사용
- 완벽한 비공개 x,  but 모든 개발자로부터 보호할 필요가 있는 기능에 적합하다.
- 심벌 서술 문자열
- 전역 심벌 저장소
- Object.getOwnPropertySymbols()
- Object.defineProperty, Object.defineProperties를 호출 해 여전히 심벌 프로퍼티를 변경할 수 있다.
- Symbol.을 사용해 표준 객체의 기능을 다양한 방법으로 수정할 수 있게 됨
