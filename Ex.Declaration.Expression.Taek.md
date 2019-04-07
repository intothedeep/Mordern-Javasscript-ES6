##### 표현식 선언식
```js
// 표현식: 함수 반환해서 test에 저장
var test = function() {
 };

// 표현식: 함수 반환만 test2 함수는 생성이 안됨
(function test2() {

})

// 아래 두개는 동일하게 표현식을 반환하고 즉시실행 된다.
(function test3() {

})();
// ( .. ) 없이 함수만 선언하면 에러가 나지만 즉시실행 표현식으로 만들면
// 즉시실행이 우선적으로 적용되어 함수를 반환하고 실행시키는 듯
(function test4() {

}());

// 아래는 undefined 반환
(function test5() {

})();
// 위와는 다르게 아래는 boolean 값 반환
!function test6() {

}();

// 가정 하나
var singleton = function(){

};
// 위 표현식은 아래 표현식의 단축형 같다.
var singleton = (function() {

});

function test7() {

}(); // Uncaught SyntaxError: Unexpected token (
// why?? 이 함수는 선언식인데 호출 연산자 ()는 표현식에서만 사용가능하다.... 선언과 동시에 호출 불가능!

```
