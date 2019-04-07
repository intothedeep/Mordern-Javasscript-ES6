# 13장 모듈로 캡슐화하기

## 13.1 모듈이란?

- 자동으로 strict 모드에서 실행
- 모듈의 최상위 수준에서 만들어진 변수는 모듈 최상의 스코프 내에만 존재
- 최상위 수준에서 모듈의 this는 undefined
- HTML 스타일 주석을 허용하지 않음

## 13.2 익스포트 기본
```javascript
    // 데이터 익스포트
    export var color = "red";
    export let name = "Nicholas";
    export const magicNumber = 7;
    
    // 함수 익스포트 
    export function sum(num1, num2) {
    	return num1 + num2;
    }
    
    // 클래스 익스포트
    export class Rectangle {
    	constructor(length, width) {
    		...
    	}
    }
    
    // 이 함수는 모듈에 비공개
    function subtract(num1, num2) {
    	...
    }
    
    //함수 정의
    function multiply(num1, num2) {
    	...
    }
    
    // 위에서 정의한 함수를 익스포트
    export {multiply};
```
## 13.3 임포트 기본
```javascript
    import {identifier1, identifier2} from "./example.js";
    import * as example from "./example.js";
```
- 바인딩은 const를 사용한것 처럼 동작.
    - 같은 이름의 또 다른 변수를 정의할 수 없고, import 문 앞에서 그 식별자를 사용할수 없고 값 변경도 불가능.
```javascript
    import { sum } from "./example.js";
    import { multiply } from "./example.js";
    import { magicNumber } from "./example.js";
```
- 모듈은 import문에서 모듈을 사용한 회수와 상관없이 한 번만 실행 된다.

**13.3.4 임포트 바인딩의 특이한 점**

단순히 일반 변수처럼 원본의 바인딩을 참조하지 않고 변수, 함수 및 클래스에 대한 읽기 전용 바인딩을 만든다. 바인딩을 임포트한 모듈은 바인딩의 값을 변경할 수 없지만, 식별자를 익스포트한 모듈은 가능하다.
```javascript
    export var name = "Nicholas";
    export function setName(newName) {
    	name = newName;
    }
```
```javascript
    import { name, setName } from "./example.js";
    
    console.log(name);   // Nicholas
    setName("Greg");
    console.log(name);   // Greg
    
    name = "Nicholas";  //에러 발생!
```
name이 익스포트된 name 식별자를 위한 지역 스코프 내의 이름이기 때문에 이런 동작을 함.

## 13.4 익스포트와 임포트에 새로운 이름 사용
```javascript
    function sum(num1, num2) {}
    export {sum as add};
    
    //import 할 때
    import { add } from "./example.js";

    import {add as sum} from "./example.js";
    console.log(typeof add);  // "undefined"
    console.log(sum(1,2));
```
## 13.5 모듈의 기본 값
```javascript
    1.
    export default function(num1, num2) {}
    
    2.
    function sum(num1, num2) {}
    export default sum;
    
    3.
    function sum(num1, num2) {}
    export {sum as default};
```
```javascript
    import sum from "./example.js";
    import sum, {color} from "./example.js";
    import {default as sum, color} from "./example.js";
```
## 13.6 바인딩을 다시 익스포트하기
```javascript
    1.
    import {sum} from "./example.js";
    export {sum}
    2.
    export {sum} from "./example.js";
    3.
    export { sum as add } from "./example.js";
    4.
    export * from "./example.js";
```
## 13.7 바인딩 없이 임포트하기

어떤 모듈은 아무것도 익스포트하지 않는 대신, 전역 스코프 객체만 수정한다.

모듈 내의 최 사우이 수준 변수와 함수, 클래스는 자동으로 전역 스코프가 되지 않는데, 이것이 모듈내에서 전역 스코프에 접근할 수 없다는 의미는 아니다.
```javascript
    // 익스포트나 임포트 없는 모듈 코드
    Array.prototype.pushAll = function(items) {
    	//items는 배열이어야 함.
    	if(!Array.isArray(items)) {
    		throw new TypeError("Argument must be an array");
    	}
    
    	// 내장 push()와 전개 연산자를 사용
    	return this.push(...items);
    }
```
```javascript
    import "./example.js";
    
    let colors=["red", "green", "blue"];
    let itmes = [];
    
    items.pushAll(colors);
```
## 13.8 모듈 로드하기

**13.8.1 웹브라우저에서 모듈 사용하기**

웹 브라우저에서 스크립트를 로드하는 옵션

