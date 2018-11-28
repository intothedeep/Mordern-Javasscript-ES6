# ch9 Class

- 특징: es6의 자바스크립트의 동적 특성을 수용 ????? why is it different from class of other languages?

## 9.1 ECMAScript 5의 유사 클래스 구조

- es5: class가 없었다.
  - 유사 클래스? << 사용자 정의 타입 생성이라는 불리는 방법으로 구현
    - 생성자를 만들고
    - 생성자의 prototype에 메서드 할당

```js
function PersonType(name) {
    this.name = name;
}

PersonType.prototype.sayName = function() {
    console.log(this.name);
};

let person = new PersonType("Nicholas");
person.sayName(); // "Nicholas" 출력

console.log(person instanceof PersonType); // true
console.log(person instanceof Object); // true
```

- [[Construtor]] 사용

## 9.2 클래스 선언

```js
class PersonClass {
    // PersonType 생성자에 해당하는 부분
    constructor(name) {
        this.name = name;
    }
    
    // PersonType.prototype.sayName에 해당하는 부분
    sayName() {
        console.log(this.name);
    }    
}

let person = new PersonClass("Nicholas");
person.sayName(); // "Nicholas" 출력

console.log(person instanceof PersonClass); // true
console.log(person instanceof Object); // true

console.log(typeof PersonClass); // "function" << class >> 함수
console.log(typeof PersonClass.prototype.sayName); // "function"
```

- 사용자 정의 타입과 유사함
  - 섞어서 사용 가능

```js
PersonClass.prototype.sayHello = () => {
    console.log("hello!");
}

person.sayHello();
```

### 9.2.2 왜? 클래스 문법을 사용하는가?

- 함수 선언과 달리 클래스 선언은 호이스팅되지 않는다.
  - 클래스 선언은 let 선언처럼 동작하므로, 실행이 선언에 도달할 때까지 TDZ에 존재한다.
- 클래스 선언 내의 모든 코드는 자동으로 strict 모드에서 실행된다. 
  - 클래스 내에서 strict 모드를 피할 방법은 없다.
- 모든 메서드는 열거할 수 없다?
  - 사용자 정의 타입과 다른 중요 변경사항으로, 사용자 정의 타입에서는 메서드를 열거 불가능하도록 만들기 위해 Object.defineProperty()를 사용해야 한다.
- 모든 메서드에는 내부 메서드 [[Construct]]가 없다! 
  - new와 함께 메서드를 호출하려면 에러가 발생한다.
- new 없이 클래스 생성자를 호출하면 에러가 발생한다.
- 클래스 메서드 내에서 클래스 이름을 덮어쓰려 하면 에러가 발생한다.

```js
let PersonType2 = (function() {
   "use strict";
    
    const PersonType2 = function(name) {
        // check if a function is called with new
        if (typeof new.target === "undefined") {
            throw new Error("Constructor must be called with new.");
        }
        
        this.name = name;
    }
    Object.defineProperty(PersonType2.prototype, "sayName", {
        value: function() {
            // 메서드가 new와 함께 호출되지 않았는지 확인
            if (typeof new.target !== "undefined") {
                throw new Error("Method cannot be called with new!");
            }
            console.log(this.name);
        },
        enumerable: false,
        writable: true,
        configurable: true
	});
    return PersonType2;
}());
// })(); 위 와 같음

```



#### !수정할 수 없는 클래스 이름

- 클래스 이름은 클래스 내부에서 실행할 수 없다.
  - 외부에서는 덮어쓰기 가능 -> 클래스 외부에서는 클래스 변수가 let 선언과 비슷
  - 내부에서는 덮어쓰기 불가능 -> 클래스 내부에서는 클래스 변수가 const 선언처럼 취급

```js
class Foo {
    constructor() {
        Foo = "bar"; // error! 내부에서 덮어쓰기
    }
}

// after declaration of class, it can be replaced
Foo = "bar";
```

## 9.3 클래스 표현식

- 클래스도 함수와 같게 __표현식, 선언식__ 두 가지 형태가 있다.

### 9.3.1 기본 클래스 표현식

