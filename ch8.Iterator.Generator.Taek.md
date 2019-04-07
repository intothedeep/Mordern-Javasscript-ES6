# ch8 Iterator/Generator

- 컬렉션의 다음 요소를 반환하는 이터레이터 객체를 사용하는 방식으로 옮겨가는 추세이다.
- 데이터를 효율적으로 처리할 수 있다. (why???)



## 8.1 반복문의 문제점

```js
var colors = ["red", "green", "blue"];
for (var i = 0; len = colors.length; i < len; i++) {
    console.log(colors[i]);
}
```

- 문제점
  - 중첩 사용 시
  - 여러 개의 변수 사용 시
- 결론: 코드 복잡도 증가로 인한 에러 발생 증가

## 8.2 이터레이터란?

- 반복을 위해 설계된 인터페이스를 가진 객체

```js
{
    value,
    done: boolean
}
```

```js
function createIterator(items) {
    var i = 0;
    return {
        next: function() {
            var done = (i >= items.length);
            var value = !done ? items[i++] : undefined;
            return {
                done: done,
                value: value
            };
        }
    };
}

var iterator = createIterator([1, 2, 3]);
console.log(iterator.next());
console.log(iterator.next());
console.log(iterator.next());
console.log(iterator.next());
// {done: false, value: 1}
// {done: false, value: 2}
// {done: false, value: 3}
// {done: true, value: undefined}
console.log(iterator.next());
console.log(iterator.next());
```



## 8.3 제네레이터란?

```js
function *createIterator() {
    yield 1;
    yield 2;
    yield 3;
}

let iterator = createiterator();
console.log(iterator.next().value); // 1
console.log(iterator.next().value); // 2
console.log(iterator.next().value); // 3
```

- 제네레이터는 이터레이터를 반환하는 함수
- 앞에 *를 사용
- 새로운 키워드인 yield 사용
- !!!!!! *__yield__ 이후 실행이 멈춘다!* 



```js
function *createIterator(items) {
    for (let i = 0; i <items.length; i++) {
        yield items[i];
    }
}

let iterator = createIterator([1, 2, 3]);

console.log(iterator.next());
console.log(iterator.next());
console.log(iterator.next());
// { value: undefined, done: true}
console.log(iterator.next());
console.log(iterator.next());
```



- ! 중요 yield  키워드는 제네레이터 안에서만 사용될 수 있다. 

```js
function *createiterator(items) {
    items.forEach(function (item) {
       // 문법 에러
        yeild item + 1;
    });
}
```



### 8.3.1 제네레이터 함수 표현식

```js
let createiterator = function *(items) {
    for (let i = 0; i < items.length; i++) {
        yield items[i];
    }
};
```

- !주의: 제네레이터인 화살표 함수를 만드는 것은 불가능하다.



### 8.3.2 제네레이터 객체 메서드

```js
var o = {
    createIterator: function *(items) {
        for (let i = 0; i < items.length; i++) {
            yield items[i];
        }
    }
};

let iterator = o.createIterator([1, 2, 3]);

// or es6 메서드 축약 사용 가능
o = {
    *createiterator(items) {
        for (let i = 0; i < items.length; i++) {
            yield items[i];
        }
    }
};

let iterator = o.createIterator([1, 2, 3]);
```

- !주의: 축약 버전에서는 *를 공백없이 함수명 바로 앞에 사용



## 8.4 이터러블과 for~of문

- 이터러블은 Symbol.iterator 프로퍼티를 가진 객체로, 이터레이터와 밀접한 관련이 있다.
- for~of문은 반복문이 실행될 때마다 이터러블의 next()를 호출하고 반환된 객체의 value를 변수에 저장한다.
  - until done == true

```js
const arr = [1, 2, 3];
for (let x of arr) {
    console.log(x);
}
```



### 8.4.1 기본 이터레이터에 접근하기

- [Symbol.iterator]

```js
let values = [1, 2, 3];
let iterator = values[Symbol.iterator]();

// same as for~of process
console.log(iterator.next());
console.log(iterator.next());
console.log(iterator.next());

// check if an object is a iterator
function isIterable(object) {
    return typeof object[Symbol.iterator] === "function";
}

```



### 8.4.2 이터러블 만들기

- 개발자가 정의한 객체는 **이터러블이 아니다**
- use [Symbol.property] to have an object iterable

```js
let collection = {
    items: [],
    *[Symbol.iterator]() {
        for (let item of this.items) {
            yield item;
        }
    }
};

collection.items.push(1);
collection.items.push(2);
collection.items.push(3);

for (let x of collection) {
    console.log(x);
}
```



