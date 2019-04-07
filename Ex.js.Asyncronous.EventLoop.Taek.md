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

```txt
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
```



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
- event
- callback
- setTimeout
- Promise



- 가장 간단한 방법은
  - 비동기 callback을 사용하면 된다.

```js
const networkRequest = () => {
  setTimeout(() => {
    console.log('Async Code');
  }, 2000);
};
console.log('Hello World');
networkRequest();
```

- setTimeout 메서드는 자바스크립트 엔진이 아니고
  - web APIs(in browsers)
    - C/C++ APIs(in node.js)

- 이 코드가 어떻게 실행되는지 이해하려면
  - 이벤트 루프나 콜백 큐에 대한 개념을 알아야 한다 (task queue or message queue)



- the event loop, web APIs, message queue/task queue
  - 는 자바스크립트 엔진이 아니고
    - 브라우저의 자바스크립트 실행환경 이거나
      - Node.js의 자바스크립트 실행환경이다



위 코드가 어떻게 실행되는지 알아보자

```js
const networkRequest = () => {
  setTimeout(() => {
    console.log('Async Code');
  }, 2000);
};
console.log('Hello World');
networkRequest();
console.log('The End');
```

- console.log("Hello World");
  - in
  - out
- networkRequest()
  - in
    - setTimeout()
      - in
        - callback
        - time in milliseconds
          - starts a timer of s2 in the web APIs environment
      - out
  - out
- console.log("The End");
  - in
  - out
- after 2s
  - the callback is pushed to the message queue
    - 그러나 콜백은 바로 실행되지 않고
      - event loop를 통해 동작



##### The Event Loop

- 이벤트 루프가 하는 일은

  - 콜 스택을 들여다 보면서
    - 콜 스택이 비어있는지 아닌지 확인한다.

- 만약 콜 스택이 비어 있으면

  - 메세지 큐에 대기하고 있는 callback이 있는지 확인해야 한다.

- 위 코드 같은 경우

  - 메세지 큐는
    - 하나의 callback(console.log("Async code!"))를 가지고 있고
    - call stack은 비어있다.
      - 그러면 콜 스택에 콜백을
        - in
        - out

- 콜백이 완전히 비워졌다~

- 프로그램 끝!





##### DOM Events

- 메세지 큐는 
  - 클릭이나 키보드 이벤트와 같은 DOM events가 가진 콜백도 저장한다.

```js
document.querySelector('.btn').addEventListener('click', (event) => {
   console.log("Button Clicked") 
});
```

- DOM 이벤트 같은 경우에 이벤트 리스너는 web APIs environment에서 특정 이벤트를 기다린다ㅏ.
  - 그리고 그 이벤트가 벌어지면
    - callback 함수가 메세지 큐에 등록된다.
- 이 때
  - Event Loop가 call stack이 비어있는지 확인하고
    - 해당 이벤트로 인해 발생한 callback을 call stack에 등록한다.



### ES6 Job Queue/Micro-Task queue

- job queue/micro-task queue는 자바스크립트 Promises에서 사용된다.
- 잡 큐와 메세지 큐의 차이점
  - 잡 큐는 메세지 큐보다 빨리 실행된다.(higher priority)
- 따라서
  - 잡 큐와 마이크로 테스크 큐에 들어 있는 
    - Promised jobs들은 메세지 큐에 들어 있는 콜백들보다 먼저 실행된다.



```js
console.log("script start");

setTimeout(() => {
    console.log("setTimeout");
}, 0);

new Promise((resolve, reject) => {
    resolve("Promise resolved@!");
}).then(res => {
   console.log(res); 
}).catch(err => console.log(err));

console.log("Script End");
```



output:

```js
Script start
Script End
Promise resolved
setTimeout
```

- 이처럼 promise가 setTimeout에 들어있는 callback보다 먼저 실행되는 것을 확인할 수 있다.



For example:

```js
console.log('Script start');
setTimeout(() => {
  console.log('setTimeout 1');
}, 0);
setTimeout(() => {
  console.log('setTimeout 2');
}, 0);
new Promise((resolve, reject) => {
    resolve('Promise 1 resolved');
  }).then(res => console.log(res))
    .catch(err => console.log(err));
new Promise((resolve, reject) => {
    resolve('Promise 2 resolved');
  }).then(res => console.log(res))
    .catch(err => console.log(err));
console.log('Script End');
```



This prints:

```js
Script start
Script End
Promise 1 resolved
Promise 2 resolved
setTimeout 1
setTimeout 2
```

- message queue에 들어있는 callback보다 micro-task queue에 들어있는 resolve가 먼저 실행된다.



예:

```js
console.log('Script start'); //1
setTimeout(() => {
  console.log('setTimeout'); // 6
}, 0);
new Promise((resolve, reject) => {
    resolve('Promise 1 resolved');
  }).then(res => console.log(res)); // 3
new Promise((resolve, reject) => {
  resolve('Promise 2 resolved');
  }).then(res => {
       console.log(res); // 4
       return new Promise((resolve, reject) => {
         resolve('Promise 3 resolved');
       })
     }).then(res => console.log(res)); // 5
console.log('Script End'); // 2
```

출력:

```js
Script start
Script End
Promise 1 resolved
Promise 2 resolved
Promise 3 resolved
setTimeout
```

