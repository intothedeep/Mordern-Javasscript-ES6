# Understanding Asynchronous JavaScript — the Event Loop

### 자바스크립트의 특징
- 싱글 쓰레드
  - 그래서 메인 쓰레드를 블락하지 않고 어떤 작업을 할 수 없다.
    - 그래서 비동기(such as callbacks, promises, and async/await)사용
      - why? 메인 쓰레드를 blocking하지 않고 작업 진행하려고



### 자바스크립트 동기 동작 방식
```js
const second = () => {
  console.log("Hello there!");
}

const first = () => {
  console.log("Hi there!");
  second();
  console.log("The End!");
}

first();
```
- 위 코드가 자바스크립트 엔진 내에서 어떻게 실행되는지 이해하려면 실행 컨텍스트와 콜 스택(execution stack or call stack)!을 이해해야 한다.

##### Execution Context
- Execution Context는 자바스크립트 코드가 평가되고 실행되는 어떤 환경에 대한 추상적인 개념이다. 항상 자바스크립트에서 어떤 코드가 동작할 때 코드는 Execution Context 내에서 동작한다.

- Function code는 function execution context에서 동장하고 전역 코드는 global executino context에서 실행된다. 각각의 코드는 각각의 실행 환경을 가진다.

##### Call Stack
- 콜 스택에는 코드가 실행되는 동안 생성되는 모든 실행 context 저장되는데 이름 자체가 의미하는 것처럼 LIFO 방식의 구조를 가지고 있다.
- 자바스크립트는 single-threaded programming 언어이기 때문에 하나의 콜 스택만 존재한다.
- 콜 스택은 LIFO 방식으로 동작하기에 스택에 적재된 작업들은 스택 맨위에 저장되거나 맨 위에서만 제거될 수 있다.

```js
const second = () => {
  console.log("Hello there!");
}

const first = () => {
  console.log("Hi there!");
  second();
  console.log("The End!");
}

first();
```

콜 스택에서 위 코드가 실행되는 순서는 아래와 같다.

main(),
main(), first()
main(), first(), console.log("Hi there!")
main(), first()
main(), first(), second()
main(), first(), second(), console.log("Hello there!");
main(), first(), second()
main(), first()
main(), first(), console.log("The End!");
main(), first()
main()


### How Does Asynchronous JavaScript Work?

##### What is Blocking?
- **동기방식으로** 이미지 작업을 하거나 네트워크 요청을 보낸다고 가정해보자.

```js
const processImage = (image) => {
  /**
  * doing some operations on image
  **/
  console.log('Image processed');
}
const networkRequest = (url) => {
  /**
  * requesting network resource
  **/
  return someData;
}
const greeting = () => {
  console.log('Hello World');
}
processImage(logo.jpg);
networkRequest('www.somerandomurl.com');
greeting();
```
- 이미지 처리와 네트워크 요청은 시간이 걸린다.
  - 이미지 크기에 따라 processImage()가 처리되는 시간이 결정되고, 
    - 이미지 프로세스가 완료되면 콜 스택에서 제거된다.
- 그 다음에 networkRequest() 호출되어 콜 스택에 쌓이고
  - 이미지 처리과정과 같은 과정을 거치고 콜 스택에서 제거된다.
- 마지막으로 greeting()이 처리된다.

- 위 처럼 한 과정이 처리가 끝날 때 까지 다음 작업은 기다려야 한다.
  - 이걸 다른 말로
    - 함수들이 콜 스택 또는 메인 쓰레드를 Block한다라고 한다.
    
- ** THIS is not IDEAL**
 
### So what's the solution?
- The simplest solution is
  - __asynchronous callbacks!!__

```js

```





