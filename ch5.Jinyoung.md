# 5장 구조분해를 이용한 보다 쉬운 데이터 접근
## 5.1 구조분해는 왜 유용한가
 ES5 까지는 객체와 배열로 부터 값을 가져와 저장할 때 다음과 같은 방법을 썼다.

```javascript
let options = {
  repeat: true,
  save: false
};

// 객체로부터 데이터 추출
let repeat = options.repeat,
    save = options.save;
```

이러한 코드는 할당할 변수가 많다면 모든 변수에 일일이 할당해야 하고, 중첩된 데이터 구조일 때 하나의 데이터를 찾기 위해 전체 구조를 탐색해야하는 불편함이 있다.

## 5.2 객체 구조분해
```javascript
let node = {
  type: "Identifier", 
  name: "foo"
 };

let { type, name } = node;

console.log(type);  // "Identifier"
console.log(name);  // "foo"
```

type과 name은 지역 변수이면서 동시에 node 객체의 값을 읽기 위한 프로퍼티로 선언되었다.

_주의: initializer 잊지 말기_

### 5.2.1 구조분해 할당

```javascript
let node = {
    type: "Identifier",
    name: "foo"
  },
  type = "Literal",
  name = 5;

// 구조분해로 다른 값을 할당
({ type, name } = node);

function outputInfo(value) {
  console.log(value === node);
}
outputInfo({ type, name } = node);

console.log(type);  // "Identifier"
console.log(name);  // "foo"
```

_주의: () 로 감싸기 {} 가 블록문이 아님을 명시_

구조분해 할당 표현식은 표현식의 오른쪽 (=뒤) 에 있는 값으로 평가되기 때문에 값이 사용되는 어디든 구조분해 할당 표현식을 쓸 수 있다.

### 5.2.2 기본값

* 객체에 없는 프로퍼티를 구조분해 할당하면  undefined값이 할당 됌.
* 프로퍼티 이름 뒤에 (=) 를 추가하면 기본값 명시 가능.

```javascript
let node = {
    type: "Identifier",
    name: "foo"
  };

let { type, name, value = true } = node;

console.log(type);  // "Identifier"
console.log(name);  // "foo"
console.log(value); // true
```

### 5.2.3 이름이 다른 지역 변수에 할당하기
* : 를 사용하여 객체 프로퍼티 이름과 다르게 설정 가능.
* : 뒤에 = 붙여서 초기화까지 가능.

```javascript
let node = {
      type: "Identifier",
    };

let { type: localType, name: localName = "bar" } = node;

console.log(localType);  // "Identifier"
console.log(localName);  // "bar"
```

### 5.2.4 중첩된 객체 구조분해

```javascript
let node = {
      type: "Identifier",
      name: "foo",
      loc: {
        start: { line: 1, column: 1 },
        end: {line:1, column: 4}
      }
    };

let { loc: { start} } = node;
console.log(start.line); // 1
console.log(start.column); // 1
```


## 5.3 배열 구조분해
배열 구조분해는 객체 구조분해와 매우 유사하지만 프로퍼티 기반이 아니라 배열의 위치를 기반으로 함.

```javascript
let colors = ["red", "green", "blue"];
let [ firstColor, secondColor ] = colors;

console.log(firstColor); // "red"
console.log(secondColor); // "green"
```

* 변수 이름은 상관없음.
* 명시적으로 언급되지 않은 요소는 무시. 
* 배열은 변경되지 않음.
* 필요한 요소만 변수를 제공할 수 있음
```javascript
let colors = ["red", "green", "blue"];
let [ , , thirdColor ] = colors;

console.log(thirdColor); // "blue"
```

### 5.3.1 구조분해 할당
객체 구조분해와는 다르게 괄호로 표현식을 감쌀 필요가 없음.
```javascript
let colors = ["red", "green", "blue"],
    firstColor = "black",
    secondColor = "purple";

[firstColor, secondColor] = colors;

console.log(firstColor); // "red"
console.log(secondColor); // "green"
```

변수의 값을 스위칭 할 때 사용할 수 있다.
```javascript
let a = 1,
    b = 2;

[a, b] = [b, a];
console.log(a, b); // 2, 1
```

### 5.3.2 기본값
(=) 를 사용해서 기본값 명시 가능.
```javascript
let colors = [ "red" ];
let [firstColors, secondColors = "green"] = colors;
console.log(firstColor); // "red"
console.log(secondColor); // "green"
```

### 5.3.3 중첩된 배열 구조분해
전체 패턴에 또 다른 배열 패턴을 넣는 방식으로 중첩된 배열을 탐색할 수 있음.
```javascript
let colors = ["red", ["green", "lightgreen"], "blue"];
let [firstColor, [secondColor]] = colors;
console.log(firstColor); // "red"
console.log(secondColor); // "green"
```

### 5.3.4 나머지 요소
```javascript
let colors = ["red", "green", "blue"];
let [firstColor, ...restColors] = colors;
console.log(firstColor); // "red"
console.log(restColors); // ["green", "blue"]
```

* 특정 요소를 추출하고 나머지를 계속 사용할 수 있게끔 유지하는데 유용.
* 배열을 복제할 때 유용. _( ES5 에서는 .concat() 을 주로 사용했다고 함.)_
```javascript
let colors = ["red", "green", "blue"];
let [...clonedColors] = colors;

console.log(clonedColors); // ["red", "green", "blue"]
```

## 5.4 혼합된 구조분해
객체 구조분해와 배열 구조분해를 함께 사용해 더 복잡한 표현식을 만들 수 있다.  객체와 배열이 섞여있는 구조일 때 잘.. 알아서 사용할 것 ㅎㅎ

## 5.5 구조분해된 매개변수
함수가 optional한 매개변수를 가질 때 유용.

* 구조분해를 사용하지 않을 때
```javascript
function setCookie(name, value, options) {
    options = options || {};
    
    let secure = options.secure,
        path = options.path,
        domain = options.domain,
        expires = options.expires;
    
    ...
}

setCookie("type", "js", {
  secure: true,
  expires: 600
});
```

함수 정의를 통해서 어떤값이 입력될지 예상할 수 없으므로 함수 본문을 살펴봐야 함.

* 구조분해 사용 버전
```javascript
function setCookie(name, value, {secure, path, domain, expires}) {
	...
}

setCookie("type", "js", {
  secure: true,
  expires: 600
});
```

세번째 매개변수를 전달하지 않으면 에러 발생.
`setCookie("type", "js"); //에러 발생!`

* 기본값 사용해서 수정
```javascript
function setCookie(name, value, {secure, path, domain, expires} = {}) {
	...
}
```

* 구조분해 된 매개변수에 기본값 설정
```javascript
function setCookie(name, value, 
{	
  secure = false, 
  path = "/", 
  domain = "example.com",
  expires = new Date(Date.now() + 360000000)} = {}) {
  ...
}
```


#recobell #javascript/es6
