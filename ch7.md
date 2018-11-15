# 7장 Set과 Map
* Set: 중복되지 않는 값의 리스트, 개별 요소에 접근x,  값이 존재하는지 알기 위해 검사.
* Map: 특정 값에 연결된 키의 컬렉션. key-value 쌍, 중복x. 저장해 놓은 데이터를 추후 빠르게 얻기 위한 캐시로서 빈번하게 사용.

## 7.1 ES5에서의 Set과 Map
```javascript
var set = Object.create(null); //객체에 상속된 프로퍼티 없음을 보장
set.foo = true;
//존재하는지 검사
if(set.foo) {
	//코드 실행
}

var map = Object.create(null);
map.foo = "bar";
// 값을 얻음
var value = map.foo;
console.log(value) // "bar"
```

## 7.2 대안의 문제점
문제1. 객체 프로퍼티는 문자열이어야 하기 때문에 같은 문자열로 평가되는 두 개의 키를 사용할 수 없다.
```javascript
var map1 = Object.create(null);
map[5] = "foo";
console.log(map["5"]); // "foo"

var map2 = Object.create(null),
	  key1 = {},
	  key2 = {};
map[key1] = "foo";
console.log(map[key2]);   // "foo"
```

문제2. falsy 한 키를 가진 Map 
```javascript
var map = Object.create(null);
map.count = 1;

//"count"가 존재하거나 0이 아닌 값인지 검사하는 것
if(map.count) {
	// 코드 실행
}
```


## 7.3 ES6의 Set
Set 
* 중복은 없고 순서는 있는 값의 리스트
* key 값을 문자열로 변경하지 않음
* 같은 값을 전달하여 .add() 하면 무시
* 배열을 사용해 초기화

```javascript
let set = new Set();
set.add(5);
set.add("5");
set.add(5); //중복 - 무시됨

console.log(set.size);  // 2

let key1 = {}, key2 = {};
set.add(key1);
set.add(key2); 

console.log(set.size);  // 4

let set2 = new Set([1,2,3,4,5,5,5,5]);
console.log(set2.size);     // 5
```

* has() 메서드로 값이 있는 지 확인
* delete() 메서드로 하나의 값 제거, clear() 메서드로 모든 값 제거.

```javascript
console.log(set.has(5));  // true
console.log(set.has(6)); // false
set.delete(5);
set.clear();
```

### Set.forEach()
다음 세개의 인자를 받는 콜백 함수 전달
	* Set에서 다음 위치의 값
	* 첫 번째 인자와 같은 값
	* 값을 읽어들인 Set
첫 번째와 두 번째 인자의 값이 동일 -> 다른 forEach 콜백 함수와 컨벤션을 유지하기 위해
```javascript
let set = new Set([1,2]);

set.forEach(function(value, key, ownerSet) {
	console.log(key + " " + value);
	console.log(ownerSet === set);
});
```

콜백 함수에서 this를 사용할 필요가 있다면 forEach 두 번째 인자로 this 를 보내면 된다.

### Set을 배열로 변환
전개 연산자(…) 로 Set을 배열을 서로 변환할 수 있다.
```javascript
let set = new Set([1,2,3,3,3,4,5]),
	  array = [...set];

console.log(array);  //[1,2,3,4,5]
```

중복을 없앨 때 유용
```javascript
function eliminateDuplicates(itmes) {
	return [...new Set(items)];
}

let numbers = [1,2,3,3,3,4,5],
	  noDuplicates = eliminateDuplicates(numbers);

console.log(noDuplicates);   //[1,2,3,4,5]
```

### Weak Set
Set은 객체 참조를 저장하는 방식, Set  인스턴스에 대한 참조가 존재하는 한, 객체는 가비지 컬렉션 될 수 없다.
```javascript
let set = new Set(),
	  key = {};

set.add(key);
console.log(set.size);   // 1

//원본 참조는 제거
key = null;
console.log(set.size);  //1

//원본 참조를 다시 가져옴
key = [...set][0];
```

WeakSet 은 add(), has(), delete() 메서드를 가지고 key로 객체 값만 받을 수 있다. 
```javascript
let set = new WeakSet(),
    key = {};

// Set에 객체 추가
set.add(key);
console.log(set.has(key));

set.delete(key);
console.log(set.has(key));
```