## 8.5 내장 이터레이터

- 이터레이터 기본적으로 내장
- when to create iterator?
  - when defined iterator is not appropriate for what you want
  - when define an object or class

### 8.5.1 컬렉션 이터레이터

- es6: 3 가지 컬렉션 객체
  - Arrayer
  - Map
  - Set
- 3 가지 내장 이터레이터
  - entries()
  - values()
  - keys()

entries()

```js
let colors = ["red", "green", "blue"];
let tracking = new Set([1234, 5678, 9012]);
let data = new Map();

data.set("title", "Understanding ECMAScript 6");
data.set("format", "ebook");

for (let entry of colors.entries()) {
    console.log(entry);
}

for (let entry of tracking.entries()) {
    console.log(entry);
}

for (let entry of data.entries()) {
    console.log(entry);
}
```

values()

```js
let colors = ["red", "green", "blue"];
let tracking = new Set([1234, 5678, 9012]);
let data = new Map();

data.set("title", "Understanding ECMAScript 6");
data.set("format", "ebook");

for (let value of colors.values()) {
    console.log(value);
}

for (let value of tracking.values()) {
    console.log(value);
}

for (let value of data.values()) {
    console.log(value);
}
```

keys()

위와 비슷



##### 컬렉션 타입의 기본 이터레이터

Set: values()

Values: entries()

##### 구조 분해와 for~of 문

```js
let data = new Map();
data.set("title", "Understanding ECMAScript 6");
data.set("format", "ebook");
for (let [key, value] of data) {
    console.log(key + " : " + value);
}
```



### 8.5.2 문자열 이터레이터

```js
var message = "A 林 士 B";
for (var i = 0; i < message.length; i++) {
    console.log(message[i]);
    console.log(message.charAt(i));
    console.log(message.charCodeAt(i));
}

for (let c of message)
	console.log(c);
```



### 8.5.3 NodeList 이터레이터

- es6에서 NodeList의 DOM 정의에는 배열 기본 이터레이터와 똑같이 동작하는 기본 이터레이터가 포함된다.
- for~of 문 호출 가능



## 8.6 전개 연산자와 배열이 아닌 이터러블

- 전개 연산자를 사용하는 것이 이터러블을 배열로 바꾸는 가장 쉬운 방법

## 8.7 이터레이터 고급 기능

### 8.7.1 이터레이터에 인자 전달하기

```js
function *createIterator() {
    let first = yield 1;
    let second = yield first + 2;
    yield second + 3;
}

let iterator = createIterator();

console.log(iterator.next()); // { value: 1, done: false }
console.log(iterator.next(4));  // { value: 6, done: false }
console.log(iterator.next(5));  // { value: 8, done: false }
console.log(iterator.next());  // { value: undefined, done: true }
```

### 8.7.2 이터레이터에 에러 발생시키기

```js
function *createIterator() {
    let first = yield 1;
    let second;
    try {
	    second = yield first + 2;    
    } catch (e) {
        second = 6;
    }
    
    yield second + 3;
}

let iterator = createIterator();

console.log(iterator.next()); // { value: 1, done: false }
console.log(iterator.next(4));  // { value: 6, done: false }
console.log(iterator.throw(new Error("Boom")));  // { value: 9, done: false }
console.log(iterator.next());  // { value: undefined, done: true }
```

### 8.7.3 제네레이터 return 문

```js
function *createIterator() {
	yield 1;
    return 3;
	yield 2;
    yield 3;
}

let iterator = createIterator();

console.log(iterator.next()); // { value: 1, done: false }
console.log(iterator.next()); // { value: 3, done: true }
console.log(iterator.next()); // { value: undefined, done: true }
```

### 8.7.4 제네레이터 위임하기

- 종종 두 이터레이터의 값을 하나로 합치는 게 유용한 경우가 있다.
- 제네레이터는 yield와 별표(*)를 함께 사용하여 다른 이터레이터에 동작을 위힘할 수 있다.

```js
function *createNumberIterator() {
    yield 1;
    yield 2;
}

function *createColorIterator() {
    yield "red";
    yield "green";
}

function *createCombinedIterator() {
    yield *createNumberIterator();
    yield *createColorIterator();
    yield true;
}

var iterator = createCombinedIterator();


console.log(iterator.next()); // { value: 1, done: false }
console.log(iterator.next()); // { value: 2, done: false }
console.log(iterator.next()); // { value: "red", done: false }
console.log(iterator.next()); // { value: "green", done: false }
console.log(iterator.next()); // { value: true, done: true }
console.log(iterator.next()); // { value: undefined, done: true }
```

