# 4 장 확장된 객체 기능

## 4.1 객체 카테고리

- 객체 동작 방식 관점

  - 일반 객체 (ordinary objects)
    exotic object 가 아닌 모든 javascript object.

  - 이형 객체 (exotic objects)
    자바스크립트 객체의 기본과 다른 내부 동작을 가짐. 예를 들어 Array 의 instance 는 length property 에 따라 객체 내부가 변경되는데 자바스크립트의 일반 객체에서 가지지 않는 Array instance 고유의 동작임.

- 자바스크립트 실행 환경 관점

  - 표준 객체 (standard objects)
    실행 환경(browser or node.js)에 관계없이 ECMAScript 언어 스펙에 명시된 표준 객체.

  - 내장 객체 (built-in objects)
    실행 환경에 존재하는 모든 내장 객체로 표준 객체(standard objects)를 포함하며 실행환경에 따라 포함되는 내장 객체가 달라짐.

    ex)

    - node: global
    - browser: window, dom



## 4.2 객체 리러털의 문법 확장

- 프로퍼티 초기자 축약 (property initializer)
- 간결한 메서드 (concise methods)
- 계산된 property 이름 (computed property name)



## 4.3 새로운 메서드

- Obejct.is
  +0 과 -0 비교, NaN 과 NaN 비교를 제외한 연산 결과가 ===와 동일함.

- Object.assign
  ES5 에서 객체를 합성하는 mixin 패턴이 Object 의 메서드로 추가됨.

  ```javascript
  // es5 mixin 함수
  function mixin(receiver, supplier) {
    Object.keys(supplier).forEach(function(key) {
      receiver[key] = supplier[key];
    });

    return receiver;
  }

  var obj = { a: 1, b: 2 };
  var clone = mixin({}, obj);

  // es6
  Object.assign({}, obj);
  ```

  주의: getter 접근자 프로퍼티는 데이터 프로퍼티로 할당되고 setter 접근자 프로퍼티는 사라짐.

  ```javascript
  // 접근자 프로퍼티 설정 방법
  const person = {
    _name: "",
    get name() {
      console.log("get name~");
      return `${this._name} lee`;
    },
    set name(value) {
      console.log("set name~");
      this._name = value;
    }
  };

  person.name = "dh";
  console.log(person.name);
  console.log(Object.assign({}, person));
  ```

## 4.4 객체 리터럴 프로퍼티의 중복

- ES5 strict-mode: 에러
- ES6: 마지막에 선언된 프로퍼티로 덮어 씀.



## 4.5 열거 순서

- ES5: 열거 순서 보장되지 않음.
- ES6: 객체의 프로퍼티를 열거할 때 반환하는 순서를 엄격하게 정의.
- for-in loop 와 Object.keys, JSON.stringify 등의 메서드는 자바스크립트 엔진에 따라 열거 순서 보장 안될 수 있음.



## 4.6 프로토타입 개선

- 프로토타입 변경하기

  `Object.setPrototypeOf`

  - 프로토타입 변경하는 비표준 방법 (\_\_proto\_\_)

    ```
    const person = {
        getGreeting() {
            return 'hello';
        }
    };
    
    const dog = {
        getGreeting() {
            return 'woof';
        }
    };
    
    const friend = Object.create(person);
    console.log(friend.getGreeting());
    
    Object.setPrototypeOf(friend, dog);
    console.log(friend.getGreeting());
    
    friend.__proto__ = person;
    console.log(friend.getGreeting());
    ```

    거의 모든 브라우저에 구현되어있지만 ECMAScript 스펙에 정의되어 있지 않은 비표준 방법. setPrototypeOf 메서드가 공식적인 지원을 하기 전에 프로토타입을 변경하는 방법. 현재는 사용하지 않는 것을 권장.

- Super 참조를 통한 쉬운 프로토타입 접근

  super 는 현재 객체의 프로토타입을 가리키는 포인터이며, 사실상 Object.getPrototypeOf(this)의 값. 일반 함수가 아닌 메서드에서만 사용 가능하다.

  **프로토타입의 메서드 호출시 재귀호출 문제**

  ```javascript
  const person = {
    getGreeting() {
      return "hello";
    }
  };
  
  const dog = {
    getGreeting() {
      return "woof";
    }
  };
  
  const friend = {
    getGreeting() {
      return super.getGreeting() + "hi";
      //return Object.getPrototypeOf(this).getGreeting.call(this) + 'hi';
    }
  };
  
  Object.setPrototypeOf(friend, person);
  const relative = Object.create(friend);
  
  console.log(person.getGreeting());
  console.log(friend.getGreeting());
  console.log(relative.getGreeting());
  ```



## 4.7 공식적인 메서드 지원

ES5 까지 메서드는 데이터 대신 함수를 가진 객체 프로퍼티일 뿐이지만, ES6 부터 메서드가 속한 객체를 내부 [[HomeObject]] 프로퍼티로 가진 함수가 메서드라고 공식적으로 정의.

```javascript
const obj = {
  method() {
    return super.hi();
  }
  // funcProperty: function () { return super.hi(); } // 함수 안에 super키워드 사용하면 에러
};

const superObj = {
  hi() {
    return "hi";
  }
};

Object.setPrototypeOf(obj, superObj).method();
```