Set 타입과의 차이
```javascript
let set = new WeakSet(),
    key = {};

// Set에 객체 추가
set.add(key);
console.log(set.has(key));   //true

// key의 마지막 강한 참조를 제거 (또한 Weak Set으로부터 제거됨)
key = null;
```

특징
* WeakSet 인스턴스에서, 객체가 아닌 값이 전달되면 add() 메서드는 에러를 발생 시킨다.
* Weak Set 은 이터러블이 아니므로 for-of 반복문에 사용될 수 없다.
* Weak Set은 어떤 이터레이터도 노출하지 않으므로 Weak Set의 내용을 프로그램적으로 확인할 방법이 없다.
* Weak Set 에는 forEach() 메서드가 없다.
* Weak Set 에는 size 프로퍼티가 없다.

## 7.3 ES6의 Map
Map
* 순서가 있는 키와 값 쌍의 리스트. 키와 값은 어떤 타입이든 될 수 있다.
* has(key), delete(key), clear() 메서드가 있다.
* size 프로퍼티가 있다.
```javascript
let map = new Map();
map.set("title", "Understanding ES6");
map.set("year", 2018);

console.log(map.get("title"));
console.log(map.get("year"));

//객체를 키로 사용하는 것이 가능
let map = new Map()
	  key1 = {},
	  key2 = {};

map.set(key1, "Understanding ES6");
map.set(key2, 2018);
console.log(map.get(key1));
console.log(map.get(key2));
```
	
* 배열로 초기화 할 수 있다.
```javascript
let map = new Map([["name", "jinyeong"], ["age", 27]]);
console.log(map.has("name"));   //true
console.log(map.get("name"));   // "jinyeong"
...
```

### Map.forEach()

다음 세개의 인자를 받는 콜백 함수 전달
* Map의 다음 위치의 값
* 값에 대한 키
* 값을 읽어들인 Map
	
콜백 함수에서 this를 사용할 필요가 있다면 forEach 두 번째 인자로 this 를 보내면 된다.
```javascript
let map = new Map([["name", "jinyeong"], ["age", 27]]);
map.forEach(function(value, key, ownerMap) {
	console.log(key + " " + value);
	console.log(ownerMap === map);
});
```

### Weak Map
	
* Map 에 객체의 약한 참조를 저장하는 방식
* 모든 키는 객체여야 함.
* 각 객체 참조는 약한 참조이므로 가비지 컬렉션의 대상이 된다
* 값이 아닌 키만 약한 참조 -> Weak Map 값으로 저장된 객체는 다른 모든 참조가 지워지는 경우에도 가비지 컬렉션이 되지 않음.
* 초기화도 Map처럼 가능, 대신 key 값으로 객체를 넘겨주기
* .has() , .delete(), .clear() 메서드를 가짐

```javascript
let map = new WeakMap(),
	  element = document.querySelector(".element");
map.set(element, "Original");

let value = map.get(element);
console.log(value);     // "Original"

//엘리먼트 제거
element.parentNode.removeChild(element);
element = null;

//여기서 Weak Map은 비어있는 상태
```

### 비공개 객체 데이터
책을 참고해 봅시다.

### Object와 Map 비교 

전통적으로 objects 는 문자열을 값에 매핑하는 데 사용되었다. Object는 키를 값으로 설정하고, 값을 검색하고, 
키를 삭제하고, 키에 저장된 내용을 검색 할 수 있게 만들어준다. 그러나 Map 객체는 더 나은 맵이 되도록 하는 몇 가지 장점을 가지고 있다.

* Object의 키는 Strings이며, Map의 키는 모든 값을 가질 수 있다.
* Object는 크기를 수동으로 추적해야하지만, Map은 크기를 쉽게 얻을 수 있다.
* Map은 삽입된 순서대로 반복된다.
* 객체(Object)에는 prototype이 있어 Map에 기본 키들이 있다. (이것은 map = Object.create(null) 를 사용하여 우회할 수 있다. )

Object 혹은 Map중에 어느 것을 사용할지를 결정하는데 도움을 줄 두가지 팁이 있다.
*  실행 시까지 키를 알수 없고, 모든 키가 동일한 type이며 모든 값들이 동일한 type일 경우에는 objects를  대신해서 map을 사용해라. 
* 각 개별 요소에 대해 적용해야 하는 로직이 있을 경우에는 objects를 사용해라. 

[키기반의 컬렉션 - JavaScript | MDN] (https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Keyed_collections)