```js
let PersonClass = class {
  	// PersonType 생성자와 같음
    constructor(name) {
        this.name = name;
    }
    
    //PersonType의 sayName()과 같음
    sayName() {
        console.log(this.name);
    }
};

let person = new PersonClass("Nicholas");

console.log(person instanceof PersonClass); // true
console.log(person instanceof Object); // true

console.log(typeof PersonClass); // "function" << class >> 함수
console.log(typeof PersonClass.prototype.sayName); // "function"
```

### 9.3.2 이름을 명시한 클래스 표현식

```js
let PersonClass = class PersonClass2 {
    // PersonType 생성자와 같음
    constructor(name) {
        this.name = name;
    }
    
    //PersonType의 sayName()과 같음
    sayName() {
        console.log(this.name);
    }
}

console.log(typeof PersonClass); // "function"
console.log(typeof PersonClass2); // "undefined"
```

- why???
  - PersonClass2는 클래스 정의할 때만 존재
    - 클래스 정의 블록 내에서는 사용 가능, but
      - 클래스 외부에서는 사용 불가능
        - 외부에서는 PersonClass2에 대한 바인딩이 존재하지 않기 때문
          - console.log(typeof PersonClass2); // "undefined" 

```js
let PersonClass = (function() {
   "use strict";
    
    const PersonClass2 = function(name) {
        // check if a function is called with new
        if (typeof new.target === "undefined") {
            throw new Error("Constructor must be called with new.");
        }
        
        this.name = name;
    }
    Object.defineProperty(PersonClass2.prototype, "sayName", {
        value: function() {
            // 메서드가 new와 함께 호출되지 않았는지 확인
            if (typeof new.target !== "undefined") {
                throw new Error("Method cannot be called with new!");
            }
            console.log(this.name);
        },
        enumerable: false,
        writable: true,
        configurable: true
	});
    return PersonClass2;
}());
```

## 9.4 일급 시민 클래스

- **일급 시민**: 
  - 함수에 전달되고
  - 함수로부터 반환되고
  - 변수에 할당될 수 있는 값
- 자바 스크립트 함수: 일급 시민(함수)
- **es6에서는 클래스를 일급 시민으로 만들어 사용**

```js
function createObject(classDef) {
    return new classDef();
}

/*
class Hello {
    sayHello() {
        console.log("hello");
    }
}
let obj = createObject(Hello);
obj.sayHello();
*/
let obj = createObject(class {
    sayHi() {
        console.log("hi");
    }
});

obj.sayHi();
```

```js
let person = new class {
    constructor(name) {
        this.name = name;
    }
    
    sayName() {
        console.log(this.name);
    }
}("Victor");

person.sayName();

// 싱글톤 함수를 만드는 것과 같음
var mySquare = (function (x) {
    return x*x;
})(2);
console.log(mySquare)
```

- 익명의 클래스 표현식이 만들어지는 즉시 실행 (like 즉시 실행 함수)
- **싱글톤을 만들 때 사용**



## 9.5 접근자 프로퍼티

- 객체 리터럴과 유사한 문법을 사용해
  - 클래스에 프로퍼티 접근자를 만들 수 있다.

- 클래스의 프로퍼티는 클래스 생성자 안에서 만들어져야 하지만, but
  - 클래스는 prototype에 접근자 property를 정의하는 것 허용
    - get
    - set

```js
class CustomHTMLElement {
    constructor(element) {
        this.element = element;
    }
    
    get html() {
        return this.element.innerHTML;
    }
    
    set html(value) {
        this.element.innerHTML = value;
    }
}

var descriptor = Object.getOwnPeropertyDescriptor(CustomHTMLElement.prototype, "html");
console.log("get" in descriptor); // true
console.log("set" in descriptor); // true
console.log(descriptor.enumerable); // false
```



- w/o class expression

```js
let CustomHTMLElement = (function() {
    "use strict";
    
    const CustomHTMLElement = function(element) {
        if (typeof new.target === "undefined") {
            throw new Error("Constructor must be called with new!");
        }
        this.element = element;
    }
    
    Object.defineProperty(CustomHTMLElement.prototype, "html", {
       enumerable: false,
       configurable: true,
        get: function() {
            return this.element.innerHTML;
        },
        set: function(value) {
            this.element.innerHTML = value;
        }
    });
    
    return CustomHTMLElement;
})();
```

