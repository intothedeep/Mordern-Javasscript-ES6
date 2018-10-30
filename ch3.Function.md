
# 3장 Function


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
***주의사항**
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
₩₩₩
