
# 3장 Function


## 3.1 함수의 매개변수 기본값
자바 스크립트의 함수는 함수 정의에 선언된 매개변수의 갯수에 상관없이 몇 개의 매개변수든 전달할 수 있도록 허용하는 특징이 있다.

#### 3.1.1 ECMAScript 5의 매개변수 기본값

```js
function makeRequest(url, timeout, callback) {
  timeout = timeout || 2000;
  callback = callback || function () {}
  
  // 나머지
  
}
```