- javascript object getter.. remind

```js
var obj = {
  log: ['a', 'b', 'c'],
  get latest() {
    if (this.log.length == 0) {
      return undefined;
    }
    return this.log[this.log.length - 1];
  }
}

console.log(obj.latest);
// expected output: "c"
```



- 이처럼 클래스를 사용하면
  - 코드 양이 적어진다.

## 9.6 계산된 멤버 이름

```js
let methodName = "sayName";

class PersonClass {
    constructor(name) {
        this.name = name;
    }
    
    [methodName]() {
        console.log(this.name);
    }
}

let me = new PersonClass("victor");
me.sayName();
```

-  let methodName = Symbol.for("sayName");
  - symbol은 Object에서 property 정의할 때 사용

```js
// 접근자 프로퍼티
let propertyName = "html";
class CustomHTMLElement {
    constructor(element) {
        this.element = element;
    }
    
    get [propertyName]() {
        return this.element.innerHTML;
    }
    
    set [propertyName](value) {
        this.element.innerHTML = value;
    }
}
```



## 9.7 제네레이터 메서드

- 클래스와 객체 리터럴의 유사점
  - 메서드
  - 접근자 프로퍼티
  - **제네레이터**
    - 8장에서 메서드 이름 앞에 *를 붙여 객체 리터럴에 제네레이터를 정의하는 방법을 배웠음
      - 같은 문법을 클래스에서도 사용 가능
        - 제네레이터를 만들 수 있음

```js
class MyClass {
    *createiterator() {
        yield 1;
        yield 2;
        yield 3;
    }
}

let instance = new MyClass();
let iterator = instance.createIterator();
```

- 클래스가 값의 컬렉션을 나타내는 경우, 클래스의 기본 이터레이터를 정의하는 것이 훨씬 유용하다.
  - 제네레이터 정의 시
    - [Symbol.iterator]를 사용

```js
class Collection {
    constructor() {
        this.items = [];
    }
    *[Symbol.iterator]() {
        yield *this.items.values(); // ??? why? *를 붙여야 하는가?
    }
}

var collection = new Collection();
collection.items.push(1);
collection.items.push(2);
collection.items.push(3);

for (let x of collection) {
    console.log(x);
}
```

- 제네레이터 메서드에 계산된 이름을 사용
  - 그 메서드는 this.items 배열의 values() 이터레이터에 동작을 위임한다.
    - 값의 컬렉션을 관리하는 모든 클래스는 기본 이터레이터를 포함해야 하는데
      - 일부 컬렉션에 특화된 연산자가 연산을 수행할 대상 컬렉션의 이터레이터를 필요로 하기 때문이다.
        - Collection의 모든 인스턴스는 for~of문이나 전개 연산자에 바로 사용될 수 있다.
- 객체 인스턴스에 메서드나 접근자 프로퍼티를 원할 때
  - 클래스 프로토타입에 추가하는 것이 유용하다.
    - 반면 클래스에 메서드나 접근자 프로퍼티를 원한다면
      - 정적 멤버(static member)를 사용할 필요가 있다.????

## 9.8 정적 멤버

```js
function PersonType(name) {
    this.name = name;
}

// 정적 메서드
PersonType.create = function(name) {
    return new PersonType(name);
};

// instance method
PersonType.prototype.sayName = function() {
    console.log(this.name);
};

var person = PersonType.create("victor");

```

```js
class PersonClass {
    // PersonType 생성자와 같음
    constructor() {
        this.name = name;
    }
    
    // PersonType.prototype.sayName과 같음
    sayName() {
        console.log(this.name);
    }
    
    // PersonType.create와 같음
    static create(name) {
        return new Personclass(name);''
    }
}

let person = PersonClass.create("victor");
```

- contractor 메서드에는 static 사용 불가



## 9.9 파생 클래스의 상속

- es6 이전
  - 사용자 정의 타입의 상속을 구현하기 위해서는
    - 비용이 큼
      - 아래와 같음