- <script> 엘리먼트의 src 속성에 로드할 코드의 위치를 명시
- src 속성없이 <script>엘리먼트에 인라인으로 js 코드를 내장
- 워커(웹 워커나 서비스 워커)를 실행하여 js 코드 파일을 로딩

**<script>로 모듈 사용하기**
```html
    <!-- 자바스크립트 파일 모듈 로딩-->
    <script type="module" src="module.js"></script>
    
    <!-- 인라인 모듈 포함 -->
    <script type="module">
    	import {sum} from "./example.js";
    	let result = sum(1,2);
    </script>
```
**웹브라우저에서 모듈이 로드되는 순서**

모듈에서는 스크립트와 달리, 올바른 실행을 보장하기 위해 먼저 로딩되어야 하는 파일을 지정하려는 목적으로 import 를 사용할 수 있다.

이를 위해서 <script type="module">은 항상 defer속성이 적용된 것처럼 동작한다. 

→ HTML파서가 <script type="module"> 내의 src속성을 만나자마자 다운로드되기 시작하지만, 문서가 완전히 파싱될 때까지 실행되지 않는다.

→ HTML 파일 내에  <script type="module"> 엘리먼트가 위치한 순서대로 실행된다.
```html
    <!-- 먼저 실행됨-->
    <script type="module" src="module1.js"></script>
    
    <!-- 두 번째로 실행됨-->
    <script type="module">
    	import {sum} from "./example.js";
    	let result = sum(1,2);
    </script>
    
    <!-- 세 번째로 실행됨-->
    <script type="module" src="module2.js"></script>
```
- module1.js를 다운로드하고 파싱
- module1.js에 있는 import 리소스를 재귀적으로 다운로드하고 파싱
- 인라인 모듈을 파싱
- 인라인 모듈 안에 있는 import 리소스를 재귀적으로 다운로드하고 파싱
- module2.js를 다운로드하고 파싱
- module2.js에 있는 import 리소스를 재귀적으로 다운로드하고 파싱

로드가 완료되면, 문서가 완전히 파싱될 때까지 아무것도 실행되지 않는다. 문서가 완전히 파싱된 후에, 다음과 같이 동작한다.

- module1.js의 import 리소스를 재귀적으로 실행
- module1.js 실행
- 인라인 모듈의 import 리소스를 재귀적으로 실행
- 인라인 모듈 실행
- module2.js의 import 리소스를 재귀적으로 실행
- module2.js 실행

**웹 브라우저에서 비동기 모듈 로드하기**
```html
    <script async src="~~~">
```
 파일 다운로드와 파싱이 완료되자마자 스크립트가 바로 실행된다.
```html
    <script async src="./src1.js">
    <script async src="./src2.js">
```
각각 파일 다운로드와 파싱이 완료되면 스크립트가 실행되지만 꼭 순서대로 실행을 한다는 보장은 없다.
```html
    <!-- 모듈에도 적용가능, 어떤 것이 먼저 실행될지는 알 수 없음 -->
    <script type="module" async src="module1.js"></script>
    <script type="module" async src="module2.js"></script>
```
두 모듈 파일은 비동기로 로딩된다.  하지만 뭐가 먼저 실행될지는 알 수 없다. module1 다운로드가 먼저 완료되면 먼저 실행되고, module2도 마찬가지이다.

[참고 script의 async와 defer속성]

[script의 async와 defer 속성](https://blog.asamaru.net/2017/05/04/script-async-defer/)

**워커에서 모듈 로드하기**
```javascript
    //스크립트처럼 script.js 를 로딩
    let worker = new Worker("script.js");
    
    //모듈처럼 module.js를 로딩
    let worker = new Worker("module.js', {type: "module"});
```
워커 모듈이 워커 스크립트와 다른 점

- 워커 스크립트는 웹 페이지와 같은 출처에서만 로딩될 수 있지만, 워커 모듈은 그런 제한이 없다. (적절한 CORS헤더를 가진 파일은 로드할 수 있다)
- 워커 스크립트에서는 워커에 추가적인 스크립트를 로딩하기 위해 self.importScripts() 메서드를 사용할 수 있지만, 워커 모듈에서는 그 대신 import를 사용해야만 하기 때문에  self.importScripts() 가 항상 실패한다.

**브라우저 모듈 명시자 결의안**

- 루트 디렉토리를 의미하는 / 로 시작
- 현재 디렉토리를 의미하는 ./ 로 시작
- 부모 디렉토리를 의미하는 ../ 로 시작
- URL 형식