```js
function *createNumberIterator() {
    yield 1;
    yield 2;
    return 3;
}

function *createRepeatingIterator() {
    for (let i = 0; i < count; i++) {
        yield "repeat";
    }
}

function *createCombinedIterator() {
    let result = yield *createNumberIterator();
    yield createRepeatingIterator(result);
}

var iterator = createCombinedIterator();

console.log(iterator.next()); // { value: 1, done: false }
console.log(iterator.next()); // { value: 2, done: false }
console.log(iterator.next()); // { value: "repeat", done: false }
console.log(iterator.next()); // { value: "repeat", done: false }
console.log(iterator.next()); // { value: "repeat", done: false }
console.log(iterator.next()); // { value: undefined, done: true }
```



- return 3 출력하기 with yield

```js
function *createNumberIterator() {
    yield 1;
    yield 2;
    return 3;
}

function *createRepeatingIterator() {
    for (let i = 0; i < count; i++) {
        yield "repeat";
    }
}

function *createCombinedIterator() {
    let result = yield *createNumberIterator();
    yield result;
    yield createRepeatingIterator(result);
}

var iterator = createCombinedIterator();

console.log(iterator.next()); // { value: 1, done: false }
console.log(iterator.next()); // { value: 2, done: false }
console.log(iterator.next()); // { value: 3, done: false }
console.log(iterator.next()); // { value: "repeat", done: false }
console.log(iterator.next()); // { value: "repeat", done: false }
console.log(iterator.next()); // { value: "repeat", done: false }
console.log(iterator.next()); // { value: undefined, done: true }
```

- ex

```js
function *createNumberIterator() {
    yield 1;
    yield 2;
    return 3;
}

function *createCombinedIterator() {
    let result = yield *createNumberIterator();
    yield result;
    yield *"hello!";
}

var iterator = createCombinedIterator();

// 문자열 이터레이터를 사용해 h,e,l,l,o,!를 사용 가능
```

## 8.8  비동기 작업 수행

- 제네레이터 사용 이점?
  - 제네레이터는 실행 중에 코드를 사실상 멈추기 때문에, 비동기 처리와 관련하여 다양한 가능성을 보인다.

```js
// 비동기 with callback
let fs = require("fs");

fs.readFile("config.json", function(err, contents) {
    if (err) {
        throw err;
    } 
    
    doSomthingWith(contents);
    console.log("done!");
});
```

### 8.8.1 간단한 작업 실행기

```js
function run(taskDef) {
    // create iterator ot make it available anywhere
    let task = taskDef();
    
    // task start
    let result = task.next();
    
    // recursive function to call next()
    function step() {
        if(!result.done) {
            result = task.next();
            step();
        }
    }
    
    // call step() function to start!
    step();
}

// example
run(function*() {
 console.log(1);
 yield;
 console.log(2);
 yield;
 console.log(3);
});
```

### 8.8.2 데이터와 함께 작업 실행하기

```js
function run(taskDef) {
    // create iterator ot make it available anywhere
    let task = taskDef();
    
    // task start
    let result = task.next();
    
    // recursive function to call next()
    function step() {
        if(!result.done) {
            result = task.next(result.value);
            step();
        }
    }
    
    // call step() function to start!
    step();
}

// 이터레이터 안과 밖에 값을 전달하기
run(function*() {
   let value = yield 1;
    console.log(value);
    value = yield value + 3;
    console.log(value);
});
```

### 8.8.3 비동기 작업 실행

```js
// callback
function fetchData() {
    return function(callback) {
        setTimeout(function() {
	        callback(null, "Hi!");    
        }, 500);    
    };
}

// with generator!
function run(taskDef) {
    // create iterator ot make it available anywhere
    let task = taskDef();
    
    // task start
    let result = task.next();
    
    // recursive function to call next()
    function step() {
        if(!result.done) {
            if (typeof result.value === "function") {
                result.value(function(err, data) {
                    if (err) {
                        result = task.throw(err);
                        return;
                    }
                    result = task.next(data);
                    step();
                });
            }
        }
    }
    // call step() function to start!
    step();
}
```

```js
// apply with readFile() function
let fs = require("fs");
function readFile(filename) {
    return function(callback) {
      fs.readFile(filename, callback);  
    };
}

// start with yield
run(function*() {
    let contents = yield readFile("config.json");
    doSomethingWith(contents);
    console.log("Done");
});
```





