```js
function Rectangle(length, width) {
    this.length = length;
    this.width = width;
}

Rectangle.prototype.getArea = function() {
    return this.length * this.width;
}

function Square(length) {
    console.log(this);
    Rectangle.call(this, length, length);
}

Square.prototype = Object.create(Rectangle.prototype, {
    constructor: {
        value: Square,
        enumerable: true,
        writable: true,
        configurable: true
    } 
});

var square = new Square(3);

console.log(square.getArea()); // 9
console.log(square instanceof Square); // true
console.log(square instanceof Rectangle); // true
```



- es6 class

```js
class Rectangle {
    constructor(length, width) {
        this.length = length;
        this.width = width;
    }
    
    geArea() {
        return this.length * this.width;
    }
}

class Square extends Rectangle{
    constructor(length) {
        // Rectangle.call(this, length, length)와 같음
        super(length, length);
    }
}

var square = new Square(3);

console.log(square.getArea()); // 9 
console.log(square instanceof Square); // true
console.log(square instanceof Rectangle); // true
```

- 파생 클래스(derived classess): 다른 클래스를 상속한 클래스
- 파생 클래스에서 생성자를 명시 하려면 반드시
  - super()를 사용
- 클래스 선언에서 생성자를 사용하지 않는 경우
  - 새 인스턴스를 만들 때 전달된 모든 인자와 함께 super()가 자동으로 호출된다.

```js
class Square extends Rectangle {
    // 생성자 없음
}

class Square extends Rectangle {
    constructor(...args) {
        super(...args);
    }
}
```

- 위 두 클래스는 같다



##### super() 사용 시 유의할 점

```txt
- 파생 클래스에서만 super()를 사용할 수 있다.
- 만약 파생 클래스가 아닌 클래스(extends를 사용하지 않은 클래스)나 함수에서 사용하면 Error 발생
- 생성자 내의 this에 접근하기 전에 super()를 호출해야만 한다.
- super()는 this를 초기화하는 역할을 하기 때문에, super()를 호출하기 전에 this에 접근하려 하면 에러가 발생한다.
- super()를 호출하지 않는 유일한 방법은 클래스 생성자에서 객체를 반환하는 것이다.
```



### 9.9.1 클래스 메서드 대신하기

- 파생 클래스에서 메서드는 항상 기반 클래스의 같은 이름을 가진 메서드를 대신(Shadowing)한다.

```js
class Square extends Rectangle {
    constructor(length) {
        super(length, length);
    }
    
    // Rectangle.prototype.getArea()를 오버라이드하여 대신함
    getArea() {
        return this.length * this.length;
    }
}
```

or

```js
class Square extends Rectangle {
    constructor(length) {
        super(length, length);
    }
    
    // Rectangle.prototype.getArea()를 오버라이드하여 대신함
    getArea() {
        return super.getArea();
    }
}
```

- 4장 "super 참조를 통한 쉬운 프로토타입 접근" 참고

### 9.9.2  정적 멤버 상속

- 기반 클래스가 정적 멤버를 가지고 있으면, 그 정적 멤버는 파생 클래스에서도 사용될 수 있다.

```js
class Rectangle {
    constructor(length, width) {
        this.length = length;
        this.width = width;
    }
    
    getArea() {
        return this.length * this.width;
    }
    
    static create(length, width) {
        return new Rectangle(length, width);
    }
}

class Square extends Rectangle {
    constructor(length) {
        // Rectangle.call(this, length, length)와 같음
        super(length, length);
    }
}

var rect = Square.create(3, 4);

console.log(rect instanceof Rectangle); // true
console.log(rect.getArea()); // 12
console.log(rect instanceof Square); // false
```

### 9.9.3 표현식으로부터 파생된 클래스

- 가장 강력한 부분
  - 표현식으로부터 클래스를 파생하는 능력
  - **[[Construct]]와 프로토타입을 가지고 있는 함수의 어떤 표현식이든지 extends와 함께 사용 가능**

```js
function Rectangle(length, width) {
    this.length = length;
    this.width = width;
}

Rectangle.prototype.getArea = function() {
    return this.length * this.width;
}

class Square extends Rectangle {
    constructor(length) {
        super(length, length);
    }
}

var x = new Square(3);
console.log(x.getArea()); // 9
console.log(x instanceof Rectangle); // true
```

- 동적으로 상속받을 대상을 결정할 수도 있다.

```js
function Rectangle(length, width) {
    this.length = length;
    this.width = width;
}

Rectangle.prototype.getArea = function() {
    return this.length * this.width;
}

function getBase() {
    return Rectangle;
}

class Square extends getBase() {
    constructor(length) {
        super(length, length);
    }
}

var x = new Square(3);

console.log(x.getArea()); // 9
console.log(x instanceof Rectangle) // true
```

- 기반 클래스를 동적으로 결정할 수 있기 때문에 다른 형태의 상속 방식을 만들 수도 있다.
- __믹스인__ 을 효과적으로 만들 수 있다.

```js
// mixin
let SerializableMixin = {
    serialize() {
        return JSON.stringify(this);
    }  
};

let AreaMixin = {
    getArea() {
        return this.length * this.length;
    }
};

function mixin(...mixins) {
    var base = function() {};
    Object.assign(base.prototype, ...mixins);
    return base;
}

class Square exxtends mixin(AreaMixin, SerializableMixin) {
    constructor(length) {
        super();
        this.length = length;
        this.width = length;
    }
}

var x = new Square(3);

console.log(x.getArea()); // 9
console.log(x.serialize()); // "{"length":3,"width":3}"
```



- extends 뒤에 어떤 표현식이든 사용할 수 있지만
  - 모든 표현식이 유효 x
    - null or generator를 사용 시 
      - error 발생
        - why??
          - 클래스의 새 인스턴스를 만들려고 시도하면 호추되어야 할 [[Construct]]가 없기 때문에 에러가 발생할 것이다.

### 9.9.4 내장된 타입을 상속하기

- es5
  - 상속을 통해 특별히 정의한 배열 타입을 만들지 못함
    - why???
      - let see~~

```js
// 내장 배열의 동작
var colors = [];
colors[0] = "red";
console.log(colors.length); // 1;

colors.length = 0;
console.log(colors[0]); // undefined

// es5에서 배열의 상속 시도
function MyArray() {
    Array.apply(this, arguments);
}

MyArray.prototype = Object.create(Array.prototype, {
    constructor: {
        value: MyArray,
        writable: true,
        configurable: true,
        enumerable: true
    }
});

var colors = new MyArray();
colors[0] = "red";
console.log(colors.length); // 0

colors.length = 0;
console.log(colors[0]); // "red"

```

- whhhhhhyyyyy?
  - MyArray 인스턴스의 length와 숫자 프로퍼티는 내장 배열과 동일하게 동작하지 않는다~
    - **이러한 기능이 Array.apply()나 프로토타입을 할당한다고 처리되지 않아서~**

- es6 클래스의 목표
  - __내장 타입의 상속을 허용하는 것__



```txt
무었이 다른가???

es5:

this 값은 파생 타입(like MyArray)에 의해 먼저 만들어지고 나서(Array.apply() 메서드 같은) 기반 타입 생성자가 호출. 이는 this가 MyArray 인스턴스로 시작하고, Array의 추가 프로퍼티를 받는다는 의미이다.

es6:

this값이 기반 타입(Array)에 의해 만들어지고 나서 파생 클래스 생성자(MyArray)에 의해 수정된다. 결과적으로 this는 기반 타입의 내장 기능과 함께 시작하고, 그와 관련된 모든 기능을 올바르게 상속 받는다.
```

```js
class MyArray extends Array {
    
}

var colors = [];
colors[0] = "red";
console.log(colors.length); // 1;

colors.length = 0;
console.log(colors[0]); // undefined
```

- Array에 의해 this가 만들어지고 MyArray이가 이걸 수정 하는 방식



### 9.9.5 Symbol.species 프로퍼티

- 내장 타입 상속
  - 편리성
    - 내장 타입의 인스턴스를 반환하는 메서드가
      - 자동으로 내장 타입의 인스턴스를 반환하는 대신!!!!!
        - 파생 클래스의 인스턴스를 반환

```js
class MyArray extends Array {
    
}

let items = new MyArray(1,2,3,4,5);
let subitems = items.slice(1, 3);

console.log(items instanceof MyArray); // true
console.log(subitems instanceof MyArray); // true
```



- 그럼 How to choose what will return?
  - [Symbol.species]
    - 를 사용해 설정

```js
Array
ArrayBuffer
Map
Promise
RegExp
Set
타입 배열 (10장에서 다룸)
```

- 리스트의 각 타입은 this를 반환하는 기본 Symbol.species  프로퍼티를 가짐
- how to implement??

```js
class MyClass {
    static get [Symbol.species]() {
        return this;
    }
    
    constructor(value) {
        this.value = value;
    }
    
    clone() {
        return new this.constructor[Symbol.species](this.value);
    }
}
```

- Symbol.species
  - 클래스의 종류를 변경할 수 없기 때문에 getter만 존재한다.
  - this.constructor[Symbol.species]를 호출 시 
    - 언제나 MyClass 반환

```js
class MyClass {
    static get [Symbol.species]() {
        return this;
    }
    
    constructor(value) {
        this.value = value;
    }
    
    clone() {
        return new this.constructor[Symbol.species](this.value);
    }
}

class MyDerivedClass1 extends MyClass {

}

class MyDerivedClass2 extends MyClass {
    static get [Symbol.species]() {
        return MyClass;
    }
}

let instance1 = new MyDerivedClass1("foo");
let clone1 = instance1.clone();
let instance2 = new MyDerivedClass2("bar");
let clone2 = instance2.clone();

console.log(clone1 instanceof MyClass); // true
console.log(clone1 instanceof MyDerivedClass1); // true
console.log(clone2 instanceof MyClass); // true
console.log(clone2 instanceof MyDerivedClass2); // false
```

- MyDerivedClass 클론은 this 자기자신을 반환
- MyDerivedClass 클론은 MyClass를 반환하도록 오버라이드
  - 그래서 not the instanceof MyDerivedClass2

```js
class MyArray extends Array {
    static get [Symbol.species]() {
        return Array;
    }
}

let items = new MyArray(1,2,3,4);
let subitems = items.slice(1, 3);

console.log(items instanceof MyArray); // true MyArray의 인스턴스니까
console.log(subitems instanceof Array); // true MyArray가 Array의 파생 클래스니까
console.log(subitems instanceof MyArray); // false static get [Symbol.species]가 Array를 돌려주도록 재정의 되어 있어서 false
```

- 추가로 Symbol.species가 정의된 클래스로부터 파생된 클래스를 만들려 한다면,
  - 반드시 생성자 대신 Symbol.species를 사용해야 한다.

## 9.10

- 3장에서 배운 new.target 값을 통해 함수가 어떻게 호출 됐는지 살펴 봤다.
  - 클래스 생성자에서도 클래스가 호출되는 방식을 결정하기 위해 
    - new.target을 사용할 수 있다.

```js
class Rectangle {
    constructor(length, width) {
        console.log(new.target === Rectangle);
        this.length = length;
        this.width = width;
    }
}

// new.target은 Rectangle
var obj = new Rectangle(3, 4); // true

class Square extends Rectangle {
    constructor(length) {
        super(length, length);
    }
}

// new.target은 Square
obj = new Square(3); // false 출력
```

- square는 Rectangle의 생성자를 호출
  - new.target은 Square
    - 그래서 Rectangle이 아니어서 false



- **다음 예제처럼 new.target을 사용하여 추상기반 클래스를 만들 수 있다**

```js
// 추상 기반 클래스
class Shape {
    constructor() {
        if (new.target === Shape) {
            throw new Error("This class cannot be instantiated directly");
        }
    }
}

class Rectangle extends Shape {
    constructor(length, width) {
        super();
        this.length = length;
        this.width = width;
    }
}

var x = new Shape(); // error!
var y = new Rectangle(3, 4) //  정상 작동
console.log(y instanceof Shape) // true
```

- Shape 클래스를 직접 호출해 instantiation 하지 못한다.
  - abstract 클래스 같은 것
- **! 클래스는 new 없이 호출 불가능**
  - new.target 프로퍼티는 클래스 생성자 내에서 undefined가 될 수 없다.
  - 클래스가 함수처럼 호출 되는 것을 막는 역활도 한다.

## 9.11 요약







